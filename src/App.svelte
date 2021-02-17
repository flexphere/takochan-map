<script>
import stationsData from './data'
let map
let activeMarker = null
const markers = []

async function initMap(container) {
    map = L.map(container, {preferCanvas: true, zoomControl: false }).setView([ 35.676192, 139.650311 ], 6);
    L.tileLayer('https://{s}.basemaps.cartocdn.com/rastertiles/voyager/{z}/{x}/{y}{r}.png',
	    {
	    	attribution: `&copy;<a href="https://www.openstreetmap.org/copyright" target="_blank">OpenStreetMap</a>,&copy;<a href="https://carto.com/attributions" target="_blank">CARTO</a>`,
	    	subdomains: 'abcd',
        	maxZoom: 20,
        	minZoom: 5,
	    }
    ).addTo(map);

	const markerEl = `<div><svg width="17" height="23" viewBox="0 0 371 506" fill="none" xmlns="http://www.w3.org/2000/svg"><path d="M185.427 0C83.182 0 0 83.182 0 185.426C0 312.314 165.939 498.593 173.004 506.461C179.64 513.852 191.226 513.839 197.85 506.461C204.915 498.593 370.854 312.314 370.854 185.426C370.852 83.182 287.671 0 185.427 0ZM185.427 278.719C133.985 278.719 92.135 236.868 92.135 185.426C92.135 133.984 133.986 92.134 185.427 92.134C236.868 92.134 278.718 133.985 278.718 185.427C278.718 236.869 236.868 278.719 185.427 278.719Z"/></svg></div>`;
	const markerIcon = L.divIcon({className: 'marker-icon', html: markerEl, iconAnchor:[12, 28]});
	const stations = loadStations()
	for (const station of stations) {
		const marker = L.marker([station.latitude, station.longitude], {icon: markerIcon}).addTo(map)
		marker.station = station
		marker.on('click', e => {
			activeMarker = e.sourceTarget;
		});
		setMarkerState(marker)
		markers.push(marker)
	}

    return {
		destroy: () => {
			map.remove();
			map = null;
		}
    };
}

function resizeMap() {
	if (map) { map.invalidateSize(); }
}

function setMarkerState(marker) {
	if (marker.station.x !== '' && marker.station.y !== '') {
		marker._icon.classList.add("marker-icon-done")
	} else {
		marker._icon.classList.remove("marker-icon-done")
	}
}

function updateXY() {
	setMarkerState(activeMarker)
	saveStations()
}

function saveStations() {
	const stations = markers.map(m => m.station)
	localStorage.setItem('stations', JSON.stringify(stations))
}

function loadStations() {
	let stations = localStorage.getItem('stations');
	if (stations) {
		return JSON.parse(stations)
	}
	return stationsData
}
</script>
<svelte:window on:resize={resizeMap} />

<div id="Map" use:initMap></div>

{#if activeMarker !== null }
<div id="window">
	<div class="station_cd">{activeMarker.station.station_cd} - {activeMarker.station.station_name}</div>
	<div class="input_field"><input type="number" on:change={updateXY} bind:value={activeMarker.station.x}></div>
	<div class="input_field"><input type="number" on:change={updateXY} bind:value={activeMarker.station.y}></div>
</div>
{/if}

<style>
#Map {
	width: 100vw;
	height: 100vh;
}
#Map :global(.marker-icon) {
	cursor: ponter;
}

#Map :global(.marker-icon > div > svg) {
	fill: #ed2f2f;
	
}
#Map :global(.marker-icon.marker-icon-done > div > svg) {
	fill: #2F80ED;
}

#window {
	position: absolute;
	top: 20px;
	right: 20px;
	
	display: flex;
	flex-direction: column;

	padding: 20px;
	border-radius: 5px;
	background: #fff;
	z-index: 999;
}

#window > * {
	margin: 4px 0;
	font-weight: bold;
}

#window .station_cd {
	text-align: center;
}

#window input {
	margin: 0;
	padding-left: 50px;
	font-weight: bold;
}

#window .input_field {
	position: relative;
}

#window .input_field:before {
	position: absolute;
	top: 2px;
	left: 2px;
	display: flex;
	justify-content: center;
	align-items: center;
	width: 40px;
	height: calc(100% - 4px);
	background: #2F80ED;
	color: #fff;
}
#window .input_field:nth-child(2):before {
	content: 'X';
}
#window .input_field:nth-child(3):before {
	content: 'Y';
}
</style>