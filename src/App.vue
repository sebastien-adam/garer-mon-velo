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
  console.log(url);
  const res = await fetch(url);
  const data = await res.json();
  arceaux.value = data.results;
}

async function locateMe() {
  isSearching.value = true
  navigator.geolocation.getCurrentPosition(setPosition, ()=>{isSearching.value=false}, {maximumAge: 50000, timeout: 20000, enableHighAccuracy: true});
}

async function setPosition(pos) {
  const crd = pos.coords;
  currentPosition.value = [crd.latitude, crd.longitude]
  coords.value = [crd.latitude, crd.longitude]
  zoom.value = 18;
  isSearching.value = false;
  await getData()
  console.log(arceaux.value);
}

function getLatLng(geo_point_2d) {
  return [geo_point_2d.lat, geo_point_2d.lon];
}

</script>

<template>
  <h1>Où garer mon vélo</h1>
  <!-- <button @click="getData">Touver des arceaux</button> -->
  <button @click="locateMe" :disabled="isSearching">Me localiser</button>
  <div v-if="currentPosition">{{ currentPosition }}</div>
  <div v-else style="color:red">Position actuelle inconnue</div>
  <div class="distance">
    <label>Distance: {{area}} m</label>
    <input type="range" v-model="area" min="5" max="800">
  </div>
  <div style="height: 600px; width: 100%" >
    <l-map ref="map" :use-global-leaflet="false" v-model:zoom="zoom" :center="coords">
      <l-tile-layer url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png" layer-type="base"
        name="OpenStreetMap"></l-tile-layer>
        <l-circle v-if="currentPosition" :lat-lng="currentPosition" :radius="area" color="green"/>  
      <l-circle-marker v-if="currentPosition" :lat-lng="currentPosition" :radius=16 color="red" />
      <l-marker v-if="currentPosition" v-for="(arceau, index) in arceaux" :key="index"
        :lat-lng="getLatLng(arceau.geo_point_2d)" />
    </l-map>
  </div>
</template>

<style scoped>
button {
  padding: 12px 32px;
  font-size: 16px;
  border-radius: 8px;
  margin: 16px;
}

.distance {
  margin: 1rem;
  display: flex;
  justify-content: center;
  flex-direction: column;
}

.map-container {
  width: 600px;
  height: 600px;
}

.red {
  background-color: blueviolet;
}
</style>
