<template>
  <div style="height:600px; width:800px">
    <l-map ref="map" v-model:zoom="zoom" :center="[53.75153044444902, 9.664619800111053]">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
      ></l-tile-layer>
      <l-geo-json :geojson="geojson" :options="geojsonOptions" />
    </l-map>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LGeoJson } from "@vue-leaflet/vue-leaflet";
import Escooter_Stellplatz from "../assets/Escooter_Stellplatz.geojson";
// import L from "leaflet";
// import { onMounted } from "vue";

export default {
  components: {
    LMap,
    LTileLayer,
    LGeoJson
  },
  data() {
    return {
      zoom: 13,
      geojson: Escooter_Stellplatz,
    };
  },
};
</script>

<style>
html, body {
  margin: 0;
  padding: 0;
}

main {
  height: 100vh;
  width: 100vw;
}
</style>