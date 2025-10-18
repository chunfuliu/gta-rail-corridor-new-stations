<script>
	import { scaleLinear } from "d3-scale";
	import { line, curveNatural } from "d3";
	import { onMount } from "svelte";
	import "../assets/global-styles.css";
	import { createEventDispatcher } from "svelte"; // for passing the info to page.svelte

	export let jsondata;
	export let lineName;
	export let fid;

	let scrollY = 0;
	let windowWidth;
	let chartContainer;
	let barWidth =40
	let selected_datapoint = undefined;
    let selected_datapoint_i = undefined;
	let selected_x
	let selected_y
	let selected_coordinates
	let mouse_x, mouse_y;
	let population
    const setMousePosition = function (event) {
        mouse_x = event.clientX;
        mouse_y = event.clientY;
    };
	var barPadding = 10; // controls how much spacing the bars will be from the
	let dispatch_fid = 0
	// Dispatch 'change' events
    const dispatch = createEventDispatcher();

    // Allows both bind:value and on:change for parent value retrieval
    function setValue(val, selected_coordinates, pop) {
      dispatch_fid = val;
	  selected_x = selected_coordinates[0]
	  selected_y = selected_coordinates[1]
	  population = pop
	  //console.log(dispatch_fid, selected_x, selected_y)
      dispatch("change", {dispatch_fid, selected_x, selected_y,population});
    }



	onMount(() => {
		window.addEventListener("scroll", handleScroll);
	});

	function handleScroll() {
		windowWidth = window.innerWidth;
		let minus = 0.002 * windowWidth ** 2 - 3.5464 * windowWidth + 2151.4;
		scrollY = window.scrollY - (windowWidth < 800 ? minus : 750);
	}

	const padding = { top: 20, right: 40, bottom: 25, left: 40 };
	let width = 7000;
	let height = 300;

	function filterDesignation(jsondata) {
		return jsondata.properties.Name === lineName;
	}
	let data = jsondata.filter(filterDesignation);

	// axis ticks
	let xTicks = data.map((d, i) => i);
	let yTicks = [0, 10000, 20000, 30000, 40000, 50000, 60000, 70000];

	function thousandToK(tick) {
		if (tick < 1000) return tick;
		if (tick < 1000000) return tick / 1000 + "K";
		return tick / 1000000 + "M";
	}

	// Swapped scales
	$: xScale = scaleLinear()
		.domain([0, data.length - 1]) // stations
		.range([padding.left, width - padding.right]);

	$: yScale = scaleLinear()
		.domain([0, Math.max(...yTicks)]) // population
		.range([height - padding.bottom, padding.top]);

	// Line generator (rotated)
	$: line_gen96 = line()
		.curve(curveNatural)
		.x((d, i) => xScale(i))
		.y((d) => yScale(d.properties["Pop21"]))(data);

	$: if (chartContainer && fid != null) {
    const index = data.findIndex(d => d.properties.Fid === fid);
    if (index !== -1) {
      const targetX = xScale(index);
      const leftVisible = chartContainer.scrollLeft;
      const rightVisible = leftVisible + chartContainer.clientWidth;
      const margin = 70; // optional margin from edges

      if (targetX < leftVisible + margin || targetX > rightVisible - margin) {
        const centerX = targetX - chartContainer.clientWidth*0.9;
        chartContainer.scrollTo({
          left: centerX,
          behavior: "smooth"
        });
      }
    }
  
}
</script>

<div class="chart" bind:this={chartContainer}>
	<svg {width} {height}>
		<!-- Y-axis (Population) -->
		<g class="y-axis">
			<line
				x1={padding.left}
				y1={padding.top}
				x2={padding.left}
				y2={height - padding.bottom}
				stroke="#191919"
				stroke-width="4"
			/>
			{#each yTicks as tick}
				
				<text
					x={padding.left - 10}
					y={yScale(tick) + 5}
					text-anchor="end"
					fill="#5e5d5d"
					font-size="10"
				>
					{thousandToK(tick)}
				</text>
			{/each}
		</g>

		<!-- Line and filled area -->
		<path
			d={line_gen96 + "L" + (width - padding.right) + "," + (height - padding.bottom) + "L" + padding.left + "," + (height - padding.bottom) + "Z"}
			fill="#748F25"
			fill-opacity="0.2"
			stroke="none"
		/>
		<path
			d={line_gen96}
			stroke="#748F25"
			stroke-width="4"
			fill="none"
			stroke-linecap="round"
			
		/>
		<!-- Grid lines -->
		{#each yTicks as tick}
			<line
				x1={padding.left}
				y1={yScale(tick)}
				x2={width - padding.right}
				y2={yScale(tick)}
				stroke="#4d4d4d"
				stroke-width="1"
				opacity="0.3"
			/>
		{/each}

		<!-- X-axis (Station names) -->
		<g class="x-axis">
			<line
				x1={padding.left}
				y1={height - padding.bottom}
				x2={width - padding.right}
				y2={height - padding.bottom}
				stroke="#191919"
				stroke-width="4"
			/>
			{#each data as point, i}
				<line
					x1={xScale(i)}
					y1={height - padding.bottom-4}
					x2={xScale(i)}
					y2={height - padding.bottom + 4}
					stroke="#5e5d5d"
					stroke-width="2"
				/>
				<rect
                        class="bar"
                        x={xScale(i)-barWidth/2}
                        y={0}
                        width={xScale(i)-xScale(i-1)}
                        height={Math.max(yScale(0))}
                        on:mouseover={(event) => {
                            selected_datapoint = point.properties["Fid"];
							selected_coordinates = point.geometry.coordinates;
							//selected_y = point.properties["Y"];
                            //selected_datapoint_i = i;
							//console.log(selected_x, "  ", selected_y)
							population = point.properties["Pop21"]
                            setValue(selected_datapoint, selected_coordinates, population)
                        }}
                        on:mouseout={() => {
                            selected_datapoint = undefined;
                        }}

                    />
				
				{#if point.properties.Location_N == null && point.properties.Distance%1000 === 0}
					<text
						x={xScale(i)}
						y={height - padding.bottom + 20}
						text-anchor="middle"
						fill="#5e5d5d"
						font-size="12"
						
					>
						{point.properties["Distance"]/1000 + "km"}
					</text>
					<!-- Station dots -->
					<circle
						r="4"
						cx={xScale(i)}
						cy={yScale(point.properties["Pop21"])}
						fill="#5e5d5d"
						stroke="#748F25"
						stroke-width="3"
					/>
				{:else if point.properties.Location_N == null}
					<circle
							r="4"
							cx={xScale(i)}
							cy={yScale(point.properties["Pop21"])}
							fill="#5e5d5d"
							stroke="#748F25"
							stroke-width="3"
						/>
				{/if}
				{#if point.properties.Location_N != null}
					<line
						x1={xScale(i)}
						y1={yScale(point.properties["Pop21"]+5000)}
						x2={xScale(i)}
						y2={height - padding.bottom}
						stroke="#5e5d5d"
						stroke-width="2"
						stroke-dasharray="2 6"

					/>
					<!-- Station dots -->
					<circle
						r="8"
						cx={xScale(i)}
						cy={yScale(point.properties["Pop21"])}
						fill="white"
						stroke="#748F25"
						stroke-width="3"
					/>
					<text
						x={xScale(i)}
						y={yScale(point.properties["Pop21"]+5000)}
						text-anchor="middle"
						fill="white"
						font-size="12"
						font-weight = "bold"
						transform={`rotate(-0), translate(0,0)`}
					>
						{point.properties["Location_N"]}
					</text>
				{/if}
				{#if point.properties.Fid == fid}
					<line
						x1={xScale(i)}
						y1={yScale(point.properties["Pop21"]+5000)}
						x2={xScale(i)}
						y2={height - padding.bottom}
						stroke="#F1C500"
						stroke-width="2"
						stroke-dasharray="2 6"

					/>
					<line
						x1={padding.left}
						y1={yScale(point.properties["Pop21"])}
						x2={width - padding.right}
						y2={yScale(point.properties["Pop21"])}
						stroke="#F1C500"
						stroke-width="2"
						stroke-dasharray="2 6"

					/>
					
					<!-- Station dots -->
					<circle
						r="10"
						cx={xScale(i)}
						cy={yScale(point.properties["Pop21"])}
						fill="#F1C500"
						stroke="#F1C500"
						stroke-width="3"
					/>
					<text
						x={xScale(i)}
						y={yScale(point.properties["Pop21"])+4}
						text-anchor="middle"
						font-weight = "bold"
						fill="black"
						font-size="11"
						
					>
						{Math.round(point.properties["Pop21"]/1000) + "K"}
					</text>
					<text
						x={xScale(i)}
						y={yScale(point.properties["Pop21"]+5000)}
						text-anchor="middle"
						fill="white"
						font-size="12"
						font-weight = "bold"
						transform={`rotate(-0), translate(0,0)`}
					>
						{point.properties["Location_N"]}
					</text>
				{/if}
			{/each}
		</g>
	</svg>
</div>

<style>
	.chart {
		width: 90vw;
		margin: 0 auto;
		background-color: var(--brandGray90);
		overflow-x: auto;
		scrollbar-width: 1px;
	}
	    .bar {

        fill: white;
        opacity: 0;
        cursor: pointer;
    }

	/* SCROLL BARS */
    ::-webkit-scrollbar {
        width: 1px;
		height: 10px;
    } /* Track */
    ::-webkit-scrollbar-track {
        box-shadow: inset 0 0 1px grey;
        border-radius: 0px;
    }

    /* Handle */
    ::-webkit-scrollbar-thumb {
        background: #41729f;
        border-radius: 2px;
    }

    /* Handle on hover */
    ::-webkit-scrollbar-thumb:hover {
        background: #41729f;
    }
	
</style>
