<script>
    import { onMount } from "svelte";
    import mapboxgl from "mapbox-gl";
    import "mapbox-gl/dist/mapbox-gl.css";
    import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
    import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";

    mapboxgl.accessToken =
        "pk.eyJ1IjoiZGFubGVpaWkiLCJhIjoiY2xqeGVxcGdyMHdrczNvbjE0MnJ6aGs1dCJ9.h_OG2gqN6HJ2JXpOZGxe4A";

    let map;
    let start = [9.955197405862306, 48.423525011347266]; // Define start location

    async function getRoute(end) {
        const query = await fetch(
            `https://api.mapbox.com/directions/v5/mapbox/cycling/${start[0]},${start[1]};${end[0]},${end[1]}?steps=true&geometries=geojson&access_token=${mapboxgl.accessToken}`,
            { method: "GET" }
        );
        const json = await query.json();
        const data = json.routes[0];
        const route = data.geometry.coordinates;
        const geojson = {
            type: "Feature",
            properties: {},
            geometry: {
                type: "LineString",
                coordinates: route,
            },
        };

        if (map.getSource("route")) {
            map.getSource("route").setData(geojson);
        } else {
            map.addSource("route", {
                type: "geojson",
                data: geojson,
            });

            map.addLayer({
                id: "route",
                type: "line",
                source: "route",
                layout: {
                    "line-join": "round",
                    "line-cap": "round",
                },
                paint: {
                    "line-color": "#70A9FF",
                    "line-width": 4,
                    "line-opacity": 0.75,
                },
            });
        }
    }

    onMount(() => {
        map = new mapboxgl.Map({
            container: "map-container",
            style: "mapbox://styles/mapbox/streets-v12",
            center: [9.955197405862306, 48.423525011347266],
            zoom: 14,
        });

        map.addControl(
            new mapboxgl.GeolocateControl({
                positionOptions: {
                    enableHighAccuracy: true,
                },
                trackUserLocation: true,
                showUserHeading: true,
                geolocateControlOptions: {
                    positionOptions: {
                        enableHighAccuracy: true,
                    },
                    trackUserLocation: true,
                    showUserLocation: true,
                },
                className: "custom-geolocate-control",
            })
        );

        const geocoder = new MapboxGeocoder({
            accessToken: mapboxgl.accessToken,
            mapboxgl: mapboxgl,
        });

        map.addControl(geocoder, "top-left");

        map.addControl(new mapboxgl.NavigationControl());


        map.on("load", () => {
            getRoute(start);

            map.addLayer({
                id: "point",
                type: "circle",
                source: {
                    type: "geojson",
                    data: {
                        type: "FeatureCollection",
                        features: [
                            {
                                type: "Feature",
                                properties: {},
                                geometry: {
                                    type: "Point",
                                    coordinates: start,
                                },
                            },
                        ],
                    },
                },
                paint: {
                    "circle-radius": 10,
                    "circle-color": "#000000",
                },
            });
        });
        map.on("click", (event) => {
            const coords = Object.keys(event.lngLat).map(
                (key) => event.lngLat[key]
            );
            const end = {
                type: "FeatureCollection",
                features: [
                    {
                        type: "Feature",
                        properties: {},
                        geometry: {
                            type: "Point",
                            coordinates: coords,
                        },
                    },
                ],
            };
            if (map.getLayer("end")) {
                map.getSource("end").setData(end);
            } else {
                map.addLayer({
                    id: "end",
                    type: "circle",
                    source: {
                        type: "geojson",
                        data: {
                            type: "FeatureCollection",
                            features: [
                                {
                                    type: "Feature",
                                    properties: {},
                                    geometry: {
                                        type: "Point",
                                        coordinates: coords,
                                    },
                                },
                            ],
                        },
                    },
                    paint: {
                        "circle-radius": 10,
                        "circle-color": "#70A9FF",
                    },
                });
            }
            getRoute(coords);
        });
    });
</script>

<div class="map-wrap">
    <div id="map-container" />
</div>

<style>
    #map-container {
        position: absolute;
        width: 100%;
        height: 100%;
    }
    .map-wrap {
        position: relative;
        width: 100%;
        height: calc(100vh - 77px);
    }
</style>
