<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	const data = [
		{
			rank: 1,
			model_name: 'Qwen3.6 Plus',
			provider: 'Qwen',
			tokens_raw: 1.01e12,
			tokens: '1.01T',
			country: 'China'
		},
		{
			rank: 2,
			model_name: 'GLM 5 Turbo',
			provider: 'Z-ai',
			tokens_raw: 9.89e11,
			tokens: '989B',
			country: 'China'
		},
		{
			rank: 3,
			model_name: 'MiniMax M2.7',
			provider: 'Minimax',
			tokens_raw: 9.33e11,
			tokens: '933B',
			country: 'China'
		},
		{
			rank: 4,
			model_name: 'Claude Sonnet 4.6',
			provider: 'Anthropic',
			tokens_raw: 8.74e11,
			tokens: '874B',
			country: 'USA'
		},
		{
			rank: 5,
			model_name: 'Nemotron 3 Super',
			provider: 'Nvidia',
			tokens_raw: 7.77e11,
			tokens: '777B',
			country: 'USA'
		},
		{
			rank: 6,
			model_name: 'Step 3.5 Flash',
			provider: 'Stepfun',
			tokens_raw: 6.71e11,
			tokens: '671B',
			country: 'China'
		},
		{
			rank: 7,
			model_name: 'Claude Opus 4.6',
			provider: 'Anthropic',
			tokens_raw: 5.96e11,
			tokens: '569B',
			country: 'USA'
		},
		{
			rank: 8,
			model_name: 'Gemini 2.5 Flash Lite',
			provider: 'Google',
			tokens_raw: 4.47e11,
			tokens: '447B',
			country: 'USA'
		},
		{
			rank: 9,
			model_name: 'MiniMax M2.5',
			provider: 'Minimax',
			tokens_raw: 4.46e11,
			tokens: '446B',
			country: 'China'
		},
		{
			rank: 10,
			model_name: 'Gemini 3 Flash Preview',
			provider: 'Google',
			tokens_raw: 3.27e11,
			tokens: '327B',
			country: 'USA'
		},
		{
			rank: 11,
			model_name: 'MiMo-V2-Pro',
			provider: 'Xiaomi',
			tokens_raw: 3.21e11,
			tokens: '321B',
			country: 'China'
		},
		{
			rank: 12,
			model_name: 'DeepSeek V3.2',
			provider: 'Deepseek',
			tokens_raw: 2.95e11,
			tokens: '285B',
			country: 'China'
		},
		{
			rank: 13,
			model_name: 'Elephant Alpha',
			provider: 'Openrouter',
			tokens_raw: 2.39e11,
			tokens: '239B',
			country: 'USA'
		},
		{
			rank: 14,
			model_name: 'Claude Opus 4.7',
			provider: 'Anthropic',
			tokens_raw: 2.23e11,
			tokens: '223B',
			country: 'USA'
		},
		{
			rank: 15,
			model_name: 'Trinity Large Preview',
			provider: 'Arcee-ai',
			tokens_raw: 1.96e11,
			tokens: '196B',
			country: 'USA'
		},
		{
			rank: 16,
			model_name: 'Kimi K2.5',
			provider: 'Moonshotai',
			tokens_raw: 1.95e11,
			tokens: '195B',
			country: 'China'
		},
		{
			rank: 17,
			model_name: 'Gemini 2.5 Flash',
			provider: 'Google',
			tokens_raw: 1.92e11,
			tokens: '192B',
			country: 'USA'
		},
		{
			rank: 18,
			model_name: 'MiMo-V2-Flash',
			provider: 'Xiaomi',
			tokens_raw: 1.78e11,
			tokens: '178B',
			country: 'China'
		},
		{
			rank: 19,
			model_name: 'gpt-oss-120b',
			provider: 'Openai',
			tokens_raw: 1.74e11,
			tokens: '174B',
			country: 'USA'
		},
		{
			rank: 20,
			model_name: 'Claude Haiku 4.5',
			provider: 'Anthropic',
			tokens_raw: 1.72e11,
			tokens: '172B',
			country: 'USA'
		}
	];

	let container;
	let tooltip = { visible: false, x: 0, y: 0, text: '' };

	const CHINA_COLOR = '#FF2A66';
	const USA_COLOR = '#1DD2DF';
	const BG_COLOR = '#13062B';

	onMount(() => {
		const margin = { top: 16, right: 72, bottom: 8, left: 172 };
		const totalWidth = container.clientWidth;
		const barHeight = 26;
		const barPadding = 8;
		const innerHeight = data.length * (barHeight + barPadding);
		const innerWidth = totalWidth - margin.left - margin.right;

		const svg = d3
			.select(container)
			.append('svg')
			.attr('width', totalWidth)
			.attr('height', innerHeight + margin.top + margin.bottom);

		const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);

		// Scales
		const xScale = d3
			.scaleLinear()
			.domain([0, d3.max(data, (d) => d.tokens_raw)])
			.range([0, innerWidth]);

		const yScale = d3
			.scaleBand()
			.domain(data.map((d) => d.model_name))
			.range([0, innerHeight])
			.paddingInner(barPadding / (barHeight + barPadding))
			.paddingOuter(0);

		// 创建 tooltip DOM，挂到 body
		const tip = d3
			.select('body')
			.append('div')
			.style('position', 'fixed')
			.style('display', 'none')
			.style('background', 'rgba(19,6,43,0.92)')
			.style('border', '1px solid rgba(255,255,255,0.18)')
			.style('color', 'white')
			.style('padding', '8px 12px')
			.style('border-radius', '6px')
			.style('font-size', '12px')
			.style('font-family', 'Inter, system-ui, sans-serif')
			.style('pointer-events', 'none')
			.style('z-index', '9999')
			.style('line-height', '1.7');

		// Y axis — model names only, no domain line, no ticks
		g.append('g')
			.call(d3.axisLeft(yScale).tickSize(0).tickPadding(10))
			.call((ax) => ax.select('.domain').remove())
			.selectAll('text')
			.style('fill', 'white')
			.style('font-size', '12px')
			.style('font-family', 'Inter, system-ui, sans-serif')
			.attr('text-anchor', 'end');

		// Bars
		g.selectAll('.bar')
			.data(data)
			.join('rect')
			.attr('class', 'bar')
			.attr('x', 0)
			.attr('y', (d) => yScale(d.model_name))
			.attr('width', (d) => xScale(d.tokens_raw))
			.attr('height', yScale.bandwidth())
			.attr('fill', (d) => (d.country === 'China' ? CHINA_COLOR : USA_COLOR))
			.attr('rx', 3)
			.style('cursor', 'crosshair')
			.on('mousemove', (event, d) => {
				tip
					.style('display', 'block')
					.style('left', `${event.clientX + 14}px`)
					.style('top', `${event.clientY - 36}px`)
					.text(d.provider)
					.html(`Model Provider: ${d.provider}`);
			})
			.on('mouseleave', () => {
				tip.style('display', 'none');
			});
		// Token labels — right of each bar
		g.selectAll('.label')
			.data(data)
			.join('text')
			.attr('class', 'label')
			.attr('x', (d) => xScale(d.tokens_raw) + 8)
			.attr('y', (d) => yScale(d.model_name) + yScale.bandwidth() / 2)
			.attr('dy', '0.35em')
			.style('fill', 'white')
			.style('font-size', '11px')
			.style('font-family', 'Inter, system-ui, sans-serif')
			.style('font-weight', '500')
			.text((d) => d.tokens);

		// Rank numbers — left of model names
		g.selectAll('.rank')
			.data(data)
			.join('text')
			.attr('class', 'rank')
			.attr('x', -margin.left + 4)
			.attr('y', (d) => yScale(d.model_name) + yScale.bandwidth() / 2)
			.attr('dy', '0.35em')
			.style('fill', 'rgba(255,255,255,0.35)')
			.style('font-size', '11px')
			.style('font-family', 'Inter, system-ui, sans-serif')
			.text((d) => `#${d.rank}`);

		return () => tip.remove();
	});
</script>

<!-- Tooltip -->
{#if tooltip.visible}
	<div class="tooltip" style="left: {tooltip.x + 14}px; top: {tooltip.y - 36}px">
		{tooltip.text}
	</div>
{/if}

<!-- Chart wrapper -->
<div class="wrapper">
	<!-- Title -->
	<h2 class="title">Top 20 Models by Token Usage on OpenClaw, April-May, 2026</h2>

	<!-- Legend -->
	<div class="legend">
		<span class="dot" style="background:{CHINA_COLOR}"></span><span>China</span>
		<span class="dot" style="background:{USA_COLOR}; margin-left:16px"></span><span>USA</span>
	</div>

	<div bind:this={container} class="chart"></div>

	<!-- Footer -->
	<div class="footer">
		<p class="note">Token usage by model, Apr. 3 – May 3, 2026</p>
		<p class="source">
			Source: <a href="https://openrouter.ai/apps/openclaw" target="_blank" rel="noopener"
				>OpenRouter</a
			>
		</p>
	</div>
</div>

<style>
	:global(body) {
		margin: 0;
		background: #13062b;
	}

	.wrapper {
		background: #13062b;
		padding: 24px 24px 16px;
		height: max-content;
		box-sizing: border-box;
	}

	.title {
		color: white;
		font-size: 2rem;
		font-family: Inter, system-ui, sans-serif;
		font-weight: 600;
		margin: 0 0 14px 0;
		line-height: 1.4;
		width: 100%;
	}

	.footer {
		margin-top: 0;
		display: flex;
		flex-direction: column;
	}

	.note,
	.source {
		margin: 0;
		font-size: 11px;
		font-family: Inter, system-ui, sans-serif;
		color: rgba(255, 255, 255, 0.4);
		line-height: 18px;
	}

	.source a {
		color: rgba(255, 255, 255, 0.55);
		text-decoration: underline;
		text-underline-offset: 2px;
	}

	.source a:hover {
		color: white;
	}

	.legend {
		display: flex;
		align-items: center;
		gap: 6px;
		margin-bottom: 16px;
		color: white;
		font-size: 12px;
		font-family: Inter, system-ui, sans-serif;
		opacity: 0.75;
	}

	.dot {
		display: inline-block;
		width: 10px;
		height: 10px;
		border-radius: 2px;
	}

	.chart {
		width: 100%;
	}
</style>
