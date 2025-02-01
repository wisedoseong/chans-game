<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import { cn } from '$lib/utils';
	import * as Progress from '$lib/components/ui/progress';
	import * as Card from '$lib/components/ui/card';
	import { Button } from '$lib/components/ui/button';

	// 디바이스 타입 체크
	let isMobile = $state(false);
	let gameWidth = $state(0);
	let gameHeight = $state(0);

	// Add new state variables near the other state declarations
	let isPhone = $state(false);
	let isLandscape = $state(false);

	// 게임 상태 관리
	let gameState = $state({
		isPlaying: false,
		score: 0,
		energy: 3,
		airKills: 0,
		subKills: 0,
		gameLoop: 0,
		lastTime: 0,
		level: 1,
		levelScore: 0,
		lastMissileTime: 0, // 마지막 미사일 발사 시간
		lastTorpedoTime: 0 // 마지막 어뢰 발사 시간
	});

	// 플레이어 상태
	let player = $state({
		x: 0,
		y: 0,
		isMoving: false,
		direction: { x: 0, y: 0 },
		size: { width: 0, height: 0 }
	});

	// 키보드 상태
	let keys = $state({
		ArrowLeft: false,
		ArrowRight: false,
		ArrowUp: false,
		ArrowDown: false,
		KeyA: false,
		KeyD: false,
		KeyS: false
	});

	// 조이스틱 상태
	let joystick = $state({
		active: false,
		startX: 0,
		startY: 0,
		currentX: 0,
		currentY: 0
	});

	// 적 객체 배열
	let enemies = $state({
		air: [] as Array<{
			id: number;
			x: number;
			y: number;
			type: 'plane' | 'helicopter';
			direction: number;
			height: number;
			lastShot: number;
		}>,
		sub: [] as Array<{
			id: number;
			x: number;
			y: number;
			type: 'submarine';
			direction: number;
			depth: number;
			lastShot: number;
		}>
	});

	// 무기 객체 배열
	let weapons = $state({
		missiles: [] as Array<{
			id: number;
			x: number;
			y: number;
			targetId?: number;
			isEnemy?: boolean;
		}>,
		torpedoes: [] as Array<{
			id: number;
			x: number;
			y: number;
			direction: number;
			isEnemy?: boolean;
		}>
	});

	let nextId = $state(0);

	// 이미지 에셋 관리
	let assets = $state({
		player: '', // 플레이어 잠수함 이미지
		enemyPlane: '', // 적 비행기 이미지
		enemyHelicopter: '', // 적 헬리콥터 이미지
		enemySubmarine: '', // 적 잠수함 이미지
		missile: '', // 미사일 이미지
		torpedo: '' // 어뢰 이미지
	});

	// 게임 일시중지 상태
	let isPaused = $state(false);

	// 이미지 로드 함수
	function loadImage(url: string): Promise<string> {
		return new Promise((resolve, reject) => {
			const img = new Image();
			img.onload = () => resolve(url);
			img.onerror = () => resolve(''); // 이미지 로드 실패시 빈 문자열 반환
			img.src = url;
		});
	}

	// 이미지 에셋 초기화
	async function initAssets() {
		try {
			assets.player = await loadImage('/images/player.png');
			assets.enemyPlane = await loadImage('/images/enemy-plane.png');
			assets.enemyHelicopter = await loadImage('/images/enemy-helicopter.png');
			assets.enemySubmarine = await loadImage('/images/enemy-submarine.png');
			assets.missile = await loadImage('/images/missile.png');
			assets.torpedo = await loadImage('/images/torpedo.png');
		} catch (error) {
			console.error('Failed to load assets:', error);
		}
	}

	// 게임 시작
	function startGame() {
		gameState.isPlaying = true;
		gameState.score = 0;
		gameState.energy = 3;
		gameState.level = 1;
		gameState.levelScore = 0;
		gameState.lastTime = performance.now();
		resetPlayer();
		gameState.gameLoop = requestAnimationFrame(update);
	}

	// 게임 종료
	function endGame() {
		gameState.isPlaying = false;
		cancelAnimationFrame(gameState.gameLoop);
		enemies.air = [];
		enemies.sub = [];
		weapons.missiles = [];
		weapons.torpedoes = [];
	}

	// 난이도 업데이트
	function updateLevel() {
		const levelThreshold = 1000;
		if (gameState.levelScore >= levelThreshold) {
			gameState.level++;
			gameState.levelScore = 0;
		}
	}

	// 게임 업데이트 루프
	function update(time: number) {
		const deltaTime = time - gameState.lastTime;
		gameState.lastTime = time;
		if (isPaused) return;

		if (!isMobile) {
			updatePlayerPC();
		} else if (player.isMoving) {
			updatePlayerMobile(deltaTime);
		}

		// 적 생성
		if (Math.random() < 0.005 * gameState.level) {
			const maxEnemies = isMobile ? 3 : 5;
			if (enemies.air.length + enemies.sub.length < maxEnemies) {
				createEnemy();
			}
		}

		// 적 이동 및 공격
		updateEnemies(deltaTime);

		// 무기 이동
		updateWeapons(deltaTime);

		// 충돌 감지
		checkCollisions();

		// 난이도 업데이트
		updateLevel();

		if (gameState.isPlaying) {
			gameState.gameLoop = requestAnimationFrame(update);
		}
	}

	// PC 플레이어 업데이트
	function updatePlayerPC() {
		const speed = 5;
		let dx = 0;
		let dy = 0;

		if (keys.ArrowLeft) dx -= speed;
		if (keys.ArrowRight) dx += speed;
		if (keys.ArrowUp) dy -= speed;
		if (keys.ArrowDown) dy += speed;

		if (dx !== 0 || dy !== 0) {
			player.x = Math.max(0, Math.min(gameWidth, player.x + dx));
			player.y = Math.max(gameHeight * 0.6, Math.min(gameHeight, player.y + dy));
		}

		// 무기 발사
		if (keys.KeyA) fireTorpedo(-1);
		if (keys.KeyD) fireTorpedo(1);
		if (keys.KeyS) fireMissile();
	}

	// 모바일 플레이어 업데이트
	function updatePlayerMobile(deltaTime: number) {
		const speed = 0.3;
		player.x += player.direction.x * speed * deltaTime;
		player.y += player.direction.y * speed * deltaTime;

		player.x = Math.max(0, Math.min(gameWidth, player.x));
		player.y = Math.max(gameHeight * 0.6, Math.min(gameHeight, player.y));
	}

	// 적 생성
	function createEnemy() {
		const id = nextId++;
		const size = isMobile ? 0.7 : 1;

		if (Math.random() < 0.5 && enemies.air.length < 3) {
			// 공중 적
			const type = Math.random() < 0.7 ? 'plane' : 'helicopter';
			const x = Math.random() < 0.5 ? -50 : gameWidth + 50;
			const height = Math.random() * 0.3 * gameHeight;
			enemies.air = [
				...enemies.air,
				{
					id,
					x,
					y: gameHeight * 0.2 + height,
					type,
					direction: x < 0 ? 1 : -1,
					height,
					lastShot: 0
				}
			];
		} else if (enemies.sub.length < 2) {
			// 수중 적
			const x = Math.random() * gameWidth;
			const y = gameHeight * 0.9;
			enemies.sub = [
				...enemies.sub,
				{
					id,
					x,
					y,
					type: 'submarine',
					direction: Math.random() < 0.5 ? 1 : -1,
					depth: 0,
					lastShot: 0
				}
			];
		}
	}

	// 적 이동
	function updateEnemies(deltaTime: number) {
		const baseAirSpeed = 0.1;
		const baseSubSpeed = 0.05;
		const speedMultiplier = 1 + (gameState.level - 1) * 0.2;
		const currentTime = performance.now();

		// 공중 적 이동 및 공격
		enemies.air = enemies.air.filter((enemy) => {
			enemy.x += enemy.direction * baseAirSpeed * speedMultiplier * deltaTime;

			// 적 공격
			if (currentTime - enemy.lastShot > 2000) {
				enemy.lastShot = currentTime;
				const id = nextId++;
				weapons.missiles = [
					...weapons.missiles,
					{
						id,
						x: enemy.x,
						y: enemy.y,
						isEnemy: true
					}
				];
			}

			return enemy.x > -100 && enemy.x < gameWidth + 100;
		});

		// 수중 적 이동 및 공격
		enemies.sub = enemies.sub.filter((enemy) => {
			enemy.x += enemy.direction * baseSubSpeed * speedMultiplier * deltaTime;
			enemy.depth = Math.sin(enemy.x * 0.01) * 30;
			enemy.y = gameHeight * 0.9 + enemy.depth;

			// 적 공격
			if (currentTime - enemy.lastShot > 3000) {
				enemy.lastShot = currentTime;
				const id = nextId++;
				weapons.torpedoes = [
					...weapons.torpedoes,
					{
						id,
						x: enemy.x,
						y: enemy.y,
						direction: enemy.x < player.x ? 1 : -1,
						isEnemy: true
					}
				];
			}

			return enemy.x > -100 && enemy.x < gameWidth + 100;
		});
	}

	// 무기 이동
	function updateWeapons(deltaTime: number) {
		const missileSpeed = 0.4;
		const torpedoSpeed = 0.25;

		// 미사일 이동
		weapons.missiles = weapons.missiles.filter((missile) => {
			if (missile.isEnemy) {
				missile.y += missileSpeed * deltaTime;
				return missile.y < gameHeight;
			} else {
				missile.y -= missileSpeed * deltaTime;
				return missile.y > 0;
			}
		});

		// 어뢰 이동
		weapons.torpedoes = weapons.torpedoes.filter((torpedo) => {
			torpedo.x += torpedo.direction * torpedoSpeed * deltaTime;
			return torpedo.x > 0 && torpedo.x < gameWidth;
		});
	}

	// 충돌 감지
	function checkCollisions() {
		const checkCollision = (
			obj1: { x: number; y: number },
			obj2: { x: number; y: number },
			radius: number
		) => {
			const dx = obj1.x - obj2.x;
			const dy = obj1.y - obj2.y;
			const distance = Math.sqrt(dx * dx + dy * dy);
			return distance < radius;
		};

		// 미사일과 적 충돌 (플레이어 미사일이 공중, 수중 적 모두 공격)
		weapons.missiles = weapons.missiles.filter((missile) => {
			if (!missile.isEnemy) {
				let hit = false;
				enemies.air = enemies.air.filter((enemy) => {
					if (checkCollision(missile, enemy, 20)) {
						gameState.score += 100;
						gameState.levelScore += 100;
						gameState.airKills++;
						hit = true;
						return false;
					}
					return true;
				});
				enemies.sub = enemies.sub.filter((enemy) => {
					if (checkCollision(missile, enemy, 20)) {
						gameState.score += 150;
						gameState.levelScore += 150;
						gameState.subKills++;
						hit = true;
						return false;
					}
					return true;
				});
				return !hit;
			}
			return true;
		});

		// 적 무기와 플레이어 충돌
		weapons.torpedoes = weapons.torpedoes.filter((torpedo) => {
			if (!torpedo.isEnemy) {
				// 플레이어 어뢰: 적 잠수함 공격
				let hit = false;
				enemies.sub = enemies.sub.filter((enemy) => {
					if (checkCollision(torpedo, enemy, 20)) {
						gameState.score += 150;
						gameState.levelScore += 150;
						gameState.subKills++;
						hit = true;
						return false;
					}
					return true;
				});
				return !hit;
			} else {
				// 적 어뢰: 플레이어 공격
				if (checkCollision(torpedo, player, 25)) {
					gameState.energy--;
					if (gameState.energy <= 0) {
						endGame();
					}
					return false;
				}
				return true;
			}
		});

		// 적과 플레이어 직접 충돌
		enemies.air.forEach((enemy) => {
			if (checkCollision(enemy, player, 30)) {
				gameState.energy--;
				if (gameState.energy <= 0) {
					endGame();
				}
			}
		});

		enemies.sub.forEach((enemy) => {
			if (checkCollision(enemy, player, 30)) {
				gameState.energy--;
				if (gameState.energy <= 0) {
					endGame();
				}
			}
		});
	}

	// 무기 발사
	function fireMissile() {
		const currentTime = performance.now();
		if (currentTime - gameState.lastMissileTime < 2000) return; // 2초 인터벌

		const id = nextId++;
		weapons.missiles = [
			...weapons.missiles,
			{
				id,
				x: player.x,
				y: player.y
			}
		];
		gameState.lastMissileTime = currentTime;
	}

	function fireTorpedo(direction: number) {
		const currentTime = performance.now();
		if (currentTime - gameState.lastTorpedoTime < 2000) return; // 2초 인터벌

		const id = nextId++;
		weapons.torpedoes = [
			...weapons.torpedoes,
			{
				id,
				x: player.x,
				y: player.y,
				direction
			}
		];
		gameState.lastTorpedoTime = currentTime;
	}

	// 플레이어 초기화
	function resetPlayer() {
		player.x = gameWidth / 2;
		player.y = gameHeight * 0.8;
		player.size = {
			width: isMobile ? 12 : 16,
			height: isMobile ? 12 : 16
		};
	}

	// PC 키보드 이벤트 핸들러
	function handleKeyDown(event: KeyboardEvent) {
		const key = event.code as keyof typeof keys;
		if (event.code === 'Space') {
			if (isPaused) {
				resumeGame();
			} else {
				pauseGame();
			}
			return;
		}
		if (key in keys) {
			keys[key] = true;
		}
	}

	function handleKeyUp(event: KeyboardEvent) {
		const key = event.code as keyof typeof keys;
		if (key in keys) {
			keys[key] = false;
		}
	}

	// 모바일 터치 이벤트 핸들러
	function handleTouchStart(event: TouchEvent) {
		const touch = event.touches[0];
		joystick.active = true;
		joystick.startX = touch.clientX;
		joystick.startY = touch.clientY;
		joystick.currentX = touch.clientX;
		joystick.currentY = touch.clientY;
	}

	function handleTouchMove(event: TouchEvent) {
		if (!joystick.active) return;
		const touch = event.touches[0];
		joystick.currentX = touch.clientX;
		joystick.currentY = touch.clientY;

		const dx = joystick.currentX - joystick.startX;
		const dy = joystick.currentY - joystick.startY;
		const distance = Math.sqrt(dx * dx + dy * dy);
		const maxDistance = 50;

		if (distance > 0) {
			player.direction.x = dx / distance;
			player.direction.y = dy / distance;
			player.isMoving = true;
		}
	}

	function handleTouchEnd() {
		joystick.active = false;
		player.isMoving = false;
		player.direction = { x: 0, y: 0 };
	}

	// 화면 크기 업데이트
	function updateGameSize() {
		if (typeof window !== 'undefined') {
			gameWidth = window.innerWidth;
			gameHeight = window.innerHeight;
			// Update isMobile using window width; but also consider phone devices regardless of orientation
			isMobile = window.innerWidth < 768 || isPhone;
			// Determine if current orientation is landscape
			isLandscape = window.innerWidth > window.innerHeight;
			if (player.x && player.y) {
				resetPlayer();
			}
		}
	}

	// 게임 일시중지 함수
	function pauseGame() {
		isPaused = true;
		cancelAnimationFrame(gameState.gameLoop);
	}

	function resumeGame() {
		isPaused = false;
		gameState.lastTime = performance.now();
		gameState.gameLoop = requestAnimationFrame(update);
	}

	onMount(() => {
		if (typeof window !== 'undefined') {
			// Use user agent to detect if device is a phone
			isPhone = /Mobi|Android/i.test(navigator.userAgent);
			updateGameSize();
			window.addEventListener('resize', updateGameSize);
			if (!isMobile) {
				window.addEventListener('keydown', handleKeyDown);
				window.addEventListener('keyup', handleKeyUp);
			}
			initAssets(); // 이미지 에셋 초기화
		}
	});

	onDestroy(() => {
		if (typeof window !== 'undefined') {
			if (gameState.gameLoop) {
				cancelAnimationFrame(gameState.gameLoop);
			}
			window.removeEventListener('resize', updateGameSize);
			if (!isMobile) {
				window.removeEventListener('keydown', handleKeyDown);
				window.removeEventListener('keyup', handleKeyUp);
			}
		}
	});
</script>

<div class="relative w-full h-screen overflow-hidden">
	<!-- 게임 배경 -->
	<div class="absolute inset-0">
		<div class="h-[60%] bg-gradient-to-b from-blue-300 to-blue-500"></div>
		<div class="h-[40%] bg-gradient-to-b from-blue-700 to-blue-900"></div>
	</div>

	<!-- 게임 시작 화면 -->
	{#if !gameState.isPlaying}
		<div class="absolute inset-0 flex items-center justify-center bg-black/50">
			<Card.Root class="w-[300px]">
				<Card.Header>
					<Card.Title>잠수함 게임</Card.Title>
					<Card.Description>
						{#if isMobile}
							조이스틱으로 이동하고 버튼으로 공격하세요
						{:else}
							방향키로 이동, A/D로 어뢰 발사, S로 미사일 발사
						{/if}
					</Card.Description>
				</Card.Header>
				<Card.Content>
					<Button class="w-full" onclick={startGame}>게임 시작</Button>
				</Card.Content>
			</Card.Root>
		</div>
	{/if}

	<!-- HUD -->
	<div class="absolute top-4 left-4 right-4 flex justify-between items-center">
		<div class="flex gap-2">
			{#each Array(gameState.energy) as _}
				<svg class="w-6 h-6 fill-red-500" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg">
					<path
						d="M12 21.35l-1.45-1.32C5.4 15.36 2 12.28 2 8.5 2 5.42 4.42 3 7.5 3c1.74 0 3.41.81 4.5 2.09C13.09 3.81 14.76 3 16.5 3 19.58 3 22 5.42 22 8.5c0 3.78-3.4 6.86-8.55 11.54L12 21.35z"
					/>
				</svg>
			{/each}
		</div>
		<div class="text-white text-xl font-bold flex flex-col items-end">
			<div>점수: {gameState.score}</div>
			<div class="text-sm">레벨: {gameState.level}</div>
		</div>
	</div>

	<!-- 일시중지 오버레이 (일시중지 상태인 경우) -->
	{#if isPaused}
		<div class="absolute inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
			<div class="text-white text-3xl font-bold">일시중지중</div>
		</div>
	{/if}

	<!-- 플레이어 -->
	<div
		class="absolute"
		class:w-12={isMobile}
		class:h-12={isMobile}
		class:w-16={!isMobile}
		class:h-16={!isMobile}
		style="left: {player.x}px; top: {player.y}px; transform: translate(-50%, -50%);"
	>
		{#if assets.player}
			<img src={assets.player} alt="player" class="w-full h-full object-contain" />
		{:else}
			<div class="w-full h-full bg-yellow-500"></div>
		{/if}
	</div>

	<!-- 모바일 컨트롤 -->
	{#if isMobile || isPhone}
		<!-- 조이스틱 (왼쪽 중앙에 배치) -->
		<div
			class="fixed left-2 bottom-2 w-32 h-32 rounded-full bg-black/20"
			ontouchstart={handleTouchStart}
			ontouchmove={handleTouchMove}
			ontouchend={handleTouchEnd}
		>
			{#if joystick.active}
				<div
					class="absolute w-16 h-16 bg-white/50 rounded-full"
					style="left: {joystick.currentX - joystick.startX + 16}px; top: {joystick.currentY -
						joystick.startY +
						16}px; transform: translate(-50%, -50%);"
				></div>
			{/if}
		</div>

		<!-- 무기 버튼 (오른쪽 하단에 배치) -->
		<div class="absolute right-2 bottom-2 flex flex-col gap-4">
			<Button variant="secondary" class="w-16 h-16 rounded-full" onclick={fireMissile}>
				미사일
			</Button>
			<div class="flex gap-4">
				<Button variant="secondary" class="w-16 h-16 rounded-full" onclick={() => fireTorpedo(-1)}>
					←
				</Button>
				<Button variant="secondary" class="w-16 h-16 rounded-full" onclick={() => fireTorpedo(1)}>
					→
				</Button>
			</div>
		</div>

		<!-- 일시중지 버튼 -->
		<div class="absolute top-16 left-4 z-[100]">
			<Button
				variant="secondary"
				onclick={() => {
					isPaused ? resumeGame() : pauseGame();
				}}
			>
				{isPaused ? '▶' : '❚❚'}
			</Button>
		</div>
	{/if}

	<!-- 적 렌더링 -->
	{#each enemies.air as enemy (enemy.id)}
		<div
			class="absolute"
			class:w-8={isMobile}
			class:h-8={isMobile}
			class:w-12={!isMobile}
			class:h-12={!isMobile}
			style="left: {enemy.x}px; top: {enemy.y}px; transform: translate(-50%, -50%) scaleX({enemy.direction});"
		>
			{#if enemy.type === 'plane'}
				{#if assets.enemyPlane}
					<img src={assets.enemyPlane} alt="enemy plane" class="w-full h-full object-contain" />
				{:else}
					<div class="w-full h-full bg-red-500 rotate-45"></div>
				{/if}
			{:else if assets.enemyHelicopter}
				<img
					src={assets.enemyHelicopter}
					alt="enemy helicopter"
					class="w-full h-full object-contain"
				/>
			{:else}
				<div class="w-full h-full bg-purple-500 rounded-full"></div>
			{/if}
		</div>
	{/each}

	{#each enemies.sub as enemy (enemy.id)}
		<div
			class="absolute"
			class:w-12={isMobile}
			class:h-6={isMobile}
			class:w-16={!isMobile}
			class:h-8={!isMobile}
			style="left: {enemy.x}px; top: {enemy.y}px; transform: translate(-50%, -50%) scaleX({enemy.direction});"
		>
			{#if assets.enemySubmarine}
				<img
					src={assets.enemySubmarine}
					alt="enemy submarine"
					class="w-full h-full object-contain"
				/>
			{:else}
				<div class="w-full h-full bg-gray-700 rounded-full"></div>
			{/if}
		</div>
	{/each}

	<!-- 무기 렌더링 -->
	{#each weapons.missiles as missile (missile.id)}
		<div
			class="absolute"
			class:w-1={isMobile}
			class:h-6={isMobile}
			class:w-2={!isMobile}
			class:h-8={!isMobile}
			style="left: {missile.x}px; top: {missile.y}px; transform: translate(-50%, -50%) {missile.isEnemy
				? 'rotate(180deg)'
				: ''};"
		>
			{#if assets.missile}
				<img src={assets.missile} alt="missile" class="w-full h-full object-contain" />
			{:else}
				<div class="w-full h-full bg-yellow-300"></div>
			{/if}
		</div>
	{/each}

	{#each weapons.torpedoes as torpedo (torpedo.id)}
		<div
			class="absolute"
			class:w-6={isMobile}
			class:h-2={isMobile}
			class:w-8={!isMobile}
			class:h-3={!isMobile}
			style="left: {torpedo.x}px; top: {torpedo.y}px; transform: translate(-50%, -50%) scaleX({torpedo.direction});"
		>
			{#if assets.torpedo}
				<img src={assets.torpedo} alt="torpedo" class="w-full h-full object-contain" />
			{:else}
				<div class="w-full h-full bg-orange-500"></div>
			{/if}
		</div>
	{/each}
</div>
