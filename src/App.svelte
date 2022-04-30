<script>
	import * as d3 from "d3";
	import AxisHorizontal from "./components/AxisHorizontal.svelte";
	import AxisVertical from "./components/AxisVertical.svelte";
	import Tooltip from "./components/Tooltip.svelte";
	import Legend from "./components/Legend.svelte";
  
	// access data
	let data = [];

	d3.csv(
	  "https://gist.githubusercontent.com/stefanreifenberg/068a2ec2f8ec5b83247630910108e195/raw/11be7cae4783c4967aae92d6add04c2448996068/freiburg2017-2019"
	).then(res => {
	  data = res
	  console.log(data)
	});
	// the angle corresponds to the day of the year
	// this way, seasons line up across years

	const angleAccessor = d => 
	  +d3.timeFormat("%-j")
		(new Date(d.date))

	//console.log(angleAccessor(data[0]))
	// earlier dates are closer to the middle of the circle
	const radiusAccessor = d => new Date(d.date)
	// we have a finer-grained radius offset showing the max temp
	// convert fahrenheit to celsius
	const fahrenheitToCelsius = fahrenheit => (fahrenheit - 32) * 5/9;

	const radiusOffsetAccessor = d => fahrenheitToCelsius(+d.temperatureHigh)
	// we also show the max temp with a color scale
	const colorAccessor = d => fahrenheitToCelsius(+d.temperatureHigh)
  
	// create chart dimensions
	let margin = {
	  left: 0,
	  top: 0,
	  right: 0,
	  bottom: 0
	};
	let width = 400;
	let height = 400;
	$: boundsWidth = width - margin.left - margin.right;
	$: boundsHeight = height - margin.top - margin.bottom;
  
	// create scales
	$: angleScale = d3.scaleLinear()
	  .domain([0, 365]) // 365 days in a year
	  .range([0, Math.PI * 2]) // Math.PI * 2 radians in a circle
  
	$: radiusScale = d3.scaleLinear()
	  .domain(d3.extent(data, radiusAccessor))
	  .range([
		// we'll start the spiral _near_ the center,
		// but a bit spaced out, to make the spiral less dramatic
		boundsWidth * 0.13,
		boundsWidth / 2
	  ]);
  
	$: radiusOffsetScale = d3.scaleLinear()
	  .domain(d3.extent(data, radiusOffsetAccessor))
	  // we want a small distance to show the max temp for each day
	  // otherwise we'll overlap between loops
	  .range([0, boundsWidth * 0.06]);
  
	// https://github.com/d3/d3-scale#scaleSequential
	// https://github.com/d3/d3-scale-chromatic
	$: colorScale = d3.scaleSequential(d3.interpolateBuPu)
	  .domain(d3.extent(data, colorAccessor))
  
	// draw data
	$: getPositionsForPoint = (d) => {
	  const r = radiusScale(radiusAccessor(d))
	  const rOffset = radiusOffsetScale(radiusOffsetAccessor(d))
	  const angle = angleScale(angleAccessor(d))
	  // remember soh cah toa?
	  // https://mathworld.wolfram.com/images/eps-svg/SOHCAHTOA_500.svg
	  const x = r * Math.cos(angle)
	  const y = r * Math.sin(angle)
	  const xMin = (r - rOffset / 2) * Math.cos(angle)
	  const yMin = (r - rOffset / 2) * Math.sin(angle)
	  const xMax = (r + rOffset / 2) * Math.cos(angle)
	  const yMax = (r + rOffset / 2) * Math.sin(angle)
	  return [
		[x, y],        // the middle point
		[xMin, yMin],  // the inner point
		[xMax, yMax],  // the outer point
	  ]
	}
	$: positions = data.map(getPositionsForPoint)
  
	const getPixelsBetweenLinesAtRadius = r => {
	  // used to get the width of each line
	  // these grow wider as we move away from the center
	  const radians = Math.PI * 2 / 365
	  const x = r * Math.cos(0)
	  const y = r * Math.sin(0)
	  const x2 = r * Math.cos(-radians)
	  const y2 = r * Math.sin(-radians)
	  const distance = Math.sqrt(
		(x - x2)**2 + (y - y2)**2
	  )
	  // we want a stroke-width of at least 0.5px
	  return Math.max(1.5, distance)
	}
  
	// set up interactions
	let hoveredPointIndex = null
	$: hoveredPoint = data[hoveredPointIndex]
	$: hoveredPointPosition = positions[hoveredPointIndex]
	$: quadtree = d3.quadtree()
	  // let's check nearness to the middle of the line
	  .x(d => getPositionsForPoint(d)[0][0])
	  .y(d => getPositionsForPoint(d)[0][1])
	  .addAll(data);
  </script>
  
  <h2>Maximum temperature in Freiburg from 2017-2019</h2>
  
  <figure>
	<div class="wrapper" bind:clientWidth={width} bind:clientHeight={height}>
	  <svg width={width} height={height}>
		<g transform="translate({margin.left}, {margin.top})">
		  <g transform="translate({boundsWidth / 2}, {boundsHeight / 2})">
  
			{#each data as d,i}
			  {console.log(d)}
			  {#if hoveredPoint === d}
				<!-- outline -->
				<line
				  x1={positions[i][1][0]}
				  y1={positions[i][1][1]}
				  x2={positions[i][2][0]}
				  y2={positions[i][2][1]}
				  stroke="black"
				  stroke-width={
					getPixelsBetweenLinesAtRadius(radiusScale(radiusAccessor(d)))
					+ 1
				  }
				  stroke-linecap="round"
				/>
			  {/if}
			  <line
				x1={positions[i][1][0]}
				y1={positions[i][1][1]}
				x2={positions[i][2][0]}
				y2={positions[i][2][1]}
				stroke={colorScale(colorAccessor(d))}
				stroke-width={
				  getPixelsBetweenLinesAtRadius(radiusScale(radiusAccessor(d)))
				  - 1
				}
				stroke-linecap="round"
			  />
			{/each}
  
		  </g>
  
		  <rect
			width={boundsWidth}
			height={boundsHeight}
			fill="transparent"
			on:mousemove={e => {
			  const pos = d3.pointer(e)
			  const x = pos[0] - boundsWidth / 2
			  const y = pos[1] - boundsWidth / 2
			  const closestPoint = quadtree.find(x, y)
			  if (!closestPoint) return
			  const closestPointIndex = data.indexOf(closestPoint)
			  const hoveredPointPosition = positions[closestPointIndex][0]
			  // don't highlight if too far away
			  // a^2 + b^2 = c^2
			  const distance = Math.sqrt(
				(x - hoveredPointPosition[0]) ** 2
				+ (y - hoveredPointPosition[1]) ** 2
			  )
			  if (distance < 50) {
				hoveredPointIndex = closestPointIndex
			  } else {
				hoveredPoint = null
			  }
			}}
		  />
  
		</g>
  
	  </svg>
  
	  {#if hoveredPoint}
		<Tooltip
		  x={margin.left + hoveredPointPosition[0][0] + boundsWidth / 2}
		  y={margin.top + hoveredPointPosition[0][1] + boundsWidth / 2 - 15}
		>
		  <strong>
			{d3.timeFormat("%b %-d, %Y")(radiusAccessor(hoveredPoint))}
		  </strong>
		  <div>
			max temp: {parseInt(fahrenheitToCelsius(hoveredPoint.temperatureHigh))}°C
		  </div>
		</Tooltip>
	  {/if}
  
	  <Legend
		heightScale={radiusOffsetScale}
		{colorScale}
	  />
	</div>
  </figure>
  
  <style>
	.wrapper {
	  position: relative;
	  margin: 0 auto;
	  font-family: sans-serif;
	  width: 80vw;
	  height: 80vw;
	  min-width: 0;
	}
  
	svg {
	  overflow: visible;
	}
  
	h2 {
	  text-align: center;
	  font-family: sans-serif;
	}
  </style>