<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/positron.json";    
    import popPoints from "../data/pop-1km-500m-intervals.geo.json"
    import gtaMain from "../data/gta-main-lines.geo.json"
    import rtp from "../data/regional-transportation-plan-non-heavy-rail.geo.json"
    //import buffer from "../data/buffer-250m.geo.json"
    import LineChart from "../lib/line-chart.svelte"
    import Writing from "../lib/writing.svelte"

    
    
    let map;
    let fid = 0
    let fids = 0
    let pointx
    let pointy
    let mapHeight = "70vh"

    let selectedLine = "All"

    let railline = [
        "All",
        "Lakeshore East Line",
        "Lakeshore West Line",
        "Richmond Hill Line",
        "Milton Line",
        "Kitchener Line",
        "Stouville Line",
        "Freight Line 1",
        "Freight Line 2",
        "Freight Line 3",
        "Freight Line 4",
        "Freight Line 5",
        "Freight Line 6",
        "Freight Line 7"
    ]

    let perc0010 = false
    let perc1120 = false
    let perc2130 = false
    let perc3140 = false
    let perc4150 = false
    let perc5160 = false
    let perc6170 = false
    let perc7180 = false
    let perc8190 = false
    let perc9199 = false
    let percgroups = []

    function dataselect (){
        percgroups = []
        map.setFilter("popPoints-layer", ["all",[">", ["get", "Pop21"], 0],["<", ["get", "Pop21"], 70000]]);

            if(perc0010){
                percgroups.push(["all",[">", ["get", "Pop21"], 0],["<", ["get", "Pop21"], 42]]);
            }

            if(perc1120){
                percgroups.push(["all",[">", ["get", "Pop21"], 42],["<=", ["get", "Pop21"], 88]])
            }

            if(perc2130){
                percgroups.push(["all",[">", ["get", "Pop21"], 88],["<=", ["get", "Pop21"], 246]])
            }

            if(perc3140){
                percgroups.push(["all",[">", ["get", "Pop21"], 246],["<=", ["get", "Pop21"], 1086]])
            }

            if(perc4150){   
                percgroups.push(["all",[">", ["get", "Pop21"], 1086],["<=", ["get", "Pop21"], 2996]])
            }

            if(perc5160){
                percgroups.push(["all",[">", ["get", "Pop21"], 2996],["<=", ["get", "Pop21"], 4962]])
            }

            if(perc6170){
                percgroups.push(["all",[">", ["get", "Pop21"], 4962],["<=", ["get", "Pop21"], 6902]])
            }

            if(perc7180){
                percgroups.push(["all",[">", ["get", "Pop21"], 6902],["<=", ["get", "Pop21"], 9235]])
            }

            if(perc8190){
                percgroups.push(["all",[">", ["get", "Pop21"], 9235],["<=", ["get", "Pop21"], 14510]])
            }
            
            if(perc9199){
                percgroups.push(["all",[">", ["get", "Pop21"], 14510],["<=", ["get", "Pop21"], 70000]])
            }

            console.log(percgroups)
            var filter = ["any", ...percgroups];
            map.setFilter("popPoints-layer", filter);
            console.log(filter)
            if (filter.length == 1) {
            // This is to remove all filters when there is just one "any" in the filter
            map.setFilter("popPoints-layer", null);
        } 


    }

    async function rail_dropdown (){
        //this function controls the actions after a value (selectedLine) is selected in the drop down
        if (selectedLine == "All"){
            map.setFilter("popPoints-layer", null);
            map.setFilter("labels", null);
            map.setPaintProperty("popPoints-layer", "circle-radius", 6);
            map.setPaintProperty("popPoints-selected-layer", "circle-radius", 6);
            mapHeight = "70vh"
            
        }
        else{
            map.setFilter("popPoints-layer", ["==", ["get", "Name"], selectedLine]);
            //map.setFilter("buffer-250m-layer", ["==", ["get", "Name"], selectedLine]);
            map.setFilter("labels", ["==", ["get", "Name"], selectedLine]);
            map.setPaintProperty("popPoints-layer", "circle-radius", 10);
            map.setPaintProperty("popPoints-selected-layer", "circle-radius", 10);
            mapHeight = "50vh" // normal height for other lines
        }
    }



    function fidvalue(fids, pointx, pointy) {
        fid = fids
        pointx = pointx
        pointy = pointy
        //console.log(fid)
        map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
        map.setCenter([pointx, pointy])
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
            "interpolate",
            ["linear"],
            ["get", "Pop21"],
            0,
            "#055E5E", //0.10, colors[1],
            42,
            "#407E6E", //0.30, colors[3],
            88,
           "#7B9E7E", //0.50, colors[5],
            246,
            "#B6BE8E", //0.70, colors[7],
            1086,
            "#F1DE9F", //0.90, colors[9],
            2996,
            "#ECBD80",
            4962,
            "#E79D62",
            6902,
            "#E37D44",
            9235,
            "#DE5D26",
            14510,
            "#DA3D08"

        ];

        

    //async function filtre ( )
    
    onMount(async () => {
        map = new maplibregl.Map({
            container: "map",
            style: map_styles, //'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
            center: [-79.4, 43.69], // starting position
            minZoom: 7,
            zoom: 9,
            maxZoom: 15,
            scrollZoom: true,
            attributionControl: false,
        });
        
        
        //console.log(stations)
        map.on("load", () => {
            const layers = map.getStyle().layers;

            
            //adding the station, the data is determined either the "origin" or "destination" data input.
            map.addSource("popPoints", {
                type: "geojson",
                data: popPoints,
            });

            map.addSource("main-lines", {
                type: "geojson",
                data: gtaMain,
            });

            map.addSource("rtp", {
                type: "geojson",
                data: rtp,
            });

            map.addLayer({
                id: "rtp-id",
                type: "line",
                source: "rtp",
                minzoom: 9,
                paint: {
                    "line-color":[
                        "match",
                        ["get", "System"],
                        "Subway", "#4a4e69",
                        "LRT",  "#9a8c98",
                        "BRT",  "#c9ada7",
                        "Priority Bus", "#cad2c5",
                        "rgba(0,0,0,0)"
                    ],
                    "line-width": [
                        "match",
                        ["get", "System"],
                        "Subway", 3,
                        "LRT",  2,
                        "BRT",  2,
                        "Priority Bus", 0.5,
                        0
                    ],
                    "line-dasharray": [5, 1]
                },
            }, "place_town");


            map.addLayer({
                id: "main-lines-id",
                type: "line",
                source: "main-lines",
                paint: {
                    "line-color":"#d6ccc2",
                    "line-width": 4
                },
            });

            map.addLayer({
                id: "popPoints-layer",
                type: "circle",
                source: "popPoints",
                paint: {
                    "circle-radius": 6,
                    "circle-stroke-width": 0,
                    "circle-stroke-color": "#0D534D",
                    "circle-color": circlecolor_perc,
                    "circle-opacity": 1,
                },
                
            });

            map.addLayer({
                id: "popPoints-selected-layer",
                type: "circle",
                source: "popPoints",
                paint: {
                    "circle-radius": 6,
                    "circle-stroke-width": 4,
                    "circle-stroke-color": "#191919",
                    "circle-color": "rgba(0,0,0,0)",
                    "circle-opacity": 1,
                },
                filter : ["==", ["get", "Fid"], fid]
            });

            map.addLayer({
                id: "labels",
                type: "symbol",
                source: "popPoints",
                minzoom: 9,
                layout: {
                "text-field": ["concat", ["get", "Location_N"], 
                    ["case",["==", ["get", "Status"], null], "",  // if null → append nothing
                    ["concat", "\n (", ["get", "Status"], ")"]  // else → add (ID)
                    ]], // <-- use your field here
                "text-size": 12,
                "text-offset": [3, 0],
                "text-anchor": "top"
                },
                paint: {
                "text-color": "#000000",
                "text-halo-color": "#ffffff",
                "text-halo-width": 2
                },
                filter : ["==", ["get", "Name"], selectedLine]
            });


            map.on("mouseenter", "popPoints-layer", (e) => {
                map.getCanvas().style.cursor = "pointer";
                fid = e.features[0].properties['Fid']
                //console.log(e.features[0].properties['Name'])
                //console.log(e.features[0].properties['Pop21'])
                //map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
                map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
            });
            map.on("mouseleave", "popPoints-layer", () => {
                map.getCanvas().style.cursor = "";
            });
            
            // Move all layers of type "symbol" to the top
            for (const layer of layers) {
                if (layer.type === "symbol") {
                    //console.log(layer)
                map.moveLayer(layer.id);
                }
            }
            
        });
    });
</script>

<div id="map" class="map" style="height: {mapHeight}"/>

<div class="header">

    <select bind:value={selectedLine} on:change={rail_dropdown}>
            {#each railline as rail}
                <option>{rail}</option>
            {/each}
    </select>

</div>
{#if selectedLine != "All"}
<div class="charts">
    {#key [selectedLine]}
    <LineChart
        on:change = {(e)=>{
            fids = e.detail.dispatch_fid;
            pointx = e.detail.selected_x;
            pointy = e.detail.selected_y;
            //console.log(fids, " ", pointx," ",pointy)
            fidvalue(fids, pointx, pointy);}
        }
        lineName={selectedLine}
        jsondata = {popPoints.features}
        fid = {fid}
    />
    {/key}


</div>

{:else}
<div class="button-container">
        <button class="application-button"
            on:click={() => {
                perc0010 = !perc0010;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc0010
                ? 'rgba(5, 94, 94, 1)': 'rgba(5, 94, 94, 0.2)'}; color: {perc0010 ? 'white' : 'black'}"
            >0 - 10%
        </button>
        <button class="application-button"
            on:click={() => {
                perc1120 = !perc1120;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc1120
                ? 'rgba(64, 126, 110, 1)': 'rgba(64, 126, 110, 0.2)'}; color: {perc1120 ? 'white' : 'black'}"
            >11 - 20%
        </button>
        <button class="application-button"
            on:click={() => {
                perc2130 = !perc2130;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc2130
                ? 'rgba(123, 158, 126, 1)': 'rgba(123, 158, 126, 0.2)'}; color: {perc2130 ? 'black' : 'black'}"
            >21 - 30%
        </button>
        <button class="application-button"
            on:click={() => {
                perc3140 = !perc3140;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc3140
                ? 'rgba(182, 190, 142, 1)': 'rgba(182, 190, 142, 0.2)'}; color: {perc3140 ? 'black' : 'black'}"
            >31 - 40%
        </button>
        <button class="application-button"
            on:click={() => {
                perc4150 = !perc4150;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc4150
                ? 'rgba(241, 222, 159, 1)': 'rgba(241, 222, 159, 0.2)'}; color: {perc4150 ? 'black' : 'black'}"
            >41 - 50%
        </button>
        <button class="application-button"
            on:click={() => {
                perc5160 = !perc5160;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc5160
                ? 'rgba(236, 189, 128, 1)': 'rgba(236, 189, 128, 0.2)'}; color: {perc5160 ? 'black' : 'black'}"
            >51 - 60%
        </button>
        <button class="application-button"
            on:click={() => {
                perc6170 = !perc6170;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc6170
                ? 'rgba(231, 157, 98, 1)': 'rgba(231, 157, 98, 0.2)'}; color: {perc6170 ? 'black' : 'black'}"
            >61 - 70%
        </button>
        <button class="application-button"
            on:click={() => {
                perc7180 = !perc7180;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc7180
                ? 'rgba(227, 125, 68, 1)': 'rgba(227, 125, 68, 0.2)'}; color: {perc7180 ? 'black' : 'black'}"
            >71 - 80%
        </button>
        <button class="application-button"
            on:click={() => {
                perc8190 = !perc8190;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc8190
                ? 'rgba(222, 93, 38, 1)': 'rgba(222, 93, 38, 0.2)'}; color: {perc8190 ? 'white' : 'black'}"
            >81 - 90%
        </button>
        <button class="application-button"
            on:click={() => {
                perc9199 = !perc9199;
                //allFilter = false;
                dataselect();
            }}
            style="background-color: {perc9199
                ? 'rgba(218, 61, 8, 1)': 'rgba(218, 61, 8, 0.2)'}; color: {perc9199 ? 'white' : 'black'}"
            >91 - 100%
        </button>
        
    

</div>
{/if}



   

<style>
    .map {
        left: 5vw;
        top: 8vh;
        width: 90vw;
        height: 70vh;
        position: absolute;
        overflow: hidden;
    }
    .header {
        left: 5vw;
        top: 2vh;
        max-width: 90vw;
        position: absolute;
        overflow: hidden;
    }

    .header select {
        width: auto;
        height: 4vh;
        font-size: 20px;
        color: var(--brandYellow);
        font-family: TradeGothicBold;
        background: var(--brandGray90);
        border: none;
    }
    .charts{
        left: 5vw;
        top: 59vh;
        max-width: 90vw;
        position: absolute;
        overflow: hidden;
        padding-bottom: 20px;
    }
    .button-container {
        position: absolute;
        top: 79vh;           /* same vertical position as before */
        left: 5vw;           /* align with map left edge */
        width: 90vw;         /* match map width */
        display: flex;       /* enable flex layout */
        justify-content: space-between; /* spread buttons evenly */
        flex-wrap: wrap;     /* allow wrapping on smaller screens */
        gap: 5px;            /* small gap between buttons */
    }

    .application-button {
        flex: 1;             /* let buttons expand evenly */
        min-width: 80px;     /* prevents them from becoming too small */
        font-size: 12px;
        height: 30px;
        border: none;
        font-weight: bold;
        cursor: pointer;
    }

    .legend-labels {
        display: flex;
        justify-content: space-between;
        font-size: 12px;
        margin-top: 4px;
    }

    .legend-labels span {
        flex: 1;
        text-align: center;
    }

</style>