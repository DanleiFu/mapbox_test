<script>
    import { onMount } from "svelte";
    import mapboxgl from "mapbox-gl";
    import "mapbox-gl/dist/mapbox-gl.css";
    import MapboxGeocoder from "@mapbox/mapbox-gl-geocoder";
    import "@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css";
    const baseUrl = "https://api.mapbox.com/directions/v5/mapbox/driving";



    mapboxgl.accessToken =
        "pk.eyJ1IjoiZGFubGVpaWkiLCJhIjoiY2xqeDJtaDBqMDB1NDNkcXozcTBiMHNkZiJ9.sZrKyLHpvsyzW3_UkcQUAg";

    onMount(() => {
        const map = new mapboxgl.Map({
            container: "map-container",
            style: "mapbox://styles/mapbox/streets-v12",
            center: [9.955197405862306, 48.423525011347266],
            zoom: 14,
        });

        map.addControl(
            new mapboxgl.AttributionControl({
                customAttribution: "Map for Ulm Herum",
            })
        );

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
    });
</script>

<div class="map-wrap">
    <div id="map-container"></div>
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
