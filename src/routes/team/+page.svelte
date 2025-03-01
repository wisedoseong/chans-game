<script lang="ts">
	import { ChevronUp, ChevronDown, Calendar, Book } from 'lucide-svelte';

	interface Skill {
		name: string;
		level: number;
		description: string;
	}

	interface Character {
		name: string;
		level: number;
		image: string;
		skills: Skill[];
	}

	interface Colleague {
		id: number;
		name: string;
		image: string;
		skills: Skill[];
		description: string;
		isPublic: boolean;
		order: number | null;
	}

	interface ActivityLog {
		date: string;
		category: string;
		content: string;
		result: string;
	}

	interface Data {
		mainCharacter: Character;
		colleagues: Colleague[];
		activityLog: ActivityLog[];
	}

	// 샘플 데이터 (실제 구현 시 JSON 파일이나 API에서 가져올 수 있음)
	const initialData: Data = {
		mainCharacter: {
			name: '나야나',
			level: 90,
			image: 'https://www.mangoboard.net/mgImg/com/IMG/IMG68306.png',
			skills: [
				{ name: 'Python', level: 85, description: 'FastApi, Django, LLM' },
				{ name: 'JavaScript', level: 85, description: '프론트엔드 개발' },
				{ name: 'React', level: 90, description: 'UI 컴포넌트 개발' },
				{ name: 'Node.js', level: 75, description: '백엔드 서버 개발' },
				{ name: 'TypeScript', level: 70, description: '타입 안전성 확보' },
				{ name: 'GraphQL', level: 40, description: 'API 쿼리 언어' },
				{ name: 'Docker', level: 65, description: '컨테이너화' }
			]
		},
		colleagues: [
			{
				id: 1,
				name: '디자이너 깔끄미',
				image: 'https://www.mangoboard.net/mgImg/com/IC/FIC/FIC116515.svg',
				skills: [
					{ name: 'Figma', level: 90, description: 'UI/UX 디자인 툴' },
					{ name: 'Photoshop', level: 85, description: '이미지 편집' }
				],
				description: '창의적인 UI/UX 디자이너',
				isPublic: true,
				order: 1
			},
			{
				id: 2,
				name: '개발자 노버그',
				image: 'https://www.mangoboard.net/mgImg/com/IC/FIC/FIC116515.svg',
				skills: [
					{ name: 'Java', level: 95, description: '백엔드 개발' },
					{ name: 'Spring', level: 80, description: '웹 프레임워크' }
				],
				description: '백엔드 전문 개발자',
				isPublic: true,
				order: 2
			},
			{
				id: 3,
				name: 'PM 따라와',
				image: 'https://www.mangoboard.net/mgImg/com/IC/FIC/FIC116515.svg',
				skills: [
					{ name: 'Jira', level: 85, description: '프로젝트 관리' },
					{ name: 'Agile', level: 90, description: '애자일 방법론' }
				],
				description: '효율적인 프로젝트 관리자',
				isPublic: false,
				order: 3
			},
			{
				id: 4,
				name: '마케팅 꼭집어',
				image: 'https://www.mangoboard.net/mgImg/com/IC/FIC/FIC116515.svg',
				skills: [
					{ name: 'SEO', level: 80, description: '검색 엔진 최적화' },
					{ name: '콘텐츠 마케팅', level: 90, description: '콘텐츠 전략' }
				],
				description: '디지털 마케팅 전문가',
				isPublic: true,
				order: null
			}
		],
		activityLog: [
			{
				date: '2025-02-28',
				category: '개발',
				content: 'React 컴포넌트 구조 리팩토링',
				result: '성능 30% 향상'
			},
			{
				date: '2025-02-25',
				category: '학습',
				content: 'GraphQL 기초 학습',
				result: '기본 쿼리 작성 가능'
			},
			{
				date: '2025-02-20',
				category: '협업',
				content: '디자인 팀과 UI/UX 회의',
				result: '새로운 디자인 시스템 합의'
			},
			{
				date: '2025-02-15',
				category: '개발',
				content: '사용자 인증 시스템 구현',
				result: '보안 취약점 해결'
			},
			{
				date: '2025-02-15',
				category: '개발',
				content: '사용자 인증 시스템 구현',
				result: '보안 취약점 해결'
			},
			{
				date: '2025-02-15',
				category: '개발',
				content: '사용자 인증 시스템 구현',
				result: '보안 취약점 해결'
			}
		]
	};

	let data: Data = initialData;
	let flippedCards: { [key: number]: boolean } = {};

	function flipCard(id: number): void {
		const colleague = data.colleagues.find((c) => c.id === id);
		if (colleague && colleague.isPublic) {
			flippedCards = { ...flippedCards, [id]: !flippedCards[id] };
		}
	}

	function sortedColleagues(): Colleague[] {
		const publicCards = data.colleagues.filter((c) => c.isPublic);
		const privateCards = data.colleagues.filter((c) => !c.isPublic);

		const orderedPublicCards = publicCards
			.filter((c) => c.order !== null)
			.sort((a, b) => (a.order as number) - (b.order as number));

		const unorderedPublicCards = publicCards.filter((c) => c.order === null);

		return [...orderedPublicCards, ...unorderedPublicCards, ...privateCards];
	}

	function calculateCardPosition(
		index: number,
		totalCards: number
	): { translateX: number; translateY: number; rotation: number } {
		if (totalCards === 1) {
			return { translateX: 0, translateY: 20, rotation: 0 };
		}

		// 카드 개수에 따라 반지름 조정
		const baseRadius = 250;
		const radius = Math.min(baseRadius, baseRadius * (totalCards / 5)); // 카드가 많을수록 반지름 증가 (최대 baseRadius)

		// 9시(180도)부터 3시(0도)까지 균등하게 분포
		const startAngle = Math.PI; // 9시 방향 (180도)
		const endAngle = 0; // 3시 방향 (0도)
		const angleRange = startAngle - endAngle;
		const angleIncrement = angleRange / (totalCards - 1 || 1); // 전체 호를 카드 개수로 나눔
		const angle = startAngle - index * angleIncrement;

		// 카드 위치 계산
		return {
			translateX: radius * Math.cos(angle), // x 좌표
			translateY: 20 + Math.abs(radius * Math.sin(angle) * 0.3), // y 좌표 (위에서부터 시작하여 아래로 볼록하게)
			rotation: 0 // 회전은 없앰 (필요시 조정)
		};
	}

	function getProgressColor(level: number): string {
		if (level >= 80) return 'bg-green-500';
		if (level >= 60) return 'bg-blue-500';
		if (level >= 40) return 'bg-yellow-500';
		return 'bg-red-500';
	}
</script>

<div class="w-full max-w-4xl mx-auto p-4 bg-gray-100 rounded-lg">
	<!-- 동료 카드 배치 -->
	<div class="relative h-52 mb-20">
		<div class="absolute left-0 right-0 flex justify-center" style="top: 0;">
			{#each sortedColleagues() as colleague, index (colleague.id)}
				{@const position = calculateCardPosition(index, data.colleagues.length)}
				<div
					class="absolute cursor-pointer transition-all duration-300 {!colleague.isPublic
						? 'opacity-50 pointer-events-none'
						: ''}"
					style="transform: translateX({position.translateX}px) translateY({Math.max(
						0,
						position.translateY
					)}px) rotate({position.rotation}deg); z-index: {flippedCards[colleague.id]
						? 10
						: data.colleagues.length - index}; {flippedCards[colleague.id]
						? 'margin-top: -20px;'
						: ''}"
					onclick={() => flipCard(colleague.id)}
				>
					<div
						class="relative w-36 h-52 perspective {flippedCards[colleague.id] ? 'scale-110' : ''}"
					>
						<div
							class="w-full h-full transition-all duration-500 transform-style-preserve-3d relative {flippedCards[
								colleague.id
							]
								? 'rotate-y-180'
								: ''}"
						>
							<!-- 카드 앞면 -->
							<div
								class="absolute w-full h-full backface-hidden {colleague.isPublic
									? 'bg-gradient-to-br from-indigo-500 to-purple-600'
									: 'bg-gradient-to-br from-gray-400 to-gray-600'} rounded-lg shadow-lg flex items-center justify-center"
							>
								<div class="text-white text-xl font-bold text-center p-4">
									{colleague.isPublic ? '팀원 카드' : '미공개 팀원'}
								</div>
							</div>
							<!-- 카드 뒷면 -->
							<div
								class="absolute w-full h-full backface-hidden bg-white rounded-lg shadow-lg rotate-y-180 p-3 flex flex-col"
							>
								<div class="text-center font-bold text-lg mb-2">{colleague.name}</div>
								<div class="flex justify-center mb-2">
									<img
										src={colleague.image}
										alt={colleague.name}
										class="w-16 h-16 rounded-full object-cover border-2 border-indigo-300"
									/>
								</div>
								<div class="text-sm text-center mb-2">{colleague.description}</div>
								<div class="flex-1 overflow-y-auto custom-scrollbar">
									<div class="text-sm font-semibold mb-1">스킬:</div>
									{#each colleague.skills as skill, idx}
										<div class="mb-2 text-xs">
											<div class="flex justify-between mb-1">
												<span>{skill.name}</span>
												<span>{skill.level}%</span>
											</div>
											<div class="w-full bg-gray-200 rounded-full h-1.5">
												<div
													class="h-1.5 rounded-full {getProgressColor(skill.level)}"
													style="width: {skill.level}%"
												></div>
											</div>
											<div class="text-gray-600 mt-1">{skill.description}</div>
										</div>
									{/each}
								</div>
							</div>
						</div>
					</div>
				</div>
			{/each}
		</div>
	</div>

	<!-- 메인 캐릭터 영역 -->
	<div class="flex flex-wrap md:flex-nowrap bg-white rounded-lg shadow-lg p-4 relative">
		<!-- 캐릭터 이미지와 레벨 - 가운데 정렬을 위해 flex 컨테이너 추가 -->
		<div class="w-full flex justify-center items-start">
			<!-- 캐릭터 이미지 영역 - margin 조정하여 스킬박스와 붙임 -->
			<div class="relative flex-shrink-0 mr-2">
				<img
					src={data.mainCharacter.image}
					alt={data.mainCharacter.name}
					class="h-48 object-contain"
				/>
				<div
					class="absolute -top-3 -left-3 bg-gradient-to-r from-yellow-400 to-yellow-600 text-white font-bold py-1 px-2 rounded-full transform -rotate-12 shadow-md"
				>
					Lv.{data.mainCharacter.level}
				</div>
				<div
					class="absolute bottom-0 left-0 right-0 bg-black bg-opacity-70 py-1 px-2 text-white text-center text-sm"
				>
					{data.mainCharacter.name}
				</div>
			</div>

			<!-- 스킬 박스 - 너비 조정 및 마진 제거하여 캐릭터와 붙임 -->
			<div class="flex-1 max-w-md">
				<!-- 최대 너비 제한으로 레이아웃 안정화 -->
				<div
					class="bg-gray-800 rounded-lg p-3 h-48 w-full relative shadow-lg border border-gray-700"
				>
					<div
						class="text-white text-lg font-bold mb-3 text-center border-b border-gray-600 pb-2 flex items-center justify-center"
					>
						<svg
							xmlns="http://www.w3.org/2000/svg"
							class="h-5 w-5 mr-1 text-indigo-400"
							viewBox="0 0 20 20"
							fill="currentColor"
						>
							<path
								fill-rule="evenodd"
								d="M11.3 1.046A1 1 0 0112 2v5h4a1 1 0 01.82 1.573l-7 10A1 1 0 018 18v-5H4a1 1 0 01-.82-1.573l7-10a1 1 0 011.12-.38z"
								clip-rule="evenodd"
							/>
						</svg>
						스킬 목록
					</div>
					<div class="h-32 overflow-y-auto pr-1 custom-scrollbar">
						{#each data.mainCharacter.skills as skill, index}
							<div class="mb-3 bg-gray-700 rounded p-2 hover:bg-gray-600 transition-colors shadow">
								<div class="flex justify-between mb-1">
									<span class="text-white font-medium">{skill.name}</span>
									<span class="text-white text-xs bg-gray-800 px-2 py-0.5 rounded-full"
										>{skill.level}%</span
									>
								</div>
								<div class="w-full bg-gray-600 rounded-full h-2 mb-1">
									<div
										class="h-2 rounded-full {getProgressColor(skill.level)}"
										style="width: {skill.level}%; transition: width 0.5s ease;"
									></div>
								</div>
								<div class="text-gray-300 text-xs mt-1">{skill.description}</div>
							</div>
						{/each}
					</div>
					<!-- <div class="absolute bottom-1 left-0 right-0 flex justify-center space-x-2">
						<button
							class="text-white bg-gray-600 rounded-full p-1 hover:bg-gray-500 focus:outline-none transition-colors"
						>
							<ChevronUp size="16" />
						</button>
						<button
							class="text-white bg-gray-600 rounded-full p-1 hover:bg-gray-500 focus:outline-none transition-colors"
						>
							<ChevronDown size="16" />
						</button>
					</div> -->
				</div>
			</div>
		</div>
	</div>

	<!-- 활동 일지 -->
	<div class="mt-6 bg-white rounded-lg shadow-lg p-4">
		<div class="flex items-center text-lg font-bold mb-3 border-b border-gray-200 pb-2">
			<Book class="text-indigo-600 mr-2" size="20" />
			<span>활동 및 학습 일지</span>
		</div>

		<div class="overflow-x-auto">
			<table class="min-w-full divide-y divide-gray-200">
				<thead class="bg-gray-50">
					<tr>
						<th
							scope="col"
							class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
						>
							일자
						</th>
						<th
							scope="col"
							class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
						>
							분류
						</th>
						<th
							scope="col"
							class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
						>
							내용
						</th>
						<th
							scope="col"
							class="px-4 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider"
						>
							결과
						</th>
					</tr>
				</thead>
				<tbody class="bg-white divide-y divide-gray-200">
					{#each data.activityLog as log, index}
						<tr class={index % 2 === 0 ? 'bg-gray-50' : 'bg-white'}>
							<td class="px-4 py-2 whitespace-nowrap text-sm text-gray-900 flex items-center">
								<Calendar size="14" class="text-gray-400 mr-1" />
								{new Date(log.date).toLocaleDateString('ko-KR')}
							</td>
							<td class="px-4 py-2 whitespace-nowrap">
								<span
									class="px-2 py-1 text-xs rounded-full {log.category === '개발'
										? 'bg-blue-100 text-blue-800'
										: log.category === '학습'
											? 'bg-green-100 text-green-800'
											: 'bg-purple-100 text-purple-800'}"
								>
									{log.category}
								</span>
							</td>
							<td class="px-4 py-2 text-sm text-gray-900">{log.content}</td>
							<td class="px-4 py-2 text-sm text-gray-900 font-medium">{log.result}</td>
						</tr>
					{/each}
				</tbody>
			</table>
		</div>
	</div>
</div>

<style>
	.perspective {
		perspective: 1000px;
	}
	.backface-hidden {
		backface-visibility: hidden;
	}
	.transform-style-preserve-3d {
		transform-style: preserve-3d;
	}
	.rotate-y-180 {
		transform: rotateY(180deg);
	}
	.scale-110 {
		transform: scale(1.1);
		transition: transform 0.3s ease;
	}
	.custom-scrollbar {
		max-height: 100px;
		overflow-y: auto;
	}
	.custom-scrollbar::-webkit-scrollbar {
		width: 6px;
	}
	.custom-scrollbar::-webkit-scrollbar-track {
		background: #f1f1f1;
		border-radius: 3px;
	}
	.custom-scrollbar::-webkit-scrollbar-thumb {
		background: #c0c0c0;
		border-radius: 3px;
	}
	.custom-scrollbar::-webkit-scrollbar-thumb:hover {
		background: #a0a0a0;
	}

	/* 메인 캐릭터 스킬 목록 스크롤바 스타일 */
	.h-32.custom-scrollbar::-webkit-scrollbar-track {
		background: #374151;
		border-radius: 3px;
	}
	.h-32.custom-scrollbar::-webkit-scrollbar-thumb {
		background: #4b5563;
		border-radius: 3px;
	}
	.h-32.custom-scrollbar::-webkit-scrollbar-thumb:hover {
		background: #6b7280;
	}
</style>
