<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/positron.json";    
    import popPoints from "../data/pop-1km-500m-intervals.geo.json"
    import gtaMain from "../data/gta-main-lines.geo.json"
    import rtp from "../data/regional-transportation-plan-non-heavy-rail.geo.json"
    import buffer from "../data/buffer-250m.geo.json"
    import LineChart from "../lib/line-chart.svelte"
    import * as d3 from "d3";

    
    
    let map;
    let fid = 0
    let fids = 0

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
            console.log(selectedLine)
            map.setFilter("popPoints-layer", null);
            map.setFilter("labels", null);
        }
        else{
            map.setFilter("popPoints-layer", ["==", ["get", "Name"], selectedLine]);
            map.setFilter("buffer-250m-layer", ["==", ["get", "Name"], selectedLine]);
            map.setFilter("labels", ["==", ["get", "Fid"], fid]);
        }
    }

    function fidvalue(fids) {
        fid = fids
        console.log(fid)
        map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
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
            minZoom: 8,
            maxZoom: 30,
            scrollZoom: true,
            attributionControl: false,
        });

        //console.log(stations)
        map.on("load", () => {
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
            
            map.addSource("buffer-250m", {
                type: "geojson",
                data: buffer,
            });

            map.addLayer({
                id: "rtp-id",
                type: "line",
                source: "rtp",
                paint: {
                    "line-color":[
                        "match",
                        ["get", "System"],
                        "LRT",  "#D3D3D3",
                        "BRT",  "#D3D3D3",
                        "Subway", "#D3D3D3",
                        "Priority Bus", "#D3D3D3",
                        "#D3D3D3"
                    ],
                    "line-width": 2
                },
            });

            map.addLayer({
                id: "buffer-250m-layer",
                type: "fill",
                source: "buffer-250m",
                paint: {
                    "fill-color":"#000000",
                    "fill-opacity": 0
                },
                filter : ["==", ["get", "Name"], selectedLine]
            });

            map.addLayer({
                id: "main-lines-id",
                type: "line",
                source: "main-lines",
                paint: {
                    "line-color":"#000000",
                    "line-width": 6
                },
            });

            map.addLayer({
                id: "popPoints-layer",
                type: "circle",
                source: "popPoints",
                paint: {
                    "circle-radius": 6,
                    "circle-stroke-width": 0.4,
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
                    "circle-radius": 6,
                    "circle-stroke-width": 4,
                    "circle-stroke-color": "#F1C500",
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

               /* 
            map.on("click", "bike-count", (e) => {
                station = e.features[0].properties["Name"];
                map.setFilter("bike-clicked", ["==", "Name", station]),
                    (capacity = e.features[0].properties["Capacity"]);
                bikecount = e.features[0].properties[daytime];
                stationIndex = stationNames.indexOf(station);
            });*/
            map.on("mouseenter", "buffer-250m-layer", (e) => {
                map.getCanvas().style.cursor = "pointer";
                fid = e.features[0].properties['Fid']
                console.log(e.features[0].properties['Name'])
                console.log(e.features[0].properties['Pop21'])
                //map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
                map.setFilter("popPoints-selected-layer", ["==", ["get", "Fid"], fid]);
            });
            map.on("mouseleave", "popPoints-layer", () => {
                map.getCanvas().style.cursor = "";
            });
            /*
            for (let i = 0; i < bikes_day1.features.length; i++) {
                //console.log(i,bikes_day1.features[i].properties.Name)
                stationNames.push(bikes_day1.features[i].properties.Name);
            }

            stationIndex = stationNames.indexOf(station);
            bikecount = bikes_day1.features[stationIndex].properties[daytime];
            */
           console.log(fid)
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
            console.log(fids)
            fidvalue(fids);}
            
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
        height: 50vh;
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
        background: white;
        border: none;
    }
    .charts{
        left: 5vw;
        top: 55vh;
        max-width: 90vw;
        position: absolute;
        overflow: hidden;
    }
    


</style>