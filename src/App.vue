<script setup>

import { ref } from 'vue';
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LMarker, LCircleMarker, LCircle } from "@vue-leaflet/vue-leaflet";

// fake in firefox / geo. : data:application/json,{"location": {"lat": 48.58114, "lng": 7.74853}, "accuracy": 2700.0}

const zoom = ref(12);
const arceaux = ref([]);
const coords = ref([48.58114, 7.74853]);
const currentPosition = ref(null);
const area = ref(100);
const isSearching = ref(false);

const API_URL = `https://data.strasbourg.eu/api/explore/v2.1/catalog/datasets/stationnement-velo/records?select=`

async function getData() {
  const ARCEAU_REQUEST = `geo_point_2d&where=within_distance(geo_point_2d%2C%20geom%27POINT(${currentPosition.value[1]}%20${currentPosition.value[0]})%27%2C%20${area.value}m)%20&limit=40`
  const url = `${API_URL}${ARCEAU_REQUEST}`
  //console.log(url);
  const res = await fetch(url);
  const data = await res.json();
  arceaux.value = data.results;
}

async function locateMe() {
  isSearching.value = true
  navigator.geolocation.getCurrentPosition(setPosition, () => { isSearching.value = false }, { maximumAge: 50000, timeout: 20000, enableHighAccuracy: true });
}

async function setPosition(pos) {
  const crd = pos.coords;
  currentPosition.value = [crd.latitude, crd.longitude]
  coords.value = [crd.latitude, crd.longitude]
  getZoom();
  isSearching.value = false;
  await getData()
}

function getLatLng(geo_point_2d) {
  return [geo_point_2d.lat, geo_point_2d.lon];
}

function getZoom() {
  let zoomValue
  if (area.value <= 100) zoomValue = 18;
  else if (area.value < 200) zoomValue = 17;
  else if (area.value < 400) zoomValue = 16;
  else zoomValue = 15;
  zoom.value = zoomValue;
}

</script>

<template>
  <header>
    <h1>Où garer mon vélo</h1>
    <h2>A Strasbourg et environs</h2>
  </header>
  <div class="flex">
    <button @click="locateMe" :disabled="isSearching" class="button"><svg xmlns="http://www.w3.org/2000/svg" width="24"
        height="24" viewBox="0 0 24 24">
        <g id="feTarget0" fill="none" fill-rule="evenodd" stroke="none" stroke-width="1">
          <g id="feTarget1" fill="currentColor" fill-rule="nonzero">
            <path id="feTarget2"
              d="M19.938 13A8.004 8.004 0 0 1 13 19.938V22h-2v-2.062A8.004 8.004 0 0 1 4.062 13H2v-2h2.062A8.004 8.004 0 0 1 11 4.062V2h2v2.062A8.004 8.004 0 0 1 19.938 11H22v2h-2.062ZM12 18a6 6 0 1 0 0-12a6 6 0 0 0 0 12Zm0-3a3 3 0 1 0 0-6a3 3 0 0 0 0 6Z" />
          </g>
        </g>
      </svg>{{ currentPosition ? "Rafraichir" : "Me localiser" }}</button>
  </div>
  <div v-if="!currentPosition | !arceaux.length" :class="[isSearching ? 'green' : 'red']">{{ isSearching ? 'Localisation en cours' :
    !arceaux.length ? "Aucun arceau trouvé" : 'Position actuelle inconnue' }}</div>
  <div class="flex distance">
    <label>Distance: {{ area }} m</label>
    <input type="range" v-model="area" min="10" max="800" @change="getZoom">
  </div>
  <div class="map">
    <l-map ref="map" :use-global-leaflet="false" v-model:zoom="zoom" :center="coords">
      <l-tile-layer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" layer-type="base"
        name="OpenStreetMap"></l-tile-layer>
      <l-circle v-if="currentPosition" :lat-lng="currentPosition" :radius="parseInt(area)" color="green" />
      <l-circle-marker v-if="currentPosition" :lat-lng="currentPosition" :radius=16 color="red" />
      <l-marker v-if="currentPosition" v-for="(arceau, index) in arceaux" :key="index"
        :lat-lng="getLatLng(arceau.geo_point_2d)" />
    </l-map>
  </div>
  <footer class="flex">
    <p>Site réalisé par <a href="https://sebastien-adam.github.io/" target="_blank" rel="noopener noreferrer">Sébastien
        ADAM</a>
    </p>
    <p>Les données : <a href="https://data.strasbourg.eu/pages/accueil/" target="_blank" rel="noopener noreferrer">Open
        Data Strasbourg</a></p>
  </footer>
</template>

<style scoped>
button {
  padding: 12px 32px;
  font-size: 16px;
  border-radius: 8px;
  margin: 16px;
}

.flex {
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.distance {
  margin: 1rem auto;
  width: 80%;
}

.map {
  height: 60vh;
  padding: 1rem 1rem;
}

header {
  border-bottom: 1px solid gray;
  margin-bottom: 1rem;
}

footer {
  padding-top: 1rem;
  border-top: 1px solid gray;
}

p {
  margin: 0
}

.red {
  color: red;
}

.green {
  @keyframes loading {
    0% {
      background-position: 0% 50%;
    }

    100% {
      background-position: 100% 50%;
    }
  }

  color: white;
  background: linear-gradient(90deg, transparent, green, transparent);
  background-size: 100% 100%;
  animation: loading 2s linear infinite;

}
</style>