<script>
	/** @typedef {{ date: Date, stars: number }} StarPoint */
	import { onMount } from 'svelte';
	import * as d3 from 'd3';

	let chartEl = $state(/** @type {SVGSVGElement | null} */ (null)); // SVG 元素的引用
	let activeScene = $state(0); // 当前在第几个场景（0还没开始，1/2/3）

	// DATA
	const OC = [
		{ date: new Date('2025-11-24T02:16:47'), stars: 0 },
		{ date: new Date('2025-11-27T12:11:23'), stars: 100 },
		{ date: new Date('2026-01-24T13:43:06'), stars: 7900 },
		{ date: new Date('2026-01-25T03:41:16'), stars: 11900 },
		{ date: new Date('2026-01-25T10:36:36'), stars: 15900 },
		{ date: new Date('2026-01-25T17:47:30'), stars: 19900 },
		{ date: new Date('2026-01-25T21:09:22'), stars: 23900 },
		{ date: new Date('2026-01-26T00:04:14'), stars: 27900 },
		{ date: new Date('2026-01-26T03:05:36'), stars: 31900 },
		{ date: new Date('2026-01-26T06:27:41'), stars: 35900 },
		{ date: new Date('2026-01-26T10:25:59'), stars: 39900 },
		{ date: new Date('2026-04-20T16:36:15'), stars: 361216 }
	];

	const PY = [
		{ date: new Date('2016-07-16T02:44:01'), stars: 0 },
		{ date: new Date('2016-07-31T06:22:52'), stars: 100 },
		{ date: new Date('2018-09-23T08:15:23'), stars: 7900 },
		{ date: new Date('2018-10-12T03:06:03'), stars: 11900 },
		{ date: new Date('2018-11-22T17:21:00'), stars: 15900 },
		{ date: new Date('2019-03-03T22:57:14'), stars: 19900 },
		{ date: new Date('2019-04-27T06:14:28'), stars: 23900 },
		{ date: new Date('2019-04-29T06:27:24'), stars: 27900 },
		{ date: new Date('2019-05-06T12:54:14'), stars: 31900 },
		{ date: new Date('2019-05-18T07:33:06'), stars: 35900 },
		{ date: new Date('2019-06-05T17:43:34'), stars: 39900 },
		{ date: new Date('2026-04-20T16:37:42'), stars: 219948 }
	];

	const RE = [
		{ date: new Date('2013-05-24T09:15:54'), stars: 0 },
		{ date: new Date('2013-05-29T15:16:19'), stars: 100 },
		{ date: new Date('2014-09-27T00:53:48'), stars: 7900 },
		{ date: new Date('2015-01-29T02:34:17'), stars: 11900 },
		{ date: new Date('2015-03-24T19:34:52'), stars: 15900 },
		{ date: new Date('2015-06-06T00:31:11'), stars: 19900 },
		{ date: new Date('2015-09-13T22:10:25'), stars: 23900 },
		{ date: new Date('2015-11-30T19:29:03'), stars: 27900 },
		{ date: new Date('2016-02-22T16:22:13'), stars: 31900 },
		{ date: new Date('2016-04-27T23:12:43'), stars: 35900 },
		{ date: new Date('2016-07-19T09:54:42'), stars: 39900 },
		{ date: new Date('2026-04-20T16:37:53'), stars: 244596 }
	];

	const X_S4 = [new Date('2026-01-27'), new Date('2026-03-15')];

	const allEvents = [
		{
			date: new Date('2026-01-29'),
			label: 'JD.com',
			sub: 'Cloud VM pre-installed image',
			lineH: 50,
			dx: 0
		},
		{
			date: new Date('2026-02-02'),
			label: 'Baidu',
			sub: 'Mobile deployment solution',
			lineH: 45,
			dx: 0
		},
		{ date: new Date('2026-02-20'), label: 'Baidu', sub: 'Smart Cloud upgrade', lineH: 40, dx: 0 },
		{
			date: new Date('2026-02-28'),
			label: 'Alibaba Cloud',
			sub: 'Workbench & image',
			lineH: 55,
			dx: 0
		},
		{
			date: new Date('2026-03-05'),
			label: 'Feishu',
			sub: 'Official plugin launched',
			lineH: 65,
			dx: 0
		},
		{ date: new Date('2026-03-06'), label: 'Xiaomi', sub: 'Xiaomi MiClaw', lineH: -50, dx: 0 },
		{
			date: new Date('2026-03-06'),
			label: 'Tencent',
			sub: 'Free pop-up in Shenzhen',
			lineH: 100,
			dx: 0
		},
		{
			date: new Date('2026-03-07'),
			label: 'Meituan',
			sub: 'Remote deployment service',
			lineH: -100,
			dx: 0
		},
		{ date: new Date('2026-03-07'), label: 'WeChat', sub: 'OpenClaw plugin', lineH: 150, dx: 0 },
		{
			date: new Date('2026-03-08'),
			label: 'Zhipu AI',
			sub: 'Local OpenClaw build',
			lineH: -155,
			dx: 0
		},
		{ date: new Date('2026-03-09'), label: 'Tencent', sub: 'WorkBuddy', lineH: 90, dx: 0 },
		{ date: new Date('2026-03-09'), label: 'ByteDance', sub: 'Volcengine SaaS', lineH: -55, dx: 0 },
		{ date: new Date('2026-03-10'), label: 'HONOR', sub: 'Lobster Universe', lineH: 120, dx: 0 },
		{
			date: new Date('2026-03-10'),
			label: 'JD.com',
			sub: '1-on-1 remote deployment',
			lineH: -230,
			dx: 0
		},
		{
			date: new Date('2026-03-10'),
			label: 'Baidu',
			sub: 'Zero-deployment solution',
			lineH: 48,
			dx: 0
		},
		{ date: new Date('2026-03-11'), label: 'Huawei', sub: 'Xiaoyi Claw beta', lineH: -200, dx: 0 }
	];

	// const allEvents = [
	// 	{ date: new Date('2026-01-29'), label: '京东', sub: '云主机预置镜像', lineH: 50, dx: 0 },
	// 	{ date: new Date('2026-02-02'), label: '百度', sub: '移动端部署方案', lineH: 45, dx: 0 },
	// 	{ date: new Date('2026-02-20'), label: '百度', sub: '智能云升级服务', lineH: 40, dx: 0 },
	// 	{ date: new Date('2026-02-28'), label: '阿里云', sub: '工作台 & 镜像', lineH: 55, dx: 0 },
	// 	{ date: new Date('2026-03-05'), label: '飞书', sub: '官方插件上线', lineH: 65, dx: 0 },
	// 	{ date: new Date('2026-03-06'), label: '小米', sub: 'Xiaomi MiClaw', lineH: -50, dx: 0 },
	// 	{ date: new Date('2026-03-06'), label: '腾讯', sub: '深圳免费快闪', lineH: 100, dx: 0 },
	// 	{ date: new Date('2026-03-07'), label: '美团', sub: '远程部署服务', lineH: -100, dx: 0 },
	// 	{ date: new Date('2026-03-07'), label: '企业微信', sub: 'OpenClaw 插件', lineH: 150, dx: 0 },
	// 	{ date: new Date('2026-03-08'), label: '智谱AI', sub: '本地版 OpenClaw', lineH: -155, dx: 0 },
	// 	{ date: new Date('2026-03-09'), label: '腾讯', sub: 'WorkBuddy', lineH: 90, dx: 0 },
	// 	{ date: new Date('2026-03-09'), label: '字节跳动', sub: '火山引擎 SaaS', lineH: -55, dx: 0 },
	// 	{ date: new Date('2026-03-10'), label: '荣耀', sub: '龙虾宇宙', lineH: 120, dx: 0 },
	// 	{ date: new Date('2026-03-10'), label: '京东', sub: '一对一远程部署', lineH: -120, dx: 0 },
	// 	{ date: new Date('2026-03-10'), label: '百度', sub: '零部署方案', lineH: 48, dx: 0 },
	// 	{ date: new Date('2026-03-11'), label: '华为', sub: '小艺 Claw 测试版', lineH: -200, dx: 0 }
	// ];

	// ── 尺寸 ──
	const margin = { top: 5, right: 25, bottom: 25, left: 25 };

	onMount(() => {
		const totalW = chartEl.parentElement.clientWidth;
		const totalH = window.innerHeight * 0.8;
		const W = totalW - margin.left - margin.right;
		const H = totalH - margin.top - margin.bottom;

		const svg = d3
			.select(chartEl)
			.attr('width', totalW)
			.attr('height', totalH)
			.style('overflow', 'visible');

		const g = svg.append('g').attr('transform', `translate(${margin.left},${margin.top})`);

		svg
			.append('defs')
			.append('clipPath')
			.attr('id', 'chart-clip')
			.append('rect')
			.attr('width', W)
			.attr('height', H);

		console.log('SVG ready:', totalW, totalH);

		// ── 两个 x 轴范围：场景1收紧，场景2/3拉开 ──
		const X_S1 = [new Date('2025-10-10'), new Date('2026-05-15')];
		const X_S2 = [new Date('2012-06-01'), new Date('2027-06-01')];

		// ── 比例尺 ──
		const xSc = d3.scaleTime().domain(X_S1).range([0, W]);

		const ySc = d3.scaleLinear().domain([0, 385000]).range([H, 0]);

		// ── 折线生成器 ──
		const lineGen = d3
			.line()
			.x(/** @param {StarPoint} d */ (d) => xSc(d.date))
			.y(/** @param {StarPoint} d */ (d) => ySc(d.stars))
			.curve(d3.curveMonotoneX);

		// ── 坐标轴容器 ──
		const xAxisG = g.append('g').attr('transform', `translate(0,${H})`);

		const yAxisG = g.append('g');

		// ── 画轴的函数（后面切换场景时会反复调用）──
		function renderAxes(domain, animate) {
			xSc.domain(domain);

			const isWide = domain[0].getFullYear() < 2020;
			const isTight = domain[1] - domain[0] < 1000 * 60 * 60 * 24 * 60;

			const xAxis = d3
				.axisBottom(xSc)
				.ticks(isWide ? 8 : isTight ? 6 : 5)
				.tickFormat(
					isWide ? d3.timeFormat('%Y') : isTight ? d3.timeFormat('%b %d') : d3.timeFormat('%b %Y')
				)
				.tickSize(0);

			const yAxis = d3
				.axisLeft(ySc)
				.ticks(5)
				.tickFormat((d) => (d === 0 ? '0' : `${d / 1000}k`))
				.tickSize(-W);

			const t = animate
				? d3.transition().duration(950).ease(d3.easeCubicInOut)
				: d3.transition().duration(0);

			xAxisG.transition(t).call(xAxis);
			yAxisG.transition(t).call(yAxis);

			// 样式
			g.selectAll('.tick line').attr('stroke', '#ffffff30');
			g.selectAll('.domain').attr('stroke', '#ffffff50');
			g.selectAll('.tick text')
				.style('fill', '#aaa')
				.style('font-family', 'Lato, sans-serif')
				.style('font-size', '1rem');
		}

		// 初始渲染（场景1的范围，不做动画）
		renderAxes(X_S1, false);
		g.append('text')
			.attr('transform', 'rotate(-90)')
			.attr('x', -H / 2)
			.attr('y', -margin.left - 30)
			.attr('text-anchor', 'middle')
			.style('font-family', 'Lato, sans-serif')
			.style('font-size', '24px')
			.style('fill', '#ffffff80')
			.text('GitHub Stars');

		// ── 水平辅助网格线 ──
		ySc.ticks(5).forEach((tick) => {
			if (tick === 0) return;
			g.append('line')
				.attr('x1', 0)
				.attr('x2', W)
				.attr('y1', ySc(tick))
				.attr('y2', ySc(tick))
				.attr('stroke', '#ffffff10')
				.attr('stroke-width', 2.5);
		});

		// ── 三条折线 ──
		// OpenClaw（最顶层，始终可见）
		const pathOC = g
			.append('path')
			.datum(OC)
			.attr('fill', 'none')
			.attr('stroke', '#FF2A66')
			.attr('stroke-width', 3)
			.attr('opacity', 1)
			.attr('d', lineGen)
			.attr('clip-path', 'url(#chart-clip)');

		// React（最底层，场景2才出现）
		const pathRE = g
			.append('path')
			.datum(RE)
			.attr('fill', 'none')
			.attr('stroke', '#EFCA23')
			.attr('stroke-width', 3)
			.attr('opacity', 0)
			.attr('d', lineGen)
			.attr('clip-path', 'url(#chart-clip)');

		// Python（中间层，场景2才出现）
		const pathPY = g
			.append('path')
			.datum(PY)
			.attr('fill', 'none')
			.attr('stroke', '#1DD2DF')
			.attr('stroke-width', 3)
			.attr('opacity', 0)
			.attr('d', lineGen)
			.attr('clip-path', 'url(#chart-clip)')
			.attr('clip-path', 'url(#chart-clip)');

		// ── 标注层 ──
		const layerS1 = g.append('g').attr('opacity', 0);

		function buildAnnotS1() {
			layerS1.selectAll('*').remove();

			const last = OC[OC.length - 1];
			const cx = xSc(last.date);
			const cy = ySc(last.stars);

			// dots
			layerS1
				.append('circle')
				.attr('cx', cx)
				.attr('cy', cy)
				.attr('r', 10)
				.attr('fill', '#FF2A66')
				.attr('stroke', '#13062b')
				.attr('stroke-width', 2);

			//star
			layerS1
				.append('text')
				.attr('x', cx - 14)
				.attr('y', cy - 2)
				.attr('text-anchor', 'end')
				.style('font-family', 'Lato, sans-serif')
				.style('font-size', '24px')
				.style('font-weight', '700')
				.style('fill', '#FF2A66')
				.text('361,216 Stars');

			// "GitHub History #6"
			layerS1
				.append('text')
				.attr('x', cx - 14)
				.attr('y', cy - 30)
				.attr('text-anchor', 'end')
				.style('font-family', 'Lato, sans-serif')
				.style('font-size', '18px')
				.style('fill', '#FF2A66')
				.text('GitHub History #6');

			// "OpenClaw" 标签，放在折线急速上升段旁边
			const midPt = OC[5]; // Jan 25 ~19,900 附近，正在爬坡
			layerS1
				.append('text')
				.attr('x', xSc(midPt.date) + 150)
				.attr('y', ySc(midPt.stars) - 300)
				.style('font-family', 'Lato, sans-serif')
				.style('font-size', '24px')
				.style('font-weight', '700')
				.style('fill', '#FF2A66')
				.text('OpenClaw');
		}

		const layerS2 = g.append('g').attr('opacity', 0);

		function buildAnnotS2() {
			layerS2.selectAll('*').remove();

			const repos = [
				{ data: OC, color: '#FF2A66', name: 'OpenClaw', dy: -12 },
				{ data: RE, color: '#EFCA23', name: 'React', dy: -12 },
				{ data: PY, color: '#1DD2DF', name: 'Python', dy: 20 }
			];

			repos.forEach(({ data, color, name, dy }) => {
				const last = data[data.length - 1];
				const cx = xSc(last.date);
				const cy = ySc(last.stars);

				// 终点圆点
				layerS2
					.append('circle')
					.attr('cx', cx)
					.attr('cy', cy)
					.attr('r', 5)
					.attr('fill', color)
					.attr('stroke', '#13062b')
					.attr('stroke-width', 2);

				// 名字标签
				layerS2
					.append('text')
					.attr('x', cx - 30)
					.attr('y', cy + dy + 60)
					.attr('text-anchor', 'end')
					.style('font-family', 'Lato, sans-serif')
					.style('font-size', '24px')
					.style('font-weight', '700')
					.style('fill', color)
					.text(name);
			});
		}

		const layerS3 = g.append('g').attr('opacity', 0);

		function buildAnnotS3() {
			layerS3.selectAll('*').remove();

			// ── 40k 参考横线 ──
			layerS3
				.append('line')
				.attr('x1', 0)
				.attr('x2', W)
				.attr('y1', ySc(40000))
				.attr('y2', ySc(40000))
				.attr('stroke', '#ffffff40')
				.attr('stroke-dasharray', '6,4')
				.attr('stroke-width', 1);

			layerS3
				.append('text')
				.attr('x', 4)
				.attr('y', ySc(40000) - 6)
				.style('font-family', 'Lato, sans-serif')
				.style('font-size', '11px')
				.style('fill', '#ffffff60')
				.text('40,000 ★');

			// ── 三个交叉点的标注 ──
			const crossings = [
				{
					date: new Date('2016-07-19'),
					label: '3 yrs 2 mo',
					color: '#EFCA23',
					anchor: 'middle',
					dx: 0
				},
				{
					date: new Date('2019-06-05'),
					label: '2 yrs 11 mo',
					color: '#1DD2DF',
					anchor: 'middle',
					dx: 0
				},
				{ date: new Date('2026-01-26'), label: '63 days', color: '#FF2A66', anchor: 'end', dx: -8 }
			];

			crossings.forEach(({ date, label, color, anchor, dx }) => {
				const cx = xSc(date);
				const cy = ySc(40000);

				// 交叉点圆点
				layerS3
					.append('circle')
					.attr('cx', cx)
					.attr('cy', cy)
					.attr('r', 4)
					.attr('fill', color)
					.attr('stroke', '#13062b')
					.attr('stroke-width', 1.5);

				// 向上的短竖线
				layerS3
					.append('line')
					.attr('x1', cx + dx / 2)
					.attr('x2', cx + dx)
					.attr('y1', cy - 6)
					.attr('y2', cy - 40)
					.attr('stroke', color)
					.attr('stroke-width', 1)
					.attr('opacity', 0.5);

				// 用时标注
				layerS3
					.append('text')
					.attr('x', cx + dx)
					.attr('y', cy - 46)
					.attr('text-anchor', anchor)
					.style('font-family', 'Lato, sans-serif')
					.style('font-size', '12px')
					.style('font-weight', '700')
					.style('fill', color)
					.text(label);
			});
		}

		const layerS4 = g.append('g').attr('opacity', 0);

		function interpolateStars(date) {
			const t0 = new Date('2026-01-26T10:25:59').getTime();
			const t1 = new Date('2026-04-20T16:36:15').getTime();
			const ratio = Math.max(0, Math.min(1, (date.getTime() - t0) / (t1 - t0)));
			return 39900 + (361216 - 39900) * ratio;
		}

		function getYOnPath(targetX) {
			// 用当前 scale 生成一个临时 path，不依赖动画进度
			const tempPath = document.createElementNS('http://www.w3.org/2000/svg', 'path');
			tempPath.setAttribute('d', lineGen(OC));
			const total = tempPath.getTotalLength();
			let lo = 0,
				hi = total;
			for (let i = 0; i < 50; i++) {
				const mid = (lo + hi) / 2;
				const pt = tempPath.getPointAtLength(mid);
				if (Math.abs(pt.x - targetX) < 0.5) return pt.y;
				pt.x < targetX ? (lo = mid) : (hi = mid);
			}
			return tempPath.getPointAtLength((lo + hi) / 2).y;
		}

		function buildAnnotS4() {
			layerS4.selectAll('*').remove();

			allEvents.forEach(({ date, label, sub, lineH, dx }) => {
				const cx = xSc(date) + dx;
				const dotY = getYOnPath(xSc(date)); // 点在折线上
				const subY = dotY - lineH; // 产品名
				const lblY = subY - 14; // 公司名（在上方）

				// 竖线：从点到标签之间
				layerS4
					.append('line')
					.attr('x1', cx)
					.attr('x2', cx)
					.attr('y1', dotY - 5)
					.attr('y2', subY - 2)
					.attr('stroke', '#ffffff35')
					.attr('stroke-width', 1)
					.attr('stroke-dasharray', '3,3');

				// 折线上的圆点
				layerS4
					.append('circle')
					.attr('cx', cx)
					.attr('cy', dotY)
					.attr('r', 6)
					.attr('fill', '#ff2a66');

				// 公司名（红色，粗体）
				layerS4
					.append('text')
					.attr('x', cx)
					.attr('y', lblY)
					.attr('text-anchor', 'middle')
					.style('font-family', 'Lato, sans-serif')
					.style('font-size', '18px')
					.style('font-weight', '700')
					.style('fill', '#ff2a66')
					.text(label);

				// 产品名（白色，细）
				layerS4
					.append('text')
					.attr('x', cx)
					.attr('y', subY)
					.attr('text-anchor', 'middle')
					.style('font-family', 'Lato, sans-serif')
					.style('font-size', '12px')
					.style('fill', '#ffffffcc')
					.text(sub);
			});
		}
		// ── 更新折线路径的辅助函数 ──
		function updatePaths(animate) {
			const t = d3
				.transition()
				.duration(animate ? 950 : 0)
				.ease(d3.easeCubicInOut);
			pathOC.transition(t).attr('d', lineGen);
			pathPY.transition(t).attr('d', lineGen);
			pathRE.transition(t).attr('d', lineGen);
		}

		// ── 场景切换主函数 ──
		function showScene(n, animate) {
			const FADE = animate ? 600 : 0;
			const DLY = animate ? 350 : 0;

			if (n === 1) {
				renderAxes(X_S1, animate);
				updatePaths(animate);
				pathPY.transition().duration(FADE).attr('opacity', 0);
				pathRE.transition().duration(FADE).attr('opacity', 0);
				buildAnnotS1();
				layerS1.transition().delay(DLY).duration(FADE).attr('opacity', 1);
				layerS2
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
				layerS3
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
				layerS4
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
			} else if (n === 2) {
				renderAxes(X_S2, animate);
				pathPY.attr('d', lineGen);
				pathRE.attr('d', lineGen);
				pathOC.transition(d3.transition().duration(animate ? 950 : 0)).attr('d', lineGen);
				pathRE.transition().delay(DLY).duration(FADE).attr('opacity', 1);
				pathPY
					.transition()
					.delay(DLY + 120)
					.duration(FADE)
					.attr('opacity', 1);
				layerS1
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
				buildAnnotS2();
				layerS2
					.transition()
					.delay(DLY + 400)
					.duration(FADE)
					.attr('opacity', 1);
				layerS3
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
				layerS4
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
			} else if (n === 3) {
				pathPY.attr('opacity', 1);
				pathRE.attr('opacity', 1);
				layerS1
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
				layerS2
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
				buildAnnotS2();
				layerS2.transition().duration(0).attr('opacity', 1);
				buildAnnotS3();
				layerS3.transition().delay(DLY).duration(FADE).attr('opacity', 1);
				layerS4
					.transition()
					.duration(FADE / 2)
					.attr('opacity', 0);
			} else if (n === 4) {
				renderAxes(X_S4, animate);
				pathPY.attr('d', lineGen);
				pathRE.attr('d', lineGen);
				pathOC.transition(d3.transition().duration(animate ? 950 : 0)).attr('d', lineGen);
				pathPY.transition().duration(FADE).attr('opacity', 0);
				pathRE.transition().duration(FADE).attr('opacity', 0);
				layerS1.transition().duration(FADE).attr('opacity', 0);
				layerS2.transition().duration(FADE).attr('opacity', 0);
				layerS3.transition().duration(FADE).attr('opacity', 0);
				buildAnnotS4();
				layerS4.transition().delay(DLY).duration(FADE).attr('opacity', 1);
			}
		}

		// 初始场景
		buildAnnotS1();
		layerS1.attr('opacity', 1);
		layerS2.attr('opacity', 0);
		showScene(1, false);

		// ── Intersection Observer：监听哪个 step 滚进视野 ──
		const steps = document.querySelectorAll('.step');

		const observer = new IntersectionObserver(
			(entries) => {
				entries.forEach((entry) => {
					if (entry.isIntersecting) {
						const n = Number(entry.target.dataset.step);
						if (n !== activeScene) {
							activeScene = n;
							showScene(n, true);
						}
					}
				});
			},
			{
				threshold: 0.5
			}
		);

		steps.forEach((s) => observer.observe(s));

		// ── 图表注释 ──
		svg
			.append('text')
			.attr('x', margin.left)
			.attr('y', totalH + 30)
			.style('font-family', 'Lato, sans-serif')
			.style('font-size', '10px')
			.style('fill', '#ffffff40')
			.text('Data as of April 20, 2026.');

		// 数据来源（可点击链接）
		const sourceLink = svg
			.append('a')
			.attr('href', 'https://seladb.github.io/StarTrack-js/#/')
			.attr('target', '_blank');

		sourceLink
			.append('text')
			.attr('x', margin.left)
			.attr('y', totalH + 12)
			.style('font-family', 'Lato, sans-serif')
			.style('font-size', '10px')
			.style('fill', '#ffffff40')
			.style('text-decoration', 'underline')
			.text('Source: seladb.github.io/StarTrack-js');

		// 组件销毁时清理监听器
		return () => observer.disconnect();

		console.log('Scales ready. Test point:', xSc(new Date('2026-01-24')));
	});
</script>

<div class="scrolly-container">
	<!-- 图表：粘在背景 -->
	<div class="chart-bg">
		<svg bind:this={chartEl}></svg>
	</div>

	<!-- 文字：滚动覆盖在图表上方 -->
	<div class="steps">
		<section class="step" data-step="1">
			<div class="sticky-text">
				<div class="text-box">
					<p>
						{@html `Before we go further, GitHub stars are among the most direct measures of an open-source
						project's reach. Each star represents a developer marking a repository as worth
						following. <br /><br />On January 24th, 2026, <mark class="pink">OpenClaw</mark> spread across developers' screens
						faster than almost anything GitHub had seen before. <mark class="pink">Within less than 48 hours</mark>, its star
						count climbed from under 8,000 to <mark class="pink">40,000</mark>. <br /><br />By April 20th, the total stood at
						361,216, make OpenClaw sixth on GitHub's all-time leaderboard.`}
					</p>
				</div>
			</div>
		</section>

		<section class="step" data-step="2">
			<div class="sticky-text">
				<div class="text-box">
					<p>
						{@html `Just looking at a single line might not tell you much, but let's compare it with React and Python.
  <br/><br/>
  For context, <mark class="yellow">React</mark> is Facebook's front-end framework, released in 2013 and now powering the interfaces of much of the web.
  <br/><br/>
  <mark class="blue">Python</mark>, you must have heard it before, is one of the world's most popular libraries for learning algorithms. It is a repository that virtually every aspiring programmer has bookmarked.
  <br/><br/>
  And that crazy vertical line on the right is <mark class="pink">OpenClaw</mark>.`}
					</p>
				</div>
			</div>
		</section>

		<section class="step" data-step="3">
			<div class="sticky-text">
				<div class="text-box">
					<p>
						<mark class="yellow">React</mark> reached 40,000 stars in
						<mark class="yellow">3 years and 2 months</mark>. <br /><mark class="blue">Python</mark>
						took <mark class="blue">2 years and 11 months.</mark>
						<br /><br /><mark class="pink">OpenClaw</mark> took only
						<mark class="pink">63 days</mark>.
					</p>
				</div>
			</div>
		</section>

		<section class="step" data-step="4">
			<div class="sticky-text">
				<div class="text-box">
					<p>
						{@html `Chinese internet users quickly gave it a nickname, <mark class="pink">“the lobster”</mark>, which is the "claw" in OpenClaw standing for. China's tech giants moved fast to plug in this new AI tool into their original products. Days, not months, not even weeks.`}
					</p>
				</div>
			</div>
		</section>
	</div>
</div>

<style>
	.scrolly-container {
		position: relative;
		width: 80vw;
		left: -15vw;
		/* background: green; */
	}

	/* 图表粘在顶部，负 margin 让文字从同一高度开始叠上来 */
	.chart-bg {
		position: sticky;
		top: 80vh;
		height: 100vh;
		z-index: 0;
		display: flex;
		align-items: center;
		justify-content: center;
		/* background: darkred;
		border: 3px solid blue; */
	}

	/* 文字层浮在图表上方 */
	.steps {
		position: relative;
		z-index: 10;
		pointer-events: none;
	}

	.step {
		height: 100vh;
		display: flex;
		align-items: center;
		padding: 0 64px;
		width: 100%;
		margin: 0 auto;
		/* background: rgba(255, 192, 203, 0.649);
		border: 3px solid pink; */
	}

	.sticky-text {
		width: 60%;
		position: sticky;
		top: 25vh; /* 文字框停在视窗25%的位置 */
		display: flex;
		align-items: flex-start;
		padding: 0 64px;
	}

	/* 文字盒子，半透明深色背景保证可读性 */
	.text-box {
		width: 100%;
		background: rgba(255, 255, 255, 0.5);
		/* border: 2px solid #ff2a66; */
		color: white;
		padding: 1rem 1.25rem;
		font-family: 'Lato', sans-serif;
		text-align: justify;
		font-size: 1.3rem;
		line-height: 1.85;
		pointer-events: all;
	}

	:global(.text-box mark.pink) {
		background: linear-gradient(120deg, #ff2a66 0%, #ff2a66 100%) no-repeat;
		background-size: 100% 40%;
		background-position: 0 85%;
		color: inherit;
	}

	:global(.text-box mark.yellow) {
		background: linear-gradient(120deg, #efca23 0%, #efca23 100%) no-repeat;
		background-size: 100% 40%;
		background-position: 0 85%;
	}

	:global(.text-box mark.blue) {
		background: linear-gradient(120deg, #1dd2df 0%, #1dd2df 100%) no-repeat;
		background-size: 100% 40%;
		background-position: 0 85%;
	}
</style>
