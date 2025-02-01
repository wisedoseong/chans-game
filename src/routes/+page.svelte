<script lang="ts">
	import { onMount } from 'svelte';
	//import { FloatingChat } from '$lib/components/ui/chat';
	let isChatOpen = false;
	let isMenuOpen = false;

	// 모바일 메뉴 토글 시 body에 overflow-hidden 클래스 추가/제거
	const toggleMenu = () => {
		isMenuOpen = !isMenuOpen;
		if (isMenuOpen) {
			document.body.classList.add('overflow-hidden');
		} else {
			document.body.classList.remove('overflow-hidden');
		}
	};

	// Smooth scroll function (메뉴 선택 후에도 overflow-hidden 제거)
	const smoothScroll = (id: string) => {
		const element = document.getElementById(id);
		if (element) {
			element.scrollIntoView({ behavior: 'smooth' });
			isMenuOpen = false;
			document.body.classList.remove('overflow-hidden');
		}
	};

	// 혹시 모를 초기 overflow 상태 제거 (예: 라우트 변경 시)
	onMount(() => {
		document.body.classList.remove('overflow-hidden');
	});
</script>

<svelte:head>
	<title>AI Enterprise Solutions | Global Technology Transformation</title>
	<meta
		name="description"
		content="Enterprise AI integration, data migration solutions, and custom application development for global businesses."
	/>
</svelte:head>

<main class="min-h-screen bg-gradient-to-b from-blue-50 to-white">
	<!-- Enhanced Header with Mobile Menu -->
	<header class="sticky top-0 z-50 bg-white/90 backdrop-blur-sm shadow-sm">
		<nav class="container mx-auto px-4 py-4 flex justify-between items-center">
			<div class="flex items-center space-x-2">
				<div class="h-8 w-8 bg-gradient-to-r from-blue-600 to-blue-400 rounded-lg"></div>
				<span class="text-xl font-bold text-gray-900">Global AI Solutions</span>
			</div>

			<!-- Desktop Navigation -->
			<div class="hidden md:flex space-x-8">
				<button
					onclick={() => smoothScroll('data-migration')}
					class="text-gray-600 hover:text-blue-600 transition-colors">Data Migration</button
				>
				<button
					onclick={() => smoothScroll('custom-apps')}
					class="text-gray-600 hover:text-blue-600 transition-colors">Custom Solutions</button
				>
				<button
					onclick={() => smoothScroll('ai-consulting')}
					class="text-gray-600 hover:text-blue-600 transition-colors">AI Consulting</button
				>
				<button
					onclick={() => smoothScroll('contact')}
					class="text-gray-600 hover:text-blue-600 transition-colors">Contact</button
				>
			</div>

			<!-- Mobile Menu Toggle -->
			<button onclick={toggleMenu} class="md:hidden p-2" aria-label="Toggle Menu">
				<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
					<path
						stroke-linecap="round"
						stroke-linejoin="round"
						stroke-width="2"
						d="M4 6h16M4 12h16M4 18h16"
					/>
				</svg>
			</button>
		</nav>

		<!-- Mobile Menu -->
		{#if isMenuOpen}
			<div class="md:hidden absolute top-full w-full bg-white shadow-lg">
				<div class="px-4 py-4 space-y-4">
					<button onclick={() => smoothScroll('data-migration')} class="block w-full text-left"
						>Data Migration</button
					>
					<button onclick={() => smoothScroll('custom-apps')} class="block w-full text-left"
						>Custom Solutions</button
					>
					<button onclick={() => smoothScroll('ai-consulting')} class="block w-full text-left"
						>AI Consulting</button
					>
					<button onclick={() => smoothScroll('contact')} class="block w-full text-left"
						>Contact</button
					>
				</div>
			</div>
		{/if}
	</header>

	<!-- Hero Section -->
	<section class="container mx-auto px-4 py-20 md:py-32">
		<div class="grid md:grid-cols-2 gap-12 items-center">
			<div>
				<p class="text-blue-600 font-semibold uppercase mb-4">Next-Gen Enterprise Solutions</p>
				<h1 class="text-5xl md:text-6xl font-bold text-gray-900 leading-tight mb-6">
					AI-Driven Business <span class="text-blue-600">Transformation</span>
				</h1>
				<div class="flex space-x-4 mb-8">
					<a
						href="#contact"
						class="px-8 py-4 bg-blue-600 text-white font-semibold rounded-lg shadow-xl hover:bg-blue-700 transition-all hover:shadow-2xl"
					>
						Schedule Consultation
					</a>
				</div>
				<div class="flex items-center space-x-6 opacity-75">
					<span class="text-sm font-medium">Trusted by industry leaders:</span>
					<div class="flex space-x-4">
						<div class="h-8 w-8 bg-gradient-to-r from-blue-500 to-blue-600 rounded-full"></div>
						<div class="h-8 w-8 bg-gradient-to-r from-purple-500 to-purple-600 rounded-full"></div>
						<div
							class="h-8 w-8 bg-gradient-to-r from-emerald-500 to-emerald-600 rounded-full"
						></div>
					</div>
				</div>
			</div>
			<div class="bg-gradient-to-br from-blue-100 to-white p-8 rounded-2xl shadow-2xl">
				<!-- Placeholder for 3D/Interactive demo -->
				<div class="aspect-video bg-gray-100 rounded-xl flex items-center justify-center">
					<span class="text-gray-400">AI Infrastructure Demo</span>
				</div>
			</div>
		</div>
	</section>

	<!-- Solutions Showcase -->
	<section class="py-20 bg-gradient-to-b to-blue-50 from-white">
		<div class="container mx-auto px-4">
			<div class="text-center mb-16">
				<p class="text-blue-600 font-semibold uppercase mb-3">Our Expertise</p>
				<h2 class="text-4xl font-bold text-gray-900 mb-8">Transforming Business Ecosystems</h2>
			</div>
			<div class="grid md:grid-cols-3 gap-8">
				<div class="p-8 bg-white rounded-xl shadow-lg hover:shadow-xl transition-shadow">
					<div class="w-12 h-12 bg-blue-100 rounded-lg mb-6 flex items-center justify-center">
						<span class="text-blue-600 text-2xl">🏬</span>
					</div>
					<h3 class="text-xl font-bold mb-4">E-commerce Systems</h3>
					<p class="text-gray-600">
						AI-powered inventory management and personalized shopping experiences
					</p>
				</div>
				<!-- Add more solution cards -->
			</div>
		</div>
	</section>

	<!-- Core Services -->
	<section class="bg-white py-20">
		<div class="container mx-auto px-4">
			<div class="text-center mb-16">
				<p class="text-blue-600 font-semibold uppercase mb-3">What We Offer</p>
				<h2 class="text-4xl font-bold text-gray-900 mb-8">Enterprise-Grade Solutions</h2>
			</div>

			<div class="grid md:grid-cols-3 gap-8">
				<div class="p-8 bg-white rounded-xl shadow-lg hover:shadow-xl transition-shadow">
					<div class="w-12 h-12 bg-blue-100 rounded-lg mb-6 flex items-center justify-center">
						<span class="text-blue-600 text-2xl">⚙️</span>
					</div>
					<h3 class="text-xl font-bold mb-4">AI Infrastructure Design</h3>
					<p class="text-gray-600">
						End-to-end implementation of scalable AI ecosystems with automated MLOps pipelines and
						enterprise-grade security
					</p>
				</div>

				<div class="p-8 bg-white rounded-xl shadow-lg hover:shadow-xl transition-shadow">
					<div class="w-12 h-12 bg-blue-100 rounded-lg mb-6 flex items-center justify-center">
						<span class="text-blue-600 text-2xl">📊</span>
					</div>
					<h3 class="text-xl font-bold mb-4">Process Optimization</h3>
					<p class="text-gray-600">
						Custom ERP integrations and legacy system modernization using microservices architecture
						and cloud-native solutions
					</p>
				</div>

				<div class="p-8 bg-white rounded-xl shadow-lg hover:shadow-xl transition-shadow">
					<div class="w-12 h-12 bg-blue-100 rounded-lg mb-6 flex items-center justify-center">
						<span class="text-blue-600 text-2xl">🧠</span>
					</div>
					<h3 class="text-xl font-bold mb-4">LLM Research & Development</h3>
					<p class="text-gray-600">
						Domain-specific language model training and AI-powered workflow automation with
						continuous performance monitoring
					</p>
				</div>
			</div>
		</div>
	</section>

	<!-- Legacy Modernization -->
	<section class="py-20 bg-gray-50">
		<div class="container mx-auto px-4">
			<div class="grid md:grid-cols-2 gap-12 items-center">
				<div class="space-y-6">
					<p class="text-blue-600 font-semibold uppercase mb-2">System Modernization</p>
					<h2 class="text-4xl font-bold text-gray-900">Future-Proof Your Technology Stack</h2>
					<ul class="space-y-4">
						<li class="flex items-center space-x-3">
							<div
								class="w-6 h-6 bg-blue-600 text-white rounded-full flex items-center justify-center"
							>
								✓
							</div>
							<span class="text-gray-700">Legacy system analysis & technical debt assessment</span>
						</li>
						<!-- ... more list items ... -->
					</ul>
				</div>
				<div class="bg-gradient-to-br from-blue-100 to-white p-8 rounded-2xl shadow-xl">
					<!-- Modernization process visualization -->
					<div class="aspect-video bg-gray-100 rounded-xl flex items-center justify-center">
						<span class="text-gray-400">Modernization Roadmap</span>
					</div>
				</div>
			</div>
		</div>
	</section>

	<!-- Blog Section -->
	<section id="blog" class="py-20 bg-white">
		<div class="container mx-auto px-4">
			<div class="text-center mb-16">
				<p class="text-blue-600 font-semibold uppercase mb-3">Latest Insights</p>
				<h2 class="text-4xl font-bold text-gray-900 mb-8">AI Innovation Blog</h2>
			</div>
			<div class="grid md:grid-cols-3 gap-8">
				<article
					class="group overflow-hidden rounded-xl shadow-lg hover:shadow-2xl transition-shadow"
				>
					<div class="aspect-video bg-gray-100"></div>
					<div class="p-6">
						<time class="text-sm text-gray-500">March 15, 2024</time>
						<h3 class="text-xl font-bold mt-2 mb-3">LLM Integration Patterns</h3>
						<p class="text-gray-600 line-clamp-3">
							Exploring modern approaches to enterprise AI integration...
						</p>
						<a href="None" class="mt-4 inline-flex items-center text-blue-600 hover:text-blue-800">
							Read More
							<svg class="w-4 h-4 ml-2" fill="none" stroke="currentColor" viewBox="0 0 24 24">
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M9 5l7 7-7 7"
								/>
							</svg>
						</a>
					</div>
				</article>
				<!-- Add more blog posts -->
			</div>
		</div>
	</section>

	<!-- Data Migration Section -->
	<section id="data-migration" class="py-20 bg-white">
		<div class="container mx-auto px-4">
			<div class="text-center mb-16">
				<p class="text-blue-600 font-semibold uppercase mb-3">Seamless Data Transition</p>
				<h2 class="text-4xl font-bold text-gray-900 mb-8">Intelligent Data Migration</h2>
				<p class="text-gray-600 max-w-3xl mx-auto">
					AI-powered data migration across heterogeneous systems with automated validation.
					Preserving data integrity during platform transitions.
				</p>
			</div>

			<div class="grid md:grid-cols-4 gap-8">
				{#each ['Shopify', 'Amazon', 'Naver', 'Cafe24'] as platform}
					<div class="p-6 bg-blue-50 rounded-xl hover:scale-105 transition-transform">
						<div class="aspect-square bg-gray-100 rounded-lg mb-4"></div>
						<h3 class="text-xl font-semibold">{platform} Migration</h3>
						<p class="text-gray-600 mt-2">Automated schema conversion & data validation</p>
					</div>
				{/each}
			</div>
		</div>
	</section>

	<!-- Custom Solutions Section -->
	<section id="custom-apps" class="py-20 bg-gray-50">
		<div class="container mx-auto px-4">
			<div class="text-center mb-16">
				<p class="text-blue-600 font-semibold uppercase mb-3">Tailored Solutions</p>
				<h2 class="text-4xl font-bold text-gray-900 mb-8">Enterprise Application Development</h2>
			</div>

			<div class="grid md:grid-cols-2 gap-8">
				<div class="p-8 bg-white rounded-xl shadow-lg">
					<h3 class="text-2xl font-bold mb-4">E-commerce Systems</h3>
					<ul class="space-y-4 text-gray-600">
						<li class="flex items-center space-x-3">
							<svg
								class="w-5 h-5 text-green-500"
								fill="none"
								stroke="currentColor"
								viewBox="0 0 24 24"
							>
								<path
									stroke-linecap="round"
									stroke-linejoin="round"
									stroke-width="2"
									d="M5 13l4 4L19 7"
								/>
							</svg>
							<span>AI-powered inventory optimization</span>
						</li>
						<!-- Add more list items -->
					</ul>
				</div>

				<div class="grid grid-cols-2 gap-4">
					{#each ['CRM Analytics', 'Appointment System', 'Coupon Management', 'Logistics AI'] as solution}
						<div class="p-6 bg-white rounded-xl hover:shadow-lg transition-shadow">
							<h4 class="font-semibold mb-2">{solution}</h4>
							<p class="text-sm text-gray-600">Custom implementation for your workflow</p>
						</div>
					{/each}
				</div>
			</div>
		</div>
	</section>

	<!-- AI Consulting Section -->
	<section id="ai-consulting" class="py-20 bg-white">
		<div class="container mx-auto px-4">
			<div class="text-center mb-16">
				<p class="text-blue-600 font-semibold uppercase mb-3">AI Integration</p>
				<h2 class="text-4xl font-bold text-gray-900 mb-8">Domain-Specific Language Models</h2>
			</div>

			<div class="max-w-5xl mx-auto grid md:grid-cols-3 gap-8">
				<div class="p-8 bg-gradient-to-b from-blue-50 to-white rounded-xl shadow-lg">
					<h3 class="text-xl font-bold mb-4">1. Legacy Analysis</h3>
					<p class="text-gray-600">Technical debt assessment and modernization roadmap</p>
				</div>
				<!-- Add more steps -->
			</div>
		</div>
	</section>

	<!-- Global Network Section -->
	<section class="py-20 bg-gray-900 text-white">
		<div class="container mx-auto px-4">
			<div class="text-center mb-16">
				<p class="text-blue-400 font-semibold uppercase mb-3">Worldwide Presence</p>
				<h2 class="text-4xl font-bold mb-8">Global Research & Development Network</h2>
			</div>
			<div class="grid md:grid-cols-4 gap-8 text-center">
				{#each ['Seoul', 'Silicon Valley', 'Berlin', 'Singapore'] as location}
					<div class="p-6 bg-gray-800 rounded-xl">
						<h3 class="text-xl font-semibold mb-2">{location}</h3>
						<p class="text-gray-400">R&D Center</p>
					</div>
				{/each}
			</div>
		</div>
	</section>

	<!-- Footer -->
	<footer class="bg-gray-900 text-gray-300 py-12">
		<div class="container mx-auto px-4">
			<div class="grid md:grid-cols-4 gap-8 mb-8">
				<div>
					<h4 class="text-white font-semibold mb-4">AI Solutions</h4>
					<p class="text-sm">123 Tech Valley</p>
					<p class="text-sm">Seoul, South Korea</p>
				</div>
				<div>
					<h4 class="text-white font-semibold mb-4">Services</h4>
					<ul class="space-y-2 text-sm">
						<li>
							<a href="None" class="hover:text-blue-400 transition-colors">ERP Modernization</a>
						</li>
						<li><a href="None" class="hover:text-blue-400 transition-colors">AI Integration</a></li>
					</ul>
				</div>
				<div>
					<h4 class="text-white font-semibold mb-4">Legal</h4>
					<!-- Legal links -->
				</div>
				<div>
					<h4 class="text-white font-semibold mb-4">Connect</h4>
					<!-- Social links -->
				</div>
			</div>
			<div class="border-t border-gray-800 pt-8 text-center text-sm">
				<p>&copy; 2024 AI Solutions. All rights reserved.</p>
			</div>
		</div>
	</footer>

	<!-- Floating Chat -->
	<div class="fixed bottom-8 right-8 z-50">
		<button
			onclick={() => (isChatOpen = !isChatOpen)}
			class="p-4 bg-blue-600 text-white rounded-full shadow-xl hover:bg-blue-700 transition-all"
			aria-label="Chat with AI"
		>
			<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path
					stroke-linecap="round"
					stroke-linejoin="round"
					stroke-width="2"
					d="M8 12h.01M12 12h.01M16 12h.01M21 12c0 4.418-4.03 8-9 8a9.863 9.863 0 01-4.255-.949L3 20l1.395-3.72C3.512 15.042 3 13.574 3 12c0-4.418 4.03-8 9-8s9 3.582 9 8z"
				/>
			</svg>
		</button>

		{#if isChatOpen}
			<div class="absolute bottom-20 right-0 w-80 bg-white rounded-xl shadow-2xl">
				<div class="p-4 border-b border-gray-200 flex justify-between items-center">
					<h3 class="font-semibold">AI Assistant</h3>
					<button onclick={() => (isChatOpen = false)} class="text-gray-500 hover:text-gray-700">
						&times;
					</button>
				</div>
				<div class="h-64 p-4 overflow-y-auto">
					<!-- Chat messages -->
					<div class="text-sm text-gray-600">How can we help you today?</div>
				</div>
				<div class="p-4 border-t border-gray-200">
					<input
						type="text"
						placeholder="Type your message..."
						class="w-full px-3 py-2 border rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
					/>
				</div>
			</div>
		{/if}
	</div>
</main>
