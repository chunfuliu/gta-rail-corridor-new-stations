<script>
    import { onMount } from "svelte";
    import maplibregl from "maplibre-gl";
    import map_styles from "../lib/positron.json";    
    import popPoints from "../data/pop-1km-500m-intervals.geo.json"
    import gtaMain from "../data/gta-main-lines.geo.json"
    import rtp from "../data/regional-transportation-plan-non-heavy-rail.geo.json"
    import * as d3 from "d3";

    console.log(popPoints)
    let map;

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
    
    onMount(async () => {
        map = new maplibregl.Map({
            container: "map",
            style: map_styles, //'https://basemaps.cartocdn.com/gl/positron-gl-style/style.json',
            center: [-79.4, 43.69], // starting position
            minZoom: 10,
            maxZoom: 19,
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
                id: "rtp-id",
                type: "line",
                source: "rtp",
                paint: {
                    "line-color":[
                        "match",
                        ["get", "System"],
                        "LRT",  "#9C92B5",
                        "BRT",  "#9C92B5",
                        "Subway", "#5A4F7B",
                        "Priority Bus", "#C5BBE6",
                        "#DED5F0"

                    ],
                    "line-width": 6
                },
            });

            map.addLayer({
                id: "popPoints_id",
                type: "circle",
                source: "popPoints",
                paint: {
                    "circle-radius": 6,
                    "circle-stroke-width": 0.4,
                    "circle-stroke-color": "#0D534D",
                    "circle-color": circlecolor_perc,
                    "circle-opacity": 1,
                },
            });

            map.addLayer({
                id: "labels",
                type: "symbol",
                source: "popPoints",
                layout: {
                "text-field": ["concat", ["get", "Location_N"], 
                    ["case",["==", ["get", "Status"], null], "",  // if null → append nothing
                    ["concat", " (", ["get", "Status"], ")"]  // else → add (ID)
                    ]], // <-- use your field here
                "text-size": 14,
                "text-offset": [0, 0.5],
                "text-anchor": "top"
                },
                paint: {
                "text-color": "#000000",
                "text-halo-color": "#ffffff",
                "text-halo-width": 2
                }
            });

          

            
            /*
            map.addLayer({
                id: "bike-clicked",
                type: "circle",
                source: "station",
                filter: ["==", "Name", station],
                paint: {
                    "circle-radius": 5,
                    "circle-opacity": 0,
                    "circle-stroke-width": 3,
                    "circle-stroke-color": "#DC4633",
                },
            });

            map.on("click", "bike-count", (e) => {
                station = e.features[0].properties["Name"];
                map.setFilter("bike-clicked", ["==", "Name", station]),
                    (capacity = e.features[0].properties["Capacity"]);
                bikecount = e.features[0].properties[daytime];
                stationIndex = stationNames.indexOf(station);
            });
            map.on("mouseenter", "bike-count", (e) => {
                map.getCanvas().style.cursor = "pointer";
            });
            map.on("mouseleave", "bike-count", () => {
                map.getCanvas().style.cursor = "";
            });

            for (let i = 0; i < bikes_day1.features.length; i++) {
                //console.log(i,bikes_day1.features[i].properties.Name)
                stationNames.push(bikes_day1.features[i].properties.Name);
            }

            stationIndex = stationNames.indexOf(station);
            bikecount = bikes_day1.features[stationIndex].properties[daytime];
            */
        });
    });
</script>

<div id="map" class="map" />

<style>
    .map {
        height: 100vh;
        width: 100vw;
        top: 0;
        left: 0%;
        position: absolute;
        overflow: hidden;
    }
</style>