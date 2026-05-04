<script>
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	const rawData = [
		{ provider: 'Anthropic', model_name: 'Claude Haiku 4.5', country: 'USA', tokens_raw: 1.72e11 },
		{ provider: 'Anthropic', model_name: 'Claude Opus 4.6', country: 'USA', tokens_raw: 5.96e11 },
		{ provider: 'Anthropic', model_name: 'Claude Opus 4.7', country: 'USA', tokens_raw: 2.23e11 },
		{ provider: 'Anthropic', model_name: 'Claude Sonnet 4.6', country: 'USA', tokens_raw: 8.74e11 },
		{ provider: 'Minimax', model_name: 'MiniMax M2.5', country: 'China', tokens_raw: 4.46e11 },
		{ provider: 'Minimax', model_name: 'MiniMax M2.7', country: 'China', tokens_raw: 9.33e11 },
		{ provider: 'Qwen', model_name: 'Qwen3.6 Plus', country: 'China', tokens_raw: 1.01e12 },
		{ provider: 'Z-ai', model_name: 'GLM 5 Turbo', country: 'China', tokens_raw: 9.89e11 },
		{ provider: 'Google', model_name: 'Gemini 2.5 Flash', country: 'USA', tokens_raw: 1.92e11 },
		{
			provider: 'Google',
			model_name: 'Gemini 2.5 Flash Lite',
			country: 'USA',
			tokens_raw: 4.47e11
		},
		{
			provider: 'Google',
			model_name: 'Gemini 3 Flash Preview',
			country: 'USA',
			tokens_raw: 3.27e11
		},
		{ provider: 'Nvidia', model_name: 'Nemotron 3 Super', country: 'USA', tokens_raw: 7.77e11 },
		{ provider: 'Stepfun', model_name: 'Step 3.5 Flash', country: 'China', tokens_raw: 6.71e11 },
		{ provider: 'Xiaomi', model_name: 'MiMo-V2-Flash', country: 'China', tokens_raw: 1.78e11 },
		{ provider: 'Xiaomi', model_name: 'MiMo-V2-Pro', country: 'China', tokens_raw: 3.21e11 },
		{ provider: 'Deepseek', model_name: 'DeepSeek V3.2', country: 'China', tokens_raw: 2.95e11 },
		{ provider: 'Openrouter', model_name: 'Elephant Alpha', country: 'USA', tokens_raw: 2.39e11 },
		{
			provider: 'Arcee-ai',
			model_name: 'Trinity Large Preview',
			country: 'USA',
			tokens_raw: 1.96e11
		},
		{ provider: 'Moonshotai', model_name: 'Kimi K2.5', country: 'China', tokens_raw: 1.95e11 },
		{ provider: 'Openai', model_name: 'gpt-oss-120b', country: 'USA', tokens_raw: 1.74e11 }
	];

	const CHINA_COLOR = '#FF2A66';
	const USA_COLOR = '#1DD2DF';
	const BG_COLOR = '#13062B';

	function fmt(v) {
		if (v >= 1e12) return (v / 1e12).toFixed(2) + 'T';
		if (v >= 1e9) return Math.round(v / 1e9) + 'B';
		return v.toString();
	}

	let container;

	onMount(() => {
		// ── 数据预处理 ──────────────────────────────────────────
		const grouped = d3.group(rawData, (d) => d.provider);

		const providers = Array.from(grouped, ([provider, models]) => {
			const country = models[0].country;
			const total = d3.sum(models, (m) => m.tokens_raw);
			// 每个 provider 内按 tokens 降序叠加
			const sorted = [...models].sort((a, b) => b.tokens_raw - a.tokens_raw);
			let cum = 0;
			const stacks = sorted.map((m) => {
				const y0 = cum;
				cum += m.tokens_raw;
				return { ...m, y0, y1: cum };
			});
			return { provider, country, total, stacks };
		}).sort((a, b) => b.total - a.total); // 按总量降序排列

		// ── 画布 ────────────────────────────────────────────────
		const margin = { top: 36, right: 16, bottom: 36, left: 16 };
		const totalW = container.clientWidth;
		const innerW = totalW - margin.left - margin.right;
		const innerH = 360;

		const svg = d3
			.select(container)
			.append('svg')
			.attr('width', totalW)
			.attr('height', innerH + margin.top + margin.bottom);

		const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);

		// ── 比例尺 ──────────────────────────────────────────────
		const xScale = d3
			.scaleBand()
			.domain(providers.map((p) => p.provider))
			.range([0, innerW])
			.paddingInner(0.28)
			.paddingOuter(0.1);

		const yScale = d3
			.scaleLinear()
			.domain([0, d3.max(providers, (p) => p.total) * 1.12])
			.range([innerH, 0]);

		// ── X 轴（provider 名） ─────────────────────────────────
		g.append('g')
			.attr('transform', `translate(0,${innerH})`)
			.call(d3.axisBottom(xScale).tickSize(0))
			.call((ax) => ax.select('.domain').remove())
			.selectAll('text')
			.style('fill', 'white')
			.style('font-size', '11px')
			.style('font-family', 'Inter, system-ui, sans-serif')
			.attr('dy', '1.4em');

		// ── Tooltip ─────────────────────────────────────────────
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

		// ── 堆叠柱 ──────────────────────────────────────────────
		providers.forEach((p) => {
			const baseColor = p.country === 'China' ? CHINA_COLOR : USA_COLOR;
			const bx = xScale(p.provider);
			const bw = xScale.bandwidth();

			p.stacks.forEach((seg, i) => {
				const y0px = yScale(seg.y0);
				const y1px = yScale(seg.y1);
				const h = y0px - y1px;
				const isTop = i === p.stacks.length - 1;

				// 顶部分段稍亮，往下递减透明度以区分层次
				const opacity = 1 - i * 0.18;

				// 顶部圆角用 path 实现
				const r = isTop ? 3 : 0;
				const rectPath = r
					? `M${bx},${y1px + r} a${r},${r} 0 0 1 ${r},-${r} h${bw - 2 * r} a${r},${r} 0 0 1 ${r},${r} v${h - r} h-${bw} z`
					: `M${bx},${y1px} h${bw} v${h} h-${bw} z`;

				g.append('path')
					.attr('d', rectPath)
					.attr('fill', baseColor)
					.attr('opacity', opacity)
					.style('cursor', 'crosshair')
					.on('mousemove', (event) => {
						tip
							.style('display', 'block')
							.style('left', `${event.clientX + 14}px`)
							.style('top', `${event.clientY - 48}px`).html(`
                <div style="font-weight:600;margin-top:1px"> Model: ${seg.model_name}</div>
                <div style="font-weight:600;margin-top:1px">Usage:${fmt(seg.tokens_raw)}</div>
              `);
					})
					.on('mouseleave', () => tip.style('display', 'none'));

				// 分段分隔线（背景色细线）
				if (i > 0) {
					g.append('line')
						.attr('x1', bx)
						.attr('x2', bx + bw)
						.attr('y1', y1px + h)
						.attr('y2', y1px + h)
						.attr('stroke', BG_COLOR)
						.attr('stroke-width', 1.5);
				}
			});

			// 柱顶 total label
			g.append('text')
				.attr('x', bx + bw / 2)
				.attr('y', yScale(p.total) - 6)
				.attr('text-anchor', 'middle')
				.style('fill', 'white')
				.style('font-size', '11px')
				.style('font-family', 'Inter, system-ui, sans-serif')
				.style('font-weight', '500')
				.text(fmt(p.total));
		});

		return () => tip.remove();
	});
</script>

<div class="wrapper">
	<h2 class="title">Token Usage by Provider, April-May,2026</h2>
	<div class="legend">
		<span class="dot" style="background:#FF2A66"></span><span>China</span>
		<span class="dot" style="background:#1DD2DF; margin-left:16px"></span><span>USA</span>
	</div>
	<div bind:this={container} class="chart"></div>
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

	.legend {
		display: flex;
		align-items: center;
		gap: 6px;
		margin-bottom: 12px;
		color: white;
		font-size: 12px;
		font-family: Inter, system-ui, sans-serif;
		opacity: 0.7;
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

	.footer {
		margin-top: 0;
		display: flex;
		flex-direction: column;
	}

	.note,
	.source {
		margin: 0;
		line-height: 18px;
		font-size: 11px;
		font-family: Inter, system-ui, sans-serif;
		color: rgba(255, 255, 255, 0.38);
	}

	.source a {
		color: rgba(255, 255, 255, 0.5);
		text-decoration: underline;
		text-underline-offset: 2px;
	}

	.source a:hover {
		color: white;
	}
</style>
