<script lang="ts">
	import maplibregl from 'maplibre-gl' // or "const maplibregl = require('maplibre-gl');"
	import 'maplibre-gl/dist/maplibre-gl.css'
	import MapboxGeocoder from '@mapbox/mapbox-gl-geocoder'
	import '@mapbox/mapbox-gl-geocoder/dist/mapbox-gl-geocoder.css'
	import { onMount } from 'svelte'
    interface IconObject {
        icon?: string;
        type?: string;
        src?: string;
        color?: string;
        size?: "large" | "small" | "medium";
        width?: string;
        height?: string;
        onclick?: any;
        pop: any;
    }
    interface MapOptions {
        center?: maplibregl.LngLatLike;
        zoom?: number;
        style?: string;
    }
	let map: maplibregl.Map
    export let controller: any = {}
    export let mapOptions: MapOptions = {}
    export let onclick: any = () => {}
    let mapStyle = 'streets-v2';

    const mapStyles = ['streets-v2','openstreetmap', 'winter', 'hybrid', 'satellite', 
    'basic', 'bright', 'positron', 'topo', 'voyager', 'basic-v2', 'toner-v2', 'dataviz']
    let mapStyleIndex = 0;

	onMount(() => {
        if (!import.meta.env.VITE_MAPBOX_KEY) {
            console.error('Environment variable VITE_MAPBOX_KEY not set: aborting...');
            return;
        }
        if (!import.meta.env.VITE_MAPTILER_KEY) {
            console.error('Environment variable VITE_MAPTILER_KEY not set: aborting...');
            return;
        }
		const el: any = document.getElementById('map')
		map = new maplibregl.Map({
			container: el, //'map',
            style: mapOptions.style || `https://api.maptiler.com/maps/${mapStyles[mapStyleIndex]}/style.json?key=${import.meta.env.VITE_MAPTILER_KEY}`,
			center: mapOptions.center || [0,0], // starting position [lng, lat]
			zoom: mapOptions.zoom || 14, // starting zoom
		})
        if (onclick) {
            map.on('click', onclick)
        }
        const loadMapImage = (id: string, img?: string) => {
            map.loadImage(
                    `/assets/map_${img || id}.png`,
                    function (error: any, image: any) {
                        if (error) throw error
                        map.addImage(id, image)
                    }
                )

        }
        loadMapImage('hostel', 'hostel')
        loadMapImage('tattoo', 'tattoo')
        loadMapImage('mall', 'mall')
        loadMapImage('beauty', 'generic')
        loadMapImage('furniture', 'generic')
        loadMapImage('books', 'generic')
        loadMapImage('nightclub', 'generic')
        loadMapImage('shoes', 'generic')
        loadMapImage('computer', 'generic')
        loadMapImage('supermarket', 'generic')
        loadMapImage('clothes', 'generic')
        loadMapImage('hardware', 'generic')
        loadMapImage('toys', 'generic')
        loadMapImage('sports', 'generic')
        loadMapImage('stationery', 'generic')
        loadMapImage('hifi', 'generic')
        loadMapImage('veterinary', 'generic')
        loadMapImage('butcher', 'generic')
        loadMapImage('bahai', 'generic')
        loadMapImage('food_court', 'generic')
        loadMapImage(' ', 'generic')

        controller.map = map;
		map.on('load', function (m) {
			map.resize()
		})
        map.on('styleimagemissing', function (e: any) {
            console.log('styleimagemissing', e)
        })

		const geocoderControl: MapboxGeocoder = new MapboxGeocoder({
			accessToken: import.meta.env.VITE_MAPBOX_KEY,
			mapboxgl: maplibregl as any,
		})
		map.addControl(geocoderControl as any, 'top-left')

		const geolocateControl = new maplibregl.GeolocateControl({
			positionOptions: {
				enableHighAccuracy: true,
			},
			trackUserLocation: true,
		})

		geolocateControl.on('geolocate', function (e: any) {
			setTimeout(() => {
				// const b = map.getBounds()
				// loadMarkerSets(b.getCenter(), b.getNorthEast().distanceTo(b.getCenter()))
			}, 500)
		})

		map.addControl(geolocateControl)

	})

    // const popup = new maplibregl.Popup()
    //     .setLngLat([metadata.Longitude, metadata.Latitude])
    //     .setDOMContent(el)
    //   popup.addTo(map)
    //   popupOpen = true
    //   popup.on('close', (e: any) => {
    //     popupOpen = false
    //   })

	controller.addMarker = (lngLat: maplibregl.LngLatLike, 
                            iconObject: IconObject, 
                            options: maplibregl.MarkerOptions = {}) => {
        // console.log('addMarker', lngLat, options, iconObject)

        if (iconObject.type === 'img') {
            const img = document.createElement('img');
            img.setAttribute('src', iconObject.src || '');
            if (iconObject.width) img.setAttribute('width', iconObject.width)
            if (iconObject.height) img.setAttribute('height', iconObject.height)
            if (iconObject.pop) {
                img.onclick = () => {
                    const popup = new maplibregl.Popup({closeButton: true})
                    .setLngLat(lngLat)
                    .setDOMContent(iconObject.pop)
                    popup.addTo(map)
                }
            } else {
                if (iconObject.onclick) img.onclick = iconObject.onclick;
            }
            if (iconObject.pop || iconObject.onclick) img.style.cursor = 'pointer';
            options.element = img;
        } else {
            const icon = document.createElement('ion-icon');
            icon.setAttribute('size', iconObject.size || 'large');
            icon.setAttribute('color', iconObject.color || 'dark');
            //icon.setAttribute('style','color: darkgreen;');
            if (iconObject.icon) {
                icon.setAttribute('icon', iconObject.icon);
            }
            if (iconObject.src) {
                icon.setAttribute('src', iconObject.src);
            }
            options.element = icon;
        }
        const marker = new maplibregl.Marker(options)
        .setLngLat(lngLat)
        .addTo(map);
        return marker;
	}
    controller.setStyle = (style?: string) => {
        map.setStyle(`https://api.maptiler.com/maps/${style || mapStyles[mapStyleIndex]}/style.json?key=${import.meta.env.VITE_MAPTILER_KEY}`);
    }
    
</script>

<div id="chooser">
</div>
<div id="map" style="height:100%; width: 100%;">
    <ion-select interface="popover" value={mapStyleIndex} class="mapStyleSelector"
        on:ionChange={(e)=>{
            mapStyleIndex = e.target.value;
            controller.setStyle();
        }}>
        {#each mapStyles as style, i}
            <ion-select-option value={i}>{style}</ion-select-option>
        {/each}
    </ion-select>    
</div>

<style>
    .mapStyleSelector {
        position: absolute; 
        bottom: 10px; 
        left: 10px; 
        width: auto; 
        height: 20px;
        z-index: 1000; 
        color: var(--ion-color-light-contrast);
        background-color: var(--ion-color-light);
        border-radius: 5px; 
    }
    .map {
        cursor: pointer;
    }
	/* .map-loader {
		margin-left: 40%;
		margin-top: 10%;
	} */
</style>
