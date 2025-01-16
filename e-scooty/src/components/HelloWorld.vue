<template>
  <div>
    <ul class="nav nav-pills padding-left" style="top: 0">
      <li class="w-75 mt-2">
          <h1>E Scooty</h1>
      </li>
      <li class="display-right mt-2 pe-5 w-25">
        <div>
          <a class="d-inline-flex link-body-emphasis text-decoration-none" id="open-settings">
            <p>settings</p>
          </a>
        </div>
      </li>
    </ul>
  </div>


  <div class = "format center">
    <l-map ref="map" v-model:zoom="zoom" :center="[53.75153044444902, 9.664619800111053]">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
      ></l-tile-layer>
      <l-geo-json :geojson="escooterStellplatz" :options="geojsonOptions" />
      <l-geo-json :geojson="escooterParkverbot" :options="geojsonOptions" />
    </l-map>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LGeoJson } from "@vue-leaflet/vue-leaflet";
import Escooter_Stellplatz from "../assets/Escooter_Stellplatz.geojson";
import EScooter_Parkverbot_EL from "../assets/EScooter_Parkverbot_EL.geojson";
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
      escooterStellplatz: Escooter_Stellplatz,
      escooterParkverbot: EScooter_Parkverbot_EL,
      geojsonOptions: {
        // Optionen für die GeoJSON-Darstellung
      }
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

.center {
  margin: auto;
  width: 50%;
  padding: 10px;
}

.format {
  width: 50vw;
  height: 50vh;
}

.display-right {
  display: right;
}
</style>