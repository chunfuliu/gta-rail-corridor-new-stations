<script>
	import { onMount } from 'svelte';
	import maplibregl from 'maplibre-gl';
	import map_styles from '../lib/positron.json';
	import popPoints from '../data/pop-1km-500m-intervals.geo.json';
	import gtaMain from '../data/gta-main-lines.geo.json';
	import rtp from '../data/regional-transportation-plan-non-heavy-rail.geo.json';
	import stations from '../data/stations-in-gta.geo.json';
	//import buffer from "../data/buffer-250m.geo.json"
	import LineChart from '../lib/line-chart.svelte';
	

	let map;
	let fid = 0;
	let fids = 0;
	let pointx;
	let pointy;
	let mapHeight = '70vh';
	let population;

	let selectedLine = 'All';

	let railline = [
		'All',
		'Lakeshore East Line',
		'Lakeshore West Line',
		'Richmond Hill Line',
		'Milton Line',
		'Kitchener Line',
		'Stouville Line',
		'Freight Line 1',
		'Freight Line 2',
		'Freight Line 3',
		'Freight Line 4',
		'Freight Line 5',
		'Freight Line 6',
		'Freight Line 7'
	];

	let perc0010 = true;
	let perc1120 = true;
	let perc2130 = true;
	let perc3140 = true;
	let perc4150 = true;
	let perc5160 = true;
	let perc6170 = true;
	let perc7180 = true;
	let perc8190 = true;
	let perc9199 = true;
	let percgroups = [];

	function dataselect() {
		percgroups = [];
		map.setFilter('popPoints-layer', [
			'all',
			['>', ['get', 'Pop21'], 0],
			['<', ['get', 'Pop21'], 70000]
		]);

		if (perc0010) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 0], ['<', ['get', 'Pop21'], 42]]);
		}
		if (perc1120) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 42], ['<=', ['get', 'Pop21'], 88]]);
		}
		if (perc2130) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 88], ['<=', ['get', 'Pop21'], 246]]);
		}
		if (perc3140) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 246], ['<=', ['get', 'Pop21'], 1086]]);
		}
		if (perc4150) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 1086], ['<=', ['get', 'Pop21'], 2996]]);
		}
		if (perc5160) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 2996], ['<=', ['get', 'Pop21'], 4962]]);
		}
		if (perc6170) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 4962], ['<=', ['get', 'Pop21'], 6902]]);
		}
		if (perc7180) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 6902], ['<=', ['get', 'Pop21'], 9235]]);
		}
		if (perc8190) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 9235], ['<=', ['get', 'Pop21'], 14510]]);
		}
		if (perc9199) {
			percgroups.push(['all', ['>', ['get', 'Pop21'], 14510], ['<=', ['get', 'Pop21'], 70000]]);
		}

		console.log(percgroups);
		var filter = ['any', ...percgroups];
		map.setFilter('popPoints-layer', filter);
		console.log(filter);
		if (filter.length == 1) {
			// This is to remove all filters when there is just one "any" in the filter
			map.setFilter('popPoints-layer', null);
			perc0010 = true;
			perc1120 = true;
			perc2130 = true;
			perc3140 = true;
			perc4150 = true;
			perc5160 = true;
			perc6170 = true;
			perc7180 = true;
			perc8190 = true;
			perc9199 = true;
		}
	}

	async function rail_dropdown() {
		//this function controls the actions after a value (selectedLine) is selected in the drop down
		if (selectedLine == 'All') {
			map.setFilter('popPoints-layer', null);
			map.setFilter('labels', null);
			map.setPaintProperty('popPoints-layer', 'circle-radius', 6);
			map.setPaintProperty('popPoints-selected-layer', 'circle-radius', 6);
			mapHeight = '70vh';
		} else {
			map.setFilter('popPoints-layer', ['==', ['get', 'Name'], selectedLine]);
			//map.setFilter("buffer-250m-layer", ["==", ["get", "Name"], selectedLine]);
			map.setFilter('labels', ['==', ['get', 'Name'], selectedLine]);
			map.setPaintProperty('popPoints-layer', 'circle-radius', 10);
			map.setPaintProperty('popPoints-selected-layer', 'circle-radius', 10);
			mapHeight = '50vh'; // normal height for other lines
		}
	}

	function fidvalue(fids, pointx, pointy) {
		fid = fids;
		pointx = pointx;
		pointy = pointy;
		//console.log(fid)
		map.setFilter('popPoints-selected-layer', ['==', ['get', 'Fid'], fid]);
		map.setCenter([pointx, pointy]);
		/*const bounds = map.getBounds();
        const extent = {
            west: bounds.getWest(),
            south: bounds.getSouth(),
            east: bounds.getEast(),
            north: bounds.getNorth(),
            };

           //console.log(extent);
            if (pointx < extent.west || pointx > extent.east || pointy < extent.south || pointy > extent.north){
                map.setCenter([pointx, pointy]);
            }
            //console.log(pointx, pointy)*/
	}

	let circlecolor_perc = [
		'step',
		['get', 'Pop21'],

		 			'#055E5E', //0.10, colors[1],
		42, 		'#407E6E', //0.30, colors[3],
		88, 		'#7B9E7E', //0.50, colors[5],
		246, 		'#B6BE8E', //0.70, colors[7],
		1086, 		'#F1DE9F', //0.90, colors[9],
		2996, 		'#ECBD80',
		4962, 		'#E79D62',
		6902, 		'#E37D44',
		9235, 		'#DE5D26',
		14510, 		'#DA3D08'
	];

	//async function filtre ( )

	onMount(async () => {
		
		map = new maplibregl.Map({
			container: 'map',
			style: map_styles, //'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
			center: [-79.4, 43.69], // starting position
			minZoom: 7,
			zoom: 8,
			maxZoom: 15,
			scrollZoom: true,
			attributionControl: false
		});
		
		//console.log(stations)
		map.on('load', () => {
			const layers = map.getStyle().layers;

			//adding the station, the data is determined either the "origin" or "destination" data input.
			map.addSource('popPoints', {
				type: 'geojson',
				data: popPoints
			});

			map.addSource('stations', {
				type: 'geojson',
				data: stations
			});

			map.addSource('main-lines', {
				type: 'geojson',
				data: gtaMain
			});

			map.addSource('rtp', {
				type: 'geojson',
				data: rtp
			});

			map.addLayer(
				{
					id: 'rtp-id',
					type: 'line',
					source: 'rtp',
					minzoom: 9,
					paint: {
						'line-color': [
							'match',
							['get', 'System'],
							'Subway',
							'#4a4e69',
							'LRT',
							'#9a8c98',
							'BRT',
							'#c9ada7',
							
							'#86B0BD'
						],
						'line-width': [
							'match',
							['get', 'System'],
							'Subway',
							3,
							'LRT',
							2,
							'BRT',
							2,
							
							2
							
						],
						'line-dasharray': [5, 1]
					}
				},
				'place_town'
			);
			map.addLayer({
				id: 'stations-layer',
				type: 'circle',
				source: 'stations',
				minzoom: 10,
				paint: {
					'circle-radius': 5,
					'circle-stroke-width': 0,
					'circle-stroke-color': '#0D534D',
					'circle-color': [
						'match',
						['get', 'TECHNOLOGY'],
						'Subway',
						'#4a4e69',
						'LRT',
						'#9a8c98',
						'BRT',
						'#c9ada7',
						'Priority Bus',
						'#cad2c5',
						'rgba(255,255,255,1)'
					],
					
					'circle-opacity': 1
				}
			});

			map.addLayer({
				id: 'main-lines-id',
				type: 'line',
				source: 'main-lines',
				paint: {
					'line-color': '#d6ccc2',
					'line-width': 4
				}
			});

			map.addLayer({
				id: 'labels-rtp',
				type: 'symbol',
				source: 'rtp',
				minzoom: 10,
				layout: {
					"symbol-placement": "line",
					"text-font": ["Noto Sans Regular"],
					"text-field": '{NAME} {System}', // part 2 of this is how to do it
					"text-size": 9,
					'text-offset': [0, 1]
				},
				paint: {
					'text-color': '#5e5d5d',
					'text-halo-color': '#ffffff',
					'text-halo-width': 2
				}
			});

			map.addLayer({
				id: 'popPoints-layer',
				type: 'circle',
				source: 'popPoints',
				paint: {
					'circle-radius': 6,
					'circle-stroke-width': 0,
					'circle-stroke-color': '#0D534D',
					'circle-color': circlecolor_perc,
					'circle-opacity': 1
				}
			});

			

			map.addLayer({
				id: 'popPoints-selected-layer',
				type: 'circle',
				source: 'popPoints',
				paint: {
					'circle-radius': 6,
					'circle-stroke-width': 4,
					'circle-stroke-color': '#191919',
					'circle-color': 'rgba(0,0,0,0)',
					'circle-opacity': 1
				},
				filter: ['==', ['get', 'Fid'], fid]
			});

			map.addLayer({
				id: 'labels',
				type: 'symbol',
				source: 'popPoints',
				minzoom: 9,
				layout: {
					'text-field': [
						'concat',
						['get', 'Location_N'],
						[
							'case',
							['==', ['get', 'Status'], null],
							'', // if null → append nothing
							['concat', '\n (', ['get', 'Status'], ')'] // else → add (ID)
						]
					], // <-- use your field here
					'text-size': 12,
					'text-offset': [3, 0],
					'text-anchor': 'top'
				},
				paint: {
					'text-color': '#000000',
					'text-halo-color': '#ffffff',
					'text-halo-width': 2
				},
				filter: ['==', ['get', 'Name'], selectedLine]
			});

			

			map.on('mouseenter', 'popPoints-layer', (e) => {
				map.getCanvas().style.cursor = 'pointer';
				fid = e.features[0].properties['Fid'];
				//console.log(e.features[0].properties['Name'])
				population = e.features[0].properties['Pop21'];
				//map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
				map.setFilter('popPoints-selected-layer', ['==', ['get', 'Fid'], fid]);
			});
			map.on('mouseleave', 'popPoints-layer', () => {
				map.getCanvas().style.cursor = '';
			});

			map.on('touchstart', 'popPoints-layer', (e) => {
				fid = e.features[0].properties['Fid'];
				//console.log(e.features[0].properties['Name'])
				population = e.features[0].properties['Pop21'];
				//map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
				map.setFilter('popPoints-selected-layer', ['==', ['get', 'Fid'], fid]);
			});

			// Move all layers of type "symbol" to the top
			for (const layer of layers) {
				if (layer.type === 'symbol') {
					//console.log(layer)
					map.moveLayer(layer.id);
				}
			}
		});
	});
</script>
<div class="header">
	<select bind:value={selectedLine} on:change={rail_dropdown}>
		{#each railline as rail}
			<option>{rail}</option>
		{/each}
	</select>
</div>
<div id="map" class="map" style="height: {mapHeight}" />



{#if selectedLine != 'All'}
	<div class="charts">
		{#key [selectedLine]}
			<LineChart
				on:change={(e) => {
					fids = e.detail.dispatch_fid;
					pointx = e.detail.selected_x;
					pointy = e.detail.selected_y;
					population = e.detail.population;
					//console.log(fids, " ", pointx," ",pointy)
					fidvalue(fids, pointx, pointy);
				}}
				lineName={selectedLine}
				jsondata={popPoints.features}
				{fid}
			/>
		{/key}
	</div>
	<h3>
		Population: {Math.round(population / 10) / 100 + 'k '}, scroll the map and chart to view more!
	</h3>
	<!--

    
    {#if selectedLine == "Lakeshore East Line"}
    <p> On the Lakeshore East Line, stops can be added every kilometre, between Union Station and Danforth to expand 
        transit services to areas including the Distillery District and the East End. Passing the Danforth Station, population
        declines to around 13k residents around the rail corridor, until Guildwood GO station. This segment can have a station
        every two kilometres. In the future, more stations can be added between Guildwood and Pickering Station.
    </p>
    {:else if selectedLine == "Lakeshore West Line"}

    
    {:else if selectedLine == "Richmond Hill Line"}
    <p>
        On the Richmond Hill Line, stations can be added along the Don River 
    </p>
    {:else if selectedLine == "Freight Line 3"}
    {:else if selectedLine == "Freight Line 3"}
    {:else if selectedLine == "Freight Line 3"}
    {:else if selectedLine == "Freight Line 3"}
    <p> Freight Line 3 runs between Brampton and Pickering. The portion of the rail line 
        with higher existing population is roughly between the two arms of TTC Line 1 Subway. 
        However, this portion of the railway also runs parallel to Steeles Ave, where a BRT / LRT
        line is proposed. 
    </p>
    
    
    {/if}

</div>-->
{:else}

	<div class="button-container">
		<button
			class="application-button"
			on:click={() => {
				perc0010 = !perc0010;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc0010
				? 'rgba(5, 94, 94, 1)'
				: 'rgba(5, 94, 94, 0.2)'}; color: {perc0010 ? 'white' : 'black'}"
			>0 - 10%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc1120 = !perc1120;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc1120
				? 'rgba(64, 126, 110, 1)'
				: 'rgba(64, 126, 110, 0.2)'}; color: {perc1120 ? 'white' : 'black'}"
			>11 - 20%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc2130 = !perc2130;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc2130
				? 'rgba(123, 158, 126, 1)'
				: 'rgba(123, 158, 126, 0.2)'}; color: {perc2130 ? 'black' : 'black'}"
			>21 - 30%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc3140 = !perc3140;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc3140
				? 'rgba(182, 190, 142, 1)'
				: 'rgba(182, 190, 142, 0.2)'}; color: {perc3140 ? 'black' : 'black'}"
			>31 - 40%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc4150 = !perc4150;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc4150
				? 'rgba(241, 222, 159, 1)'
				: 'rgba(241, 222, 159, 0.2)'}; color: {perc4150 ? 'black' : 'black'}"
			>41 - 50%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc5160 = !perc5160;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc5160
				? 'rgba(236, 189, 128, 1)'
				: 'rgba(236, 189, 128, 0.2)'}; color: {perc5160 ? 'black' : 'black'}"
			>51 - 60%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc6170 = !perc6170;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc6170
				? 'rgba(231, 157, 98, 1)'
				: 'rgba(231, 157, 98, 0.2)'}; color: {perc6170 ? 'black' : 'black'}"
			>61 - 70%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc7180 = !perc7180;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc7180
				? 'rgba(227, 125, 68, 1)'
				: 'rgba(227, 125, 68, 0.2)'}; color: {perc7180 ? 'black' : 'black'}"
			>71 - 80%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc8190 = !perc8190;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc8190
				? 'rgba(222, 93, 38, 1)'
				: 'rgba(222, 93, 38, 0.2)'}; color: {perc8190 ? 'white' : 'black'}"
			>81 - 90%
		</button>
		<button
			class="application-button"
			on:click={() => {
				perc9199 = !perc9199;
				//allFilter = false;
				dataselect();
			}}
			style="background-color: {perc9199
				? 'rgba(218, 61, 8, 1)'
				: 'rgba(218, 61, 8, 0.2)'}; color: {perc9199 ? 'white' : 'black'}"
			>91 - 100%
		</button> <br />
	</div>
	<h1>FINDING FUTURE STATIONS: POPULATION NEAR RAIL CORRIDORS</h1>


	<div class="column-text">
		<h2>Click the Buttons & Play with the Drop Down</h2>
		<p>
			Click the buttons above to filter the points in the network. The colouring of the dots
			represent the number of people living near each point on the map. Red dots indicate areas with
			higher populations, while teal dots represent areas with lower populations. The colour scale
			is based on ranking the data from the lowest to highest values and dividing it into 10 groups.
			In other words, the red dots correspond to some of the most populated areas along the rail
			network, whereas the teal dots correspond to the least populated areas.
		</p>
		<h2>Why Make This Map?</h2>
		<p>
			Sometimes when I open Google Maps, I get distracted by the extensive network of rail tracks
			that run through Toronto. Every time this happens, I find myself thinking, “How nice would it
			be to have a station here… and here… and here…” (and another 20 minutes of this). I’ve also
			heard many people say that the rail line running through midtown Toronto would be ideal for an
			east–west rail service in Toronto. Given new transit lines are time consuming and expensive to
			build, is there a way for the GTA to leverage existing railway infrastructure to increase
			transit capacity? That question led me to explore: How many people live within 1 km of our
			existing rail tracks in the Greater Toronto Area? The findings suggest that there are many
			locations with great potential for adding new stations. Even in areas with lower residential
			populations, nearby industrial lands could attract a significant number of workers who might
			benefit from having a station in the area (this is my best guess, since I don’t have access to
			spatial employment data across the GTA). One caveat, however, is that I’m not a transportation
			expert—there are many other factors that need to be considered when assessing the feasibility
			of a station location. In my mind, these trains don’t have to be long. They could be something
			like Montreal’s REM or Vancouver’s skyTrain, serving at medium-capacity, automated, and
			frequent. The existing GO trains could then function as express services, similar to the
			express trains we see on the Lakeshore West Line today.
		</p>

		<h2>Methodology</h2>
		<p>
			The dots are generated along existing rail lines at 500-metre intervals. The population for
			each dot represents the number of people living within 1 kilometre of that point.

			The passenger lines follow the existing Metrolinx passenger rail service routes, as well as
			the proposed ones. This map also includes rail lines that currently do not have passenger
			service, such as freight rail lines. The extent of the rail network shown is limited to the
			boundaries of the Greater Toronto Area.
		</p>
	</div>
{/if}

<style>
	.map {
		left: 5%;
		top: 5%;
		width: 90%;
		height: 70vh;
		position: relative;
		overflow: hidden;
	}
	.column-text {
		left: 5%;
		width: 90%;
		position: relative;
	}
	.column-text p {
		margin-top: 0; /* Removes default top margin for paragraphs */
		padding-top: 0; /* Removes default top padding for paragraphs */
	}
	.column-text h2 {
		margin-top: 0; /* Removes default top margin for paragraphs */
		padding-top: 0; /* Removes default top padding for paragraphs */
	}
	.header {
		left: 5%;
		top: 5%;
		max-width: 90vw;
		position: relative;
		overflow: hidden;
	}

	.header select {
		width: auto;
		height: auto;
		font-size: 20px;
		color: var(--brandWhite);
		font-family: TradeGothicBold;
		background: var(--brandDarkGreen);
		border: none;
	}
	.charts {
		left: 5%;
		top: 5%;
		max-width: 90%;
		position: relative;
		overflow: hidden;
		display: block;
		padding-bottom: 0px;
	}
	.button-container {
		position: relative; /* same vertical position as before */
		left: 5%; /* align with map left edge */
		width: 90%; /* match map width */
		display: flex; /* enable flex layout */
		justify-content: space-between; /* spread buttons evenly */
		flex-wrap: wrap; /* allow wrapping on smaller screens */
		gap: 0px; /* small gap between buttons */
	}

	.application-button {
		flex: 1; /* let buttons expand evenly */
		min-width: 80px; /* prevents them from becoming too small */
		font-size: 12px;
		height: 30px;
		border: none;
		font-weight: bold;
		cursor: pointer;
	}
	h1 {
		position: relative;
		max-width: 90%;
		left: 5%;
	}
	h3{
		position: relative;
		max-width: 90%;
		left: 5%;
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
	@media (min-width:600px){
		.column-text {
			left: 5%;
			position: relative;
			display: block; /* ensures normal block layout */
			column-count: 2;
			column-gap: 20px;
			width: 90%;
			padding-top: 1vh;
			padding-bottom: 15px;
		}
	}
</style>
