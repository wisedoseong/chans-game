<script lang="ts">
	// 상태 관리
	const characterWidth = 20; // 캐릭터의 실제 히트박스 너비
	const characterHeight = 20; // 캐릭터의 실제 히트박스 높이
	const numberSize = 10; // 떨어지는 숫자의 히트박스 크기

	let position = $state(50);
	let velocity = $state(0);
	let gameStarted = $state(false);
	let difficulty = $state<'basic' | 'advanced' | null>(null);
	let currentProblem = $state({ question: '', answer: 0 });
	let fallingNumbers = $state<
		Array<{
			id: number;
			value: number;
			x: number;
			y: number;
			isCorrect: boolean;
			speed: number;
		}>
	>([]);
	let problemIndex = $state(0);
	let timeRecords = $state<number[]>([]);
	let gameOver = $state(false);
	let countdown = $state(5);
	let problemStartTime = $state<number | null>(null);
	let operationType = $state<'addition' | 'subtraction' | 'both' | null>(null);
	let difficultySelected = $state(false);
	let limitUnder100 = $state(true); // 기본값 true로 설정
	let isPaused = $state(false);
	let isTransitioning = $state(false); // 다음 단계로 넘어가는 중인지 여부
	let selectedCharacter = $state<'avata1' | 'avata2' | 'avata3' | 'custom'>('avata1');
	let customCharacterImage = $state<string | null>(null);
	let showHitbox = $state(false); // 개발 모드에서만 사용
	let gameCleared = $state(false);

	// 추가: 캐릭터 생명(하트)
	let lives = $state(3);

	// 터치 관련 상태
	let touchStartX = $state(0);
	let isTouching = $state(false);

	// 모바일 여부 확인
	let isMobile = $state(false);

	// 아바타 이미지 경로 상수 추가
	const AVATAR_IMAGES = {
		avata1: '/avata/avata1.webp',
		avata2: '/avata/avata2.png',
		avata3: '/avata/avata3.png'
	} as const;

	// 컴포넌트 마운트 시 모바일 체크
	$effect(() => {
		isMobile = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
			navigator.userAgent
		);
	});

	// 게임 설정 상수 (정답 확률을 40%로 조정)
	const GAME_CONFIG = {
		// 스테이지별 설정
		STAGES: [
			{ speed: 0.2, correctAnswerRate: 0.3 }, // 1-3 스테이지
			{ speed: 0.3, correctAnswerRate: 0.2 }, // 4-6 스테이지
			{ speed: 0.4, correctAnswerRate: 0.1 }, // 7-8 스테이지
			{ speed: 0.5, correctAnswerRate: 0.1 } // 9-10 스테이지
		],
		// 캐릭터 이동 관련 설정
		MOVEMENT: {
			ACCELERATION: 0.5,
			FRICTION: 0.92,
			MAX_SPEED: 5
		},
		// 숫자 생성 관련 설정
		NUMBERS: {
			SPAWN_INTERVAL: 800, // ms (필요 시 이 값을 조정)
			MIN_SPAWN_COUNT: 2,
			MAX_SPAWN_COUNT: 3,
			MAX_SUM: 999, // 제한 없을 때의 최대값
			MAX_SUM_UNDER_100: 99 // 100미만 제한시 최대값
		}
	};

	// 최소 x 좌표 간격 (겹치지 않도록)
	const MIN_X_DISTANCE = 12;

	// 현재 스테이지 설정 가져오기
	function getCurrentStageConfig(problemIndex: number) {
		const stageIndex = Math.floor(problemIndex / 3); // 3문제마다 스테이지 변경
		return GAME_CONFIG.STAGES[Math.min(stageIndex, GAME_CONFIG.STAGES.length - 1)];
	}

	// 문제 생성 함수 수정
	function generateProblem() {
		if (!difficulty || !operationType) return { question: '', answer: 0 };

		let num1: number, num2: number;
		const maxSum = limitUnder100
			? GAME_CONFIG.NUMBERS.MAX_SUM_UNDER_100
			: GAME_CONFIG.NUMBERS.MAX_SUM;

		// 덧셈 문제 생성
		function generateAdditionProblem() {
			do {
				// 100미만 제한시 10-59, 아닐 경우 10-899
				const maxNum = limitUnder100 ? 50 : 450;
				num1 = Math.floor(Math.random() * maxNum) + 10;
				num2 = Math.floor(Math.random() * maxNum) + 10;
			} while (
				num1 + num2 > maxSum || // 최대 합 제한
				(difficulty === 'basic' && (num1 % 10) + (num2 % 10) >= 10) // 기본 모드에서는 올림이 없도록
			);
			return { num1, num2, isAddition: true };
		}

		// 뺄셈 문제 생성
		function generateSubtractionProblem() {
			do {
				// 100미만 제한시 10-99, 아닐 경우 10-999
				const maxNum = limitUnder100 ? 89 : 989;
				num1 = Math.floor(Math.random() * maxNum) + 10;
				num2 = Math.floor(Math.random() * Math.min(num1 - 9, maxSum - num1)) + 1;
			} while (
				difficulty === 'basic' &&
				num1 % 10 < num2 % 10 // 기본 모드에서는 내림이 없도록
			);
			return { num1, num2, isAddition: false };
		}

		let problem;
		if (operationType === 'addition') {
			problem = generateAdditionProblem();
		} else if (operationType === 'subtraction') {
			problem = generateSubtractionProblem();
		} else {
			// both인 경우 랜덤하게 선택
			problem = Math.random() > 0.5 ? generateAdditionProblem() : generateSubtractionProblem();
		}

		return {
			question: `${problem.num1} ${problem.isAddition ? '+' : '-'} ${problem.num2}`,
			answer: problem.isAddition ? problem.num1 + problem.num2 : problem.num1 - problem.num2
		};
	}

	// 잘못된 답 생성
	function generateWrongAnswer(correctAnswer: number) {
		const errors = [
			correctAnswer + 10,
			correctAnswer - 10,
			correctAnswer + (Math.floor(Math.random() * 3) + 1),
			correctAnswer - (Math.floor(Math.random() * 3) + 1),
			correctAnswer + (Math.floor(Math.random() * 2) + 1) * 10,
			correctAnswer - (Math.floor(Math.random() * 2) + 1) * 10
		];
		return errors[Math.floor(Math.random() * errors.length)];
	}

	// 떨어지는 숫자 생성 함수 수정
	function generateFallingNumber() {
		const stageConfig = getCurrentStageConfig(problemIndex);
		// 40% 정답 확률로 결정 (GAME_CONFIG 의 값 사용)
		const isCorrect = Math.random() < stageConfig.correctAnswerRate;
		const correctAnswer = currentProblem.answer;
		let number;

		if (isCorrect) {
			number = correctAnswer;
		} else {
			do {
				number = generateWrongAnswer(correctAnswer);
			} while (number === correctAnswer);
		}

		const baseSpeed = stageConfig.speed;
		const randomSpeedOffset = Math.random() * 0.1; // 약간의 랜덤성 추가

		return {
			id: Date.now() + Math.random(),
			value: number,
			x: Math.random() * 80 + 10, // 나중에 spawn 시 겹침 여부 체크
			y: 0,
			isCorrect,
			speed: baseSpeed + randomSpeedOffset
		};
	}

	// 숫자 생성 및 스폰 (동일한 x 값이 겹치지 않도록 하고, 한 화면 최대 6개)
	const spawnInterval = setInterval(() => {
		if (isPaused || isTransitioning) return; // 일시정지나 전환 중에는 생성 중지

		// 한 화면에 최대 6개 유지
		if (fallingNumbers.length >= 6) return;

		// 새로 생성할 개수를 결정(최대 생성 개수가 넘지 않도록)
		const spawnCount = Math.min(
			Math.floor(
				Math.random() *
					(GAME_CONFIG.NUMBERS.MAX_SPAWN_COUNT - GAME_CONFIG.NUMBERS.MIN_SPAWN_COUNT + 1)
			) + GAME_CONFIG.NUMBERS.MIN_SPAWN_COUNT,
			6 - fallingNumbers.length
		);

		const newNumbers = [];
		let attempts = 0;
		while (newNumbers.length < spawnCount && attempts < 20) {
			// 최대 시도 횟수 제한
			const candidate = generateFallingNumber();
			// 기존 fallingNumbers와 새로 생성할 newNumbers에 대해 x 좌표 간격 체크
			const isOverlapping = [...fallingNumbers, ...newNumbers].some(
				(num) => Math.abs(num.x - candidate.x) < MIN_X_DISTANCE
			);
			if (!isOverlapping) {
				newNumbers.push(candidate);
			}
			attempts++;
		}
		fallingNumbers = [...fallingNumbers, ...newNumbers];
	}, GAME_CONFIG.NUMBERS.SPAWN_INTERVAL);

	// 위치 업데이트 및 충돌 감지 부분 수정
	const updateInterval = setInterval(() => {
		if (isPaused || isTransitioning) return;

		const newNumbers = fallingNumbers
			.map((num) => ({
				...num,
				y: num.y + num.speed
			}))
			.filter((num) => {
				// 캐릭터 히트박스 수정 (화면 비율 기반으로 캐릭터가 위치한 영역에 맞춤)
				const characterLeft = position - characterWidth / 2;
				const characterRight = position + characterWidth / 2;
				// 기존 85 ~ 95 대신 캐릭터 이미지의 실제 위치를 감안하여 (예: 90 ~ 100)로 조정
				const characterTop = 90;
				const characterBottom = 100;

				// 숫자의 히트박스 계산 (기존 값 유지)
				const numberLeft = num.x - numberSize / 2;
				const numberRight = num.x + numberSize / 2;
				const numberTop = num.y - numberSize / 2;
				const numberBottom = num.y + numberSize / 2;

				const collision =
					numberRight >= characterLeft &&
					numberLeft <= characterRight &&
					numberBottom >= characterTop &&
					numberTop <= characterBottom;

				if (collision) {
					if (num.isCorrect) {
						handleCorrectAnswer();
						return false;
					} else {
						// 생명이 남아있으면 1개씩 깎고, 0이 되면 게임 오버
						lives--;
						if (lives <= 0) {
							gameOver = true;
						}
						return false;
					}
				}

				// 화면 밖으로 나간 숫자 제거
				return num.y < 100;
			});

		fallingNumbers = newNumbers;
	}, 16);

	// 새로운 문제 시작
	function startNewProblem() {
		const newProblem = generateProblem();
		currentProblem = newProblem;
		fallingNumbers = [];
		problemStartTime = Date.now();
		countdown = -1;
	}

	// 게임 로직
	$effect(() => {
		if (!gameStarted || gameOver) return;

		// 키보드 이벤트 핸들러 수정
		function handleKeyDown(e: KeyboardEvent) {
			if (e.key === ' ') {
				// 스페이스바
				e.preventDefault();
				isPaused = !isPaused;
				return;
			}

			if (isPaused) return; // 일시정지 중에는 이동 불가

			if (e.key === 'ArrowLeft' || e.key === 'ArrowRight') {
				e.preventDefault();
				const direction = e.key === 'ArrowLeft' ? -1 : 1;
				velocity = Math.max(
					Math.min(
						velocity + direction * GAME_CONFIG.MOVEMENT.ACCELERATION,
						GAME_CONFIG.MOVEMENT.MAX_SPEED
					),
					-GAME_CONFIG.MOVEMENT.MAX_SPEED
				);
			}
		}

		window.addEventListener('keydown', handleKeyDown);

		// 움직임 업데이트
		const moveInterval = setInterval(() => {
			if (isPaused || isTransitioning) return; // 일시정지나 전환 중에는 업데이트 중지
			velocity *= GAME_CONFIG.MOVEMENT.FRICTION;
			position = Math.max(0, Math.min(100, position + velocity));
		}, 16);

		return () => {
			window.removeEventListener('keydown', handleKeyDown);
			clearInterval(moveInterval);
			clearInterval(spawnInterval);
			clearInterval(updateInterval);
		};
	});

	// 카운트다운 효과 수정
	$effect(() => {
		if (countdown > 0) {
			const timer = setInterval(() => {
				countdown--;
			}, 1000);
			return () => clearInterval(timer);
		} else if (countdown === 0) {
			isTransitioning = false; // 전환 상태 해제
			startNewProblem();
		}
	});

	// 게임 시작시 첫 문제 생성
	$effect(() => {
		if (gameStarted && !gameOver && problemIndex === 0) {
			startNewProblem();
		}
	});

	// 터치 이벤트 핸들러
	function handleTouchStart(e: TouchEvent) {
		touchStartX = e.touches[0].clientX;
		isTouching = true;
	}

	function handleTouchMove(e: TouchEvent) {
		if (!isTouching || isPaused) return;

		const touchX = e.touches[0].clientX;
		const diff = touchX - touchStartX;
		const moveSpeed = diff * 0.1; // 이동 속도 조절

		velocity = Math.max(
			Math.min(moveSpeed, GAME_CONFIG.MOVEMENT.MAX_SPEED),
			-GAME_CONFIG.MOVEMENT.MAX_SPEED
		);

		touchStartX = touchX;
	}

	function handleTouchEnd() {
		isTouching = false;
		velocity = 0;
	}

	// 방향 버튼 핸들러
	function handleDirectionButton(direction: 'left' | 'right') {
		if (isPaused) return;

		const moveValue = direction === 'left' ? -1 : 1;
		// 연속 이동을 위한 velocity 직접 설정
		velocity = moveValue * GAME_CONFIG.MOVEMENT.MAX_SPEED * 0.7;
	}

	// 이미지 업로드 핸들러 추가
	function handleImageUpload(event: Event) {
		const input = event.target as HTMLInputElement;
		if (input.files && input.files[0]) {
			const reader = new FileReader();
			reader.onload = (e) => {
				customCharacterImage = e.target?.result as string;
			};
			reader.readAsDataURL(input.files[0]);
		}
	}

	// 스테이지 전환 효과 수정을 위한 함수
	function handleCorrectAnswer() {
		const timeTaken = Date.now() - (problemStartTime ?? 0);
		timeRecords = [...timeRecords, timeTaken];
		problemIndex++;
		if (problemIndex < 10) {
			isTransitioning = true;
			countdown = 5;
			// 모든 게임 요소 초기화
			fallingNumbers = [];
			position = 50; // 캐릭터 위치 중앙으로 초기화
			velocity = 0;
		} else {
			gameCleared = true; // gameOver 대신 gameCleared로 변경
		}
	}

	// 게임 초기화 함수 추가
	function resetGame() {
		gameStarted = false;
		gameCleared = false;
		gameOver = false;
		difficulty = null;
		operationType = null;
		difficultySelected = false;
		problemIndex = 0;
		timeRecords = [];
		position = 50;
		velocity = 0;
		fallingNumbers = [];
		currentProblem = { question: '', answer: 0 };
		lives = 3;
	}
</script>

<div class="fixed inset-0 bg-gray-900 overflow-hidden flex flex-col" style="touch-action: none;">
	<!-- 추가: 좌측 상단에 생명(하트) 표시 -->
	{#if gameStarted}
		<div class="absolute top-4 left-4 flex gap-2 z-20">
			{#each Array(lives) as _, i}
				<span class="text-red-500 text-2xl" title="Life {i + 1}">❤️</span>
			{/each}
		</div>
	{/if}

	{#if !gameStarted}
		<div class="flex-1 flex flex-col items-center justify-center gap-6 overflow-auto">
			<h1 class="text-4xl text-white mb-8">힘내라! 연산 하자</h1>

			{#if !difficultySelected}
				<h2 class="text-2xl text-white mb-4">난이도 선택</h2>
				<div class="flex gap-4">
					<button
						onclick={() => {
							difficulty = 'basic';
							difficultySelected = true;
						}}
						class="px-6 py-3 bg-blue-500 text-white rounded-lg text-xl hover:bg-blue-600"
					>
						기본 (올림 없음)
					</button>
					<button
						onclick={() => {
							difficulty = 'advanced';
							difficultySelected = true;
						}}
						class="px-6 py-3 bg-green-500 text-white rounded-lg text-xl hover:bg-green-600"
					>
						심화 (올림 포함)
					</button>
				</div>
			{:else}
				<h2 class="text-2xl text-white mb-4">캐릭터 선택</h2>
				<div class="flex gap-4 mb-6">
					<button
						class="p-4 bg-gray-700 rounded-lg {selectedCharacter === 'avata1'
							? 'ring-2 ring-yellow-400'
							: ''}"
						onclick={() => (selectedCharacter = 'avata1')}
					>
						<img
							src={AVATAR_IMAGES.avata1}
							alt="Avatar 1"
							class="w-14 h-14 object-cover rounded-lg"
						/>
					</button>
					<button
						class="p-4 bg-gray-700 rounded-lg {selectedCharacter === 'avata2'
							? 'ring-2 ring-yellow-400'
							: ''}"
						onclick={() => (selectedCharacter = 'avata2')}
					>
						<img
							src={AVATAR_IMAGES.avata2}
							alt="Avatar 2"
							class="w-14 h-14 object-cover rounded-lg"
						/>
					</button>
					<button
						class="p-4 bg-gray-700 rounded-lg {selectedCharacter === 'avata3'
							? 'ring-2 ring-yellow-400'
							: ''}"
						onclick={() => (selectedCharacter = 'avata3')}
					>
						<img
							src={AVATAR_IMAGES.avata3}
							alt="Avatar 3"
							class="w-14 h-14 object-cover rounded-lg"
						/>
					</button>
					<button
						class="p-4 bg-gray-700 rounded-lg {selectedCharacter === 'custom'
							? 'ring-2 ring-yellow-400'
							: ''}"
						onclick={() => (selectedCharacter = 'custom')}
					>
						<label class="cursor-pointer">
							{#if customCharacterImage}
								<img
									src={customCharacterImage}
									alt="Custom"
									class="w-12 h-12 object-cover rounded-lg"
								/>
							{:else}
								<div
									class="w-12 h-12 bg-gray-500 flex items-center justify-center text-white rounded-lg"
								>
									+
								</div>
							{/if}
							<input type="file" accept="image/*" class="hidden" onchange={handleImageUpload} />
						</label>
					</button>
				</div>
				<h2 class="text-2xl text-white mb-4">연산 선택</h2>
				<div class="flex flex-col items-center gap-4">
					<!-- 100미만 제한 체크박스 -->
					<label class="flex items-center gap-2 text-white text-lg">
						<input type="checkbox" bind:checked={limitUnder100} class="w-4 h-4" />
						<span>100미만으로 제한</span>
					</label>

					<!-- 기존 버튼들 -->
					<div class="flex gap-4">
						<button
							onclick={() => {
								operationType = 'addition';
								gameStarted = true;
							}}
							class="px-6 py-3 bg-blue-500 text-white rounded-lg text-xl hover:bg-blue-600"
						>
							덧셈만
						</button>
						<button
							onclick={() => {
								operationType = 'subtraction';
								gameStarted = true;
							}}
							class="px-6 py-3 bg-green-500 text-white rounded-lg text-xl hover:bg-green-600"
						>
							뺄셈만
						</button>
						<button
							onclick={() => {
								operationType = 'both';
								gameStarted = true;
							}}
							class="px-6 py-3 bg-purple-500 text-white rounded-lg text-xl hover:bg-purple-600"
						>
							모두
						</button>
					</div>
				</div>
				<button
					onclick={() => {
						difficultySelected = false;
						difficulty = null;
					}}
					class="mt-4 px-4 py-2 bg-gray-500 text-white rounded-lg text-sm hover:bg-gray-600"
				>
					난이도 다시 선택
				</button>
			{/if}
		</div>
	{:else}
		<div class="flex-none h-[20vh] flex items-center justify-center">
			<div class="text-center">
				<div class="text-2xl text-white mb-2">
					문제 {problemIndex + 1}/10
				</div>
				<div class="text-3xl text-yellow-400">
					{currentProblem.question}
				</div>
			</div>
		</div>

		<div class="flex-1 relative">
			{#each fallingNumbers as num (num.id)}
				<div
					class="absolute"
					style="left: {num.x}%; top: {num.y}%; transform: translate(-50%, -50%);"
				>
					<!-- 원형 스타일: 그라데이션, 그림자 효과 -->
					<div
						class="w-10 h-10 bg-gradient-to-br from-gray-700 to-gray-900 text-white flex items-center justify-center rounded-full shadow-lg"
					>
						{num.value}
					</div>

					{#if showHitbox}
						<div
							class="absolute w-[10px] h-[10px] border-2 border-red-500"
							style="transform: translate(-50%, -50%)"
						></div>
					{/if}
				</div>
			{/each}

			<div
				class="absolute bottom-4 w-20 h-20 cursor-grab"
				style:left="{position}%"
				style:transform="translateX(-50%)"
				ontouchstart={handleTouchStart}
				ontouchmove={handleTouchMove}
				ontouchend={handleTouchEnd}
			>
				{#if selectedCharacter === 'custom' && customCharacterImage}
					<img
						src={customCharacterImage}
						alt="Custom"
						class="w-full h-full object-cover rounded-lg"
					/>
				{:else if selectedCharacter !== 'custom'}
					<img
						src={AVATAR_IMAGES[selectedCharacter]}
						alt="Avatar"
						class="w-full h-full object-cover rounded-lg"
					/>
				{/if}
			</div>

			{#if showHitbox}
				<div
					class="absolute border-2 border-blue-500"
					style:left="{position}%"
					style:bottom="4px"
					style:width="{characterWidth}px"
					style:height="{characterHeight}px"
					style:transform="translateX(-50%)"
				></div>
			{/if}
		</div>

		{#if isMobile}
			<!-- 모바일용 일시정지 버튼 -->
			<button
				class="fixed top-4 right-4 w-12 h-12 bg-gray-800 bg-opacity-50 rounded-full text-white flex items-center justify-center z-10"
				onclick={() => (isPaused = !isPaused)}
			>
				{#if isPaused}
					▶
				{:else}
					❚❚
				{/if}
			</button>

			<div class="flex-none h-[20vh] flex justify-between items-center px-4">
				<button
					class="w-20 h-20 bg-gray-800 bg-opacity-50 rounded-full text-white text-4xl flex items-center justify-center active:bg-opacity-75"
					onpointerdown={() => handleDirectionButton('left')}
					onpointerup={() => (velocity = 0)}
					onpointerleave={() => (velocity = 0)}
				>
					←
				</button>
				<button
					class="w-20 h-20 bg-gray-800 bg-opacity-50 rounded-full text-white text-4xl flex items-center justify-center active:bg-opacity-75"
					onpointerdown={() => handleDirectionButton('right')}
					onpointerup={() => (velocity = 0)}
					onpointerleave={() => (velocity = 0)}
				>
					→
				</button>
			</div>
		{/if}

		{#if isPaused}
			<div class="absolute inset-0 flex items-center justify-center bg-black bg-opacity-50">
				<div class="text-4xl text-white font-bold text-center">
					일시정지
					<div class="text-xl mt-4">
						{#if isMobile}
							계속하려면 재생 버튼을 누르세요
						{:else}
							계속하려면 스페이스바를 누르세요
						{/if}
					</div>
				</div>
			</div>
		{/if}

		{#if gameCleared}
			<div
				class="absolute inset-0 flex flex-col items-center justify-center bg-black bg-opacity-75 text-white"
			>
				<div class="text-6xl mb-8 animate-bounce">🎉</div>
				<h2 class="text-4xl mb-4 text-yellow-400">축하합니다!</h2>
				<h3 class="text-2xl mb-6">모든 문제를 완료했습니다!</h3>

				<div class="text-xl mb-8">
					<p class="mb-4">문제 해결 시간</p>
					{#each timeRecords as time, index}
						<p class="mb-2">
							문제 {index + 1}: <span class="text-green-400">{(time / 1000).toFixed(2)}초</span>
						</p>
					{/each}
					<p class="mt-4 text-2xl">
						평균 시간: <span class="text-yellow-400">
							{(timeRecords.reduce((a, b) => a + b, 0) / timeRecords.length / 1000).toFixed(2)}초
						</span>
					</p>
				</div>

				<button
					onclick={resetGame}
					class="px-8 py-4 bg-gradient-to-r from-blue-500 to-purple-500 rounded-lg text-xl hover:from-blue-600 hover:to-purple-600 transition-all duration-300 shadow-lg hover:shadow-xl"
				>
					처음으로 돌아가기
				</button>
			</div>
		{/if}

		{#if gameOver}
			<div
				class="absolute inset-0 flex flex-col items-center justify-center bg-black bg-opacity-75 text-white"
			>
				<h2 class="text-4xl mb-4">Game Over!</h2>
				<p class="text-2xl mb-4">완료된 문제: {problemIndex}/10</p>
				<div class="text-xl">
					{#each timeRecords as time, index}
						<p>
							문제 {index + 1}: {(time / 1000).toFixed(2)}초
						</p>
					{/each}
				</div>
				<button
					onclick={() => window.location.reload()}
					class="mt-6 px-6 py-3 bg-blue-500 rounded-lg hover:bg-blue-600"
				>
					다시 시작
				</button>
			</div>
		{/if}

		{#if isTransitioning}
			<div class="absolute inset-0 bg-black bg-opacity-75 flex items-center justify-center">
				<div class="text-4xl text-white text-center">
					다음 문제 시작까지: {countdown}초
				</div>
			</div>
		{/if}
	{/if}
</div>

<style>
	/* iOS에서 바운스 스크롤 방지 */
	:global(body) {
		position: fixed;
		width: 100%;
		height: 100%;
		overflow: hidden;
	}
</style>
