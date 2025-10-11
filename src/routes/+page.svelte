<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/positron.json";    
    import popPoints from "../data/pop-1km-500m-intervals.geo.json"
    import gtaMain from "../data/gta-main-lines.geo.json"
    import rtp from "../data/regional-transportation-plan-non-heavy-rail.geo.json"
    //import buffer from "../data/buffer-250m.geo.json"
    import LineChart from "../lib/line-chart.svelte"
    import * as d3 from "d3";

    
    
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
                minzoom: 10,
                paint: {
                    "line-color":[
                        "match",
                        ["get", "System"],
                        "Subway", "#D97D55",
                        "LRT",  "#52796f",
                        "BRT",  "#84a98c",
                        "Priority Bus", "#cad2c5",
                        "#D3D3D3"
                    ],
                    "line-width": 2
                },
            }, "place_town");


            map.addLayer({
                id: "main-lines-id",
                type: "line",
                source: "main-lines",
                paint: {
                    "line-color":"#5e5d5d",
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
            /*
            // Move all layers of type "symbol" to the top
            for (const layer of layers) {
                if (layer.type === "symbol") {
                    //console.log(layer)
                map.moveLayer(layer.id);
                }
            }
            */
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
<div class="legend">
        <span class="rect" style = "background-color: #055E5E">Bottom 10%</span>    
        <span class="rect" style = "background-color: #407E6E">11-20%</span>
        <span class="rect" style = "background-color: #7B9E7E">21-30%</span>
        <span class="rect" style = "background-color: #B6BE8E">31-40%</span>
        <span class="rect" style = "background-color: #F1DE9F">41-50%</span>
        <span class="rect" style = "background-color: #ECBD80">51-60%</span>
        <span class="rect" style = "background-color: #E79D62">61-70%</span>
        <span class="rect" style = "background-color: #E37D44">71-80%</span>
        <span class="rect" style = "background-color: #DE5D26">81-90%</span>
        <span class="rect" style = "background-color: #DA3D08">Top 10%</span>
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
        /*display: flex;
        flex-wrap: wrap;
        align-items: center;
        padding-left: 10px;
        color: #f9f6f1;
        font-size: 20px;*/

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
    
    

    .legend {
        
        top: 79vh;
        left: 5vw;
        height: 20vh;
        width: 90vw;
        max-width: 90vw;
        background-color: rgba(255, 255, 255, 0);
        padding: 10px;;
        /*border-radius: 5px;*/
        
        position: absolute;
        text-align: center;
        
        box-sizing: border-box; /* <-- include padding in width calculation */
        display: flex;         /* make children flex items */
        flex-wrap: wrap;       /* wrap if needed */
    }
    .rect {
        flex: 1;               /* each rect takes equal share of available width */
        margin: 0 2px;         /* spacing between boxes */
        height: 15vh;           /* fixed height */
        
        display: flex;          /* use flex to center text */
        align-items: center;    /* vertical centering */
        justify-content: center;/* horizontal centering */

        font-size: 14px;
        font-weight: bolder;

        overflow: hidden;       /* prevents text overflow */
        white-space: nowrap;    /* keep text in one line */
        text-overflow: ellipsis;/* show "..." if text is too long */
        background-color: #bbb;
        color: #000;

        border-radius: 3px;     /* optional rounding */
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