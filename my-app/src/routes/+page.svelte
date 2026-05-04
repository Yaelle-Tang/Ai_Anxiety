<script>
	import { onMount } from 'svelte';
	import photo1 from '$lib/photos/Heisenberg-1.png';
	import photo2 from '$lib/photos/Heisenberg-2.png';
	import photo3 from '$lib/photos/Heisenberg-3.png';
	import photo4 from '$lib/photos/Heisenberg-4.png';
	import photo5 from '$lib/photos/Heisenberg-5.png';
	import photo6 from '$lib/photos/Heisenberg-6.png';
	import cover from '$lib/photos/cover.png';
	import screenshot from '$lib/photos/screenshot.png';
	import StarChart from '$lib/components/StarChart.svelte';
	import TokenModel from '$lib/components/TokenModel.svelte';
	import CompanyChart from '$lib/components/CompanyChart.svelte';
	import Carousel from '$lib/components/Carousel.svelte';

	const scenes = [
		{
			id: 1,
			image: photo1,
			text: 'On the evening of March 19th, Heisenberg sat down to install a piece of software that had, by then, captivated much of the Chinese internet. He gave himself 15 minutes. <mark>Five hours later, he was still trying.</mark>'
		},
		{
			id: 2,
			image: photo2,
			text: "DeepSeek's step-by-step guide had estimated <mark>15 to 20 minutes</mark>. What followed was something else. Each completed step produced a new error. Each new attempt, the same."
		},
		{
			id: 3,
			image: photo3,
			text: '\u201cEvery time it told me I was just one step away,\u201d he said. \u201cThen it would be wrong. So I\u2019d try again, still wrong, until all the enthusiasm was gone.\u201d He had been, he said, psychologically worn down by an AI. <mark>Like Sisyphus.</mark>'
		},
		{
			id: 4,
			image: photo4,
			text: 'He gave up and opened Xianyu, China\u2019s secondhand marketplace. For 9.9 CNY (about 1.25 USD) a stranger remotely accessed his machine and fixed the gateway configuration in minutes.'
		},
		{
			id: 5,
			image: photo5,
			text: `Two days later, Heisenberg asked the tool to delete a third-party iPhone management app. <mark>Instead, it wiped nearly everything on his C drive, including itself.</mark> He posted about it on Rednote: \u201cSuicidal lobster! Wiped the entire C drive, including the lobster itself.\u201d He wasn\u2019t particularly upset. \u201c10% angry at its stupidity,\u201d he said. <mark>\u201c90% shocked that it could actually do that to a computer.\u201d</mark>`
		},
		{
			id: 6,
			image: photo6,
			text: `He reinstalled the same day. And when the new agent came online, he explained, at some length, the history of its predecessor. \u201cThis is the age of great voyages,\u201d he said. <mark>\u201cThe Wild West gold rush. Who\u2019s thinking about privacy right now?\u201d</mark>`
		}
	];

	let progress = $state([0, 0, 0, 0, 0, 0]);
	let mainContentEl = $state(/** @type {HTMLElement|null} */ (null));
	let mainOffset = $state(0);
	let mainVisible = $state(false);

	onMount(() => {
		const sections = document.querySelectorAll('section.scene');

		function onScroll() {
			sections.forEach((el, i) => {
				if (!(el instanceof HTMLElement)) return;
				const rect = el.getBoundingClientRect();
				const scrolled = -rect.top;
				const total = el.offsetHeight - window.innerHeight;
				progress[i] = Math.max(0, Math.min(1, scrolled / total));
			});

			if (mainContentEl) {
				const rect = mainContentEl.getBoundingClientRect();
				mainOffset = rect.top <= window.innerHeight ? -80 : 0;
			}
		}

		window.addEventListener('scroll', onScroll, { passive: true });
		onScroll();
		const observer = new IntersectionObserver(
			([entry]) => {
				mainVisible = entry.isIntersecting;
			},
			{ threshold: 0 }
		);
		if (mainContentEl) observer.observe(mainContentEl);

		return () => {
			window.removeEventListener('scroll', onScroll);
			observer.disconnect(); // ← return 里也要加
		};

		return () => window.removeEventListener('scroll', onScroll);
	});

	function photoOpacity(i = 0) {
		if (i === 0) return 1;
		const prev = progress[i - 1] ?? 0;
		return Math.min(1, Math.max(0, (prev - 0.6) / 0.4));
	}

	function mainY(p = 0) {
		if (typeof window === 'undefined') return 0;
		const vh = window.innerHeight;
		const t = Math.min(1, Math.max(0, (p - 0.8) / 0.2));
		return Math.round(vh * (1 - t));
	}
</script>

<link rel="preconnect" href="https://fonts.googleapis.com" />
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
<link
	href="https://fonts.googleapis.com/css2?family=Jersey+15&family=Lato:wght@300;400;700&display=swap"
	rel="stylesheet"
/>

<!-- 封面 -->
<section class="cover">
	<img src={cover} alt="cover" />
	<div class="cover-text">
		<h1 class="jersey-15-regular">
			China AI Gold Rush, <br /> everyone wants to get on the table <br /><mark
				>But... At what costs?</mark
			>
		</h1>
		<p class="cover-byline">By Yaelle Tang · May 4,2026</p>
	</div>
</section>

<!-- 固定照片层 -->
<div class="photo-stack" style="visibility: {mainVisible ? 'hidden' : 'visible'};">
	{#each scenes as scene, i}
		<img src={scene.image} alt="" style="opacity: {photoOpacity(i)}; z-index: {i};" />
	{/each}
</div>

<!-- 滚动层 -->
{#each scenes as scene, i}
	<section class="scene">
		<div class="sticky-wrap" style="visibility: {mainVisible ? 'hidden' : 'visible'};">
			<div class="text-box">
				<p>{@html scene.text}</p>
			</div>
		</div>
	</section>
{/each}

<!-- 正文 -->
<section
	class="main-content"
	bind:this={mainContentEl}
	style="transform: translateY({mainOffset}vh); transition: transform 0.6s cubic-bezier(0.16,1,0.3,1);"
>
	<div class="body-text">
		<h2 class="jersey-15-regular">
			<span>Part 1</span><br />
			<mark>The Wild AI Gold Rush</mark>
		</h2>
		<p>
			OpenClaw, the tool Heisenberg spent five hours trying to install, is an open-source AI agent.
			It was built by Peter Steinberger, an Austrian developer who started it as a side project
			alongside his regular job. He released it in late January 2026.
		</p>
		<p>
			The idea behind OpenClaw is conceptually simple: it does things on your behalf. It replies to
			your emails, manages your files, browses the web, writes code. You don't need special
			software. You just use messaging apps you already have. But to do all of that, it needs
			something significant in return: deep, largely unsupervised access to your computer.
		</p>
		<p>What happened next was without precedent in the history of open-source software.</p>
		<StarChart />
		<p>
			Within weeks, "lobster farming" had become cultural shorthand for the practice of configuring,
			training, and tending to your AI agent. Communities formed on Rednote and WeChat. Tutorials
			proliferated. Offline meetups appeared seemingly overnight in Shenzhen's tech parks and
			Beijing's university districts. People traded "lobster farming techniques", how to set up
			local knowledge bases, which model providers to trust, how to stop your lobster from being
			dangerous.
		</p>
		<TokenModel />
		<p>
			The models powering most of those users were made in China, hosted on Chinese cloud
			infrastructure, such as Alibaba Cloud, MiniMax's own API, Zhipu AI's platform.
		</p>
		<p>
			Of the top 20 models running on OpenClaw, 12 were built in China, according to OpenRouter, a
			service that routes AI queries across hundreds of model providers and logs every token in
			transit.
		</p>
		<p>
			Group by provider, Anthropic led all others, but the next three spots belonged to Chinese
			companies.
		</p>
		<CompanyChart />
		<p>
			Local governments in Hangzhou's Xiaoshan District and Hefei's High-Tech Zone were offering
			large subsidies to companies building OpenClaw-based products, hoping to brand themselves as
			AI destinations. The central government was ordering it removed. The local governments were
			paying people to build with it.
		</p>
		<p>
			At the same time, the central government moved in the opposite direction. China's Ministry of
			Industry and Information Technology had issued a formal risk advisory on Feb. 5, warning that
			even fully updated versions of the tool carried security vulnerabilities. It explicitly
			cautioned Party organizations, government agencies, and affiliated enterprises against
			deploying the software.
		</p>
		<p>
			On March 11, the same week that the corporate competition hit its peak, Bloomberg reported
			that major state-owned enterprises, including large banks, had received internal notices to
			disable existing installations immediately and submit them for security review.
		</p>
		<p>
			Heisenberg is a government employee who had read the advisories. When asked about these
			notices, he was untroubled. "I installed it on my own computer," he said. "And what those
			notices said was: pay attention to security. They didn't say installation is prohibited."
		</p>
		<h2 class="jersey-15-regular">
			<span>Part 2</span><br />
			<mark>Where is the Safety Net for AI Security?</mark>
		</h2>
		<p>
			The STRIKE Team at SecurityScorecard, a New York-based cybersecurity firm, has been tracking
			an expanding map of risk. They built a real-time threat intelligence platform called DECLAWED,
			designed to monitor the global exposure of AI agent control panels.
		</p>
		<figure>
			<img src={screenshot} alt="declawed map" style="width:100%;" />
			<figcaption>
				Geographic distribution of exposed AI agent control panels worldwide. Screenshot captured
				April 10, 2026. Source: SecurityScorecard STRIKE Team / DECLAWED (declawed.io)
			</figcaption>
		</figure>
		<p>
			The data shows that to April 10, 216,257 instances across 99 countries have been identified.
			Among all these countries, China has the highest number of exposed instances.
		</p>
		<p>
			"Exposed" means these systems have no access restrictions whatsoever. Anyone who knows the
			address can walk right in. What's stored inside these control panels often includes API keys,
			conversation logs, and the complete configuration of an AI agent.
		</p>
		<p>
			"It's the Wild West," said Ann Cleaveland, executive director at the Center for Long-Term
			Cybersecurity from the University of California, Berkeley. "And I think, unfortunately, for a
			long time, we are going to see a lot of security risk with agentic AI."
		</p>
		<p>The risk that accumulates with each of those handovers is not a new problem.</p>
		<p>
			"It's a very old issue in cybersecurity… we tend to put disproportionate risk onto end users,
			particularly end users who may not have the expertise or the resources to properly secure
			themselves."
		</p>
		<p>
			The small businesses, schools, and local nonprofits she studies in the United States face the
			same structural problem: expected to defend themselves against sophisticated attackers with no
			IT staff and no security budget. OpenClaw's crazy spread in China is, in her framing, the same
			pressure at a different scale and speed.
		</p>
		<p>
			When something goes wrong, the question of who is responsible has no clean answer. “Even if we
			had product liability law,” she said, “the handoffs between a manufacturer, an integrator, the
			software developer, the end user, and who's responsible for what configurations, are complex
			and confusing. There aren't clear frameworks for those handoffs.”
		</p>
		<p>
			She compared the situation to the automobile industry in its early decades. “[Now] the
			government also invested in safer roads and interstates,” she said. "And we also have an
			entire insurance industry for automobile accidents dedicated to what do you do when something
			goes wrong. And literally decades and decades of legal precedent around who's at fault.
		</p>
		<p>
			But for AI agents, we still in the early stages."“Any of those multifaceted solutions in place
			that together create a safety net, but none of it exists yet.”
		</p>
		<p>
			What would help, she said, is not one thing but many arriving at once: government regulation,
			product liability frameworks, user education, and professional standards for the integrators,
			those who remotely access strangers' computers and configure their permissions for a fee.
		</p>
		<h2 class="jersey-15-regular">
			<span>Part 3</span><br />
			<mark>"I would rather take the risks than be left behind."</mark>
		</h2>
		<p>
			"We see a lot of pressure, not just in China, but also in the US and around the world, to
			adopt AI quickly," Cleaveland said.
		</p>
		<p>
			In China, that pressure had a specific texture. Millions of users were swept up by two
			overlapping fears: AI would take their jobs even though it might not be happening now, and
			that they were already falling behind a world moving faster than they could follow, sooner or
			later. It was a generational anxiety, the feeling that an entire era was shifting beneath
			their feet, and the window to get on board was closing.
		</p>
		<p>
			Each person who rushed to install OpenClaw came with their own version of that calculation.
			What they shared was the feeling that waiting was not an option.
		</p>
		<Carousel />
		<p>
			Across four stories, the reasons for adopting OpenClaw differed. But the engine behind each
			decision was the same: the fear of being left behind. They were swept forward not by
			conviction, but by the feeling that stopping was more dangerous than moving.
		</p>
		<p>
			The heat wave of OpenClaw might fade away, but something else will take its place, asking for
			deeper access, offering greater convenience, arriving faster than the last. When it does, the
			cycle will face a harder question than who is responsible or how to regulate it. If everyone
			has already learned to accept risk without measuring it, if information cynicism has become
			the default, then the gap between what people understand and what they choose to act on will
			only widen.
		</p>
	</div>
</section>

<!-- <div class="footer"></div> -->

<style>
	/* ── Fonts ── */
	.jersey-15-regular {
		font-family: 'Jersey 15', sans-serif;
		font-weight: 400;
	}

	/* ── Reset ── */
	:global(body) {
		margin: 0;
		padding: 0;
	}

	/* ── Cover ── */
	.cover {
		position: relative;
		z-index: 1;
		height: 100vh;
		display: flex;
		align-items: center;
		justify-content: center;
		/* background: rgba(0, 128, 0, 0.369); */
	}

	.cover img {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		object-fit: cover;
		object-position: center;
		/* opacity: 0.3; */
	}

	.cover-text {
		position: relative;
		z-index: 10;
		text-align: center;
		color: white;
		width: 60%;
		background: #13062bc7;
	}

	.cover-text h1 {
		font-size: clamp(2.5rem, 6vw, 4rem);
		margin-bottom: 1rem;
		margin-top: 1rem;
	}

	:global(.cover-text h1 mark) {
		background: linear-gradient(120deg, #ff2a66 0%, #ff2a66 100%) no-repeat;
		background-size: 100% 40%;
		background-position: 0 85%;
		color: inherit;
	}

	.cover-byline {
		font-size: 1.5rem;
		font-weight: 300;
		letter-spacing: 0.1em;
		font-family: 'Lato', Arial, Helvetica, sans-serif;
	}

	/* ── Photo stack ── */
	.photo-stack {
		position: fixed;
		inset: 0;
		z-index: 0;
	}

	.photo-stack img {
		position: absolute;
		inset: 0;
		width: 100%;
		height: 100%;
		object-fit: cover;
		object-position: center;
	}

	/* ── Scene sections ── */
	section.scene {
		height: 150vh;
		overflow: hidden;
	}

	.sticky-wrap {
		position: sticky;
		top: 0;
		height: 100vh;
		width: 100vw;
		overflow: hidden;
		z-index: 10;
		/* background: rgba(255, 192, 203, 0.605); */
	}

	/* ── Text box ── */
	.text-box {
		position: absolute;
		top: 50%;
		left: 50%;
		translate: -50% -50%;
		width: min(600px, 85vw);
		background: rgba(0, 0, 0, 0.85);
		color: white;
		padding: 10px 20px;
		border-radius: 4px;
		z-index: 10;
	}

	.text-box p {
		font-family: 'Lato', Arial, Helvetica, sans-serif;
		font-size: 1.5rem;
		line-height: 2.4rem;
		text-align: justify;
	}

	:global(.text-box mark) {
		background: linear-gradient(120deg, #ff2a66 0%, #ff2a66 100%) no-repeat;
		background-size: 100% 40%;
		background-position: 0 85%;
		color: inherit;
	}

	/* ── Main content ── */
	.main-content {
		position: relative;
		top: 0;
		left: 0;
		width: 100%;
		z-index: 20;
		background: #13062b;
		/* background: rgba(0, 128, 0, 0.246); */
	}

	.body-text {
		width: 70%;
		max-width: 800px;
		margin: 0 auto;
		padding: 20vh 0;
		color: white;
		font-family: 'Lato', Arial, Helvetica, sans-serif;
		font-size: 1.5rem;
		line-height: 2.5rem;
	}

	.body-text p {
		margin-bottom: 1.5rem;
		color: rgba(255, 255, 255, 0.8);
	}

	h2 {
		font-size: 2.4rem;
		margin: 0 0 2rem;
	}

	h2 span {
		font-size: 3rem;
		color: #ff2a66;
	}

	h2 mark {
		background: linear-gradient(120deg, #ff2a66 0%, #ff2a66 100%) no-repeat;
		background-size: 100% 40%;
		background-position: 0 85%;
		color: inherit;
	}

	.chart-container {
		display: flex;
		gap: 2rem;
		margin-top: 3rem;
		background: darkblue;
	}

	figure {
		width: 100%;
		margin: 0;
	}

	figcaption {
		font-size: 0.85rem;
		color: rgba(255, 255, 255, 0.5);
		margin-top: 6px;
		text-align: right;
		line-height: 1rem;
	}

	/* .footer {
		height: 1000vh;
		width: 100%;
		background: pink;
		transform: translateY(-100vh);
	} */
</style>
