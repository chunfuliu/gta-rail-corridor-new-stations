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


    let selectedLine = "Lakeshore East Line"

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
        }
        else{
            map.setFilter("popPoints-layer", ["==", ["get", "Name"], selectedLine]);
            //map.setFilter("buffer-250m-layer", ["==", ["get", "Name"], selectedLine]);
            map.setFilter("labels", ["==", ["get", "Name"], selectedLine]);
        }
    }

    function fidvalue(fids, pointx, pointy) {
        fid = fids
        pointx = pointx
        pointy = pointy
        //console.log(fid)
        map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
        const bounds = map.getBounds();
        const extent = {
            west: bounds.getWest(),
            south: bounds.getSouth(),
            east: bounds.getEast(),
            north: bounds.getNorth(),
            };

            console.log(extent);
            if (pointx < extent.west || pointx > extent.east || pointy < extent.south || pointy > extent.north){
                map.setCenter([pointx, pointy]);
            }
            console.log(pointx, pointy)
    }

    let circlecolor_perc = [
            "interpolate",
            ["linear"],
            ["get", "Pop21"],
            0,
            "#055E5E", //0.10, colors[1],
            45,
            "#407E6E", //0.30, colors[3],
            98,
           "#7B9E7E", //0.50, colors[5],
            311,
            "#B6BE8E", //0.70, colors[7],
            1567,
            "#F1DE9F", //0.90, colors[9],
            3564,
            "#ECBD80",
            5785,
            "#E79D62",
            7888,
            "#E37D44",
            11094,
            "#DE5D26",
            18561,
            "#DA3D08"

        ];

    //async function filtre ( )
    
    onMount(async () => {
        map = new maplibregl.Map({
            container: "map",
            style: map_styles, //'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
            center: [-79.4, 43.69], // starting position
            minZoom: 9,
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
                    "circle-radius": 10,
                    "circle-stroke-width": 0,
                    "circle-stroke-color": "#0D534D",
                    "circle-color": circlecolor_perc,
                    "circle-opacity": 1,
                },
                filter : ["==", ["get", "Name"], selectedLine]
            });

            map.addLayer({
                id: "popPoints-selected-layer",
                type: "circle",
                source: "popPoints",
                paint: {
                    "circle-radius": 10,
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

<div id="map" class="map" />

<div class="header">

    <select bind:value={selectedLine} on:change={rail_dropdown}>
            {#each railline as rail}
                <option>{rail}</option>
            {/each}
    </select>

</div>
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



   

<style>
    .map {
        left: 5vw;
        top: 8vh;
        max-height: 50vh;
        width: 90vw;
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
    }
    


</style>