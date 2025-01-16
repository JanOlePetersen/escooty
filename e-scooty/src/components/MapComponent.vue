<script setup>
  var filterOneOptions = ["Straßennetz", 2024, 2025];
  var filterTwoOptions = [2023, 2024, 2025];
</script>


<template>
  <div class="nav justify-content-center mb-4">
    <h1>E Scooty</h1>
  </div>

  <div>
    <div class="mb-5"> <!-- Filter active / completed project partizipation -->
      <button @click="dropdownInformation = !dropdownInformation"  class="btn btn-outline-dropdown align-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow w-25" data-bs-toggle="dropdown" aria-expanded="false" id="filter-active">Ebenen</button>
      <ul v-show="dropdownInformation" @:mouseleave="dropdownInformation = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 shadow w-25 z-2 ml-filter-one" id="traffic-options">
        <li v-for="(option, index) in filterOneOptions" :key="index"><a class="dropdown-item rounded-2 active" @mousedown="noneLoaded = false" :id="options" v-on:mouseup="dropdownInformation = !dropdownInformation;" aria-current="page">{{ option }}</a></li>
      </ul>

      <button @click="dropdownActuality = !dropdownActuality"  class="btn btn-outline-dropdown align-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow w-25" data-bs-toggle="dropdown" aria-expanded="false" id="filter-active" style="margin-left: 10vw">Anderer Filter</button>
      <ul v-show="dropdownActuality" @:mouseleave="dropdownActuality = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 shadow w-25 z-2 ml-filter-two" id="active-options">
        <li v-for="(option, index) in filterTwoOptions" :key="index"><a class="dropdown-item rounded-2 active" @mousedown="noneLoaded = false" :id="options" v-on:mouseup="dropdownInformation = !dropdownInformation;" aria-current="page">{{ option }}</a></li>
      </ul>
    </div>
  </div>


  <div class = "format center">
    <l-map ref="map" v-model:zoom="zoom" :center="[53.75153044444902, 9.664619800111053]">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
      ></l-tile-layer>
      <!-- <l-geo-json :geojson="elmshornStrassenbeleuchtung" :options="geojsonOptionsStrassenbeleuchtung" /> -->
      <l-geo-json :geojson="elmshornStraßennetz" :options="geojsonOptionsStraßennetz" />
      <l-geo-json :geojson="elmshornVelorouten" :options="geojsonOptionsVelorouten" />
      <l-geo-json :geojson="escooterParkverbot" :options="geojsonOptionsParkverbot" />
      <l-geo-json :geojson="escooterStellplatz" :options="geojsonOptionsStellplatz" />
      <l-geo-json :geojson="elmshornHaltestellen" :options="geojsonOptionsHaltestellen" />
      <l-geo-json :geojson="elmshornBebauungsFlaeche" :options="geojsonOptionsBebauungsFlaeche" />
      

    </l-map>
  </div>
</template>

<script>
import '../assets/main.css';
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LGeoJson } from "@vue-leaflet/vue-leaflet";
// import Strassenbeleuchtung from "../assets/Strassenbeleuchtung.geojson";
import Straßennetz from "../assets/Straßennetz.geojson";
import Velorouten from "../assets/Velorouten.geojson";
import Escooter_Stellplatz from "../assets/Escooter_Stellplatz.geojson";
import EScooter_Parkverbot_EL from "../assets/EScooter_Parkverbot_EL.geojson";
import Haltestellen from "../assets/Haltestellen.geojson";
import BebauungsFlaeche from "../assets/BebauungsFlaeche.geojson";
import L from "leaflet";

export default {
  components: {
    LMap,
    LTileLayer,
    LGeoJson
  },
  data() {
    return {
      dropdownInformation: false,
      dropdownActuality: false,
      zoom: 13,
      escooterStellplatz: Escooter_Stellplatz,
      escooterParkverbot: EScooter_Parkverbot_EL,
      elmshornHaltestellen: Haltestellen,
      // elmshornStrassenbeleuchtung: Strassenbeleuchtung,
      elmshornStraßennetz: Straßennetz,
      elmshornVelorouten: Velorouten,
      elmshornBebauungsFlaeche: BebauungsFlaeche,
      geojsonOptionsStellplatz: {
      // Färbt die jeweilige genannte EScooterID ein
        onEachFeature: (feature, layer) => {
          if (feature.properties && feature.properties.Escoote_ID === 1) {
            layer.setStyle({
              color: 'red'
            });
          }
        }
      },
      geojsonOptionsParkverbot: {
        //"color": "#ffffff",
      },
      geojsonOptionsHaltestellen: {
        pointToLayer: (feature, latlng) => {
          return L.circleMarker(latlng, {
            radius: 5,
            fillColor: "#ff7800",
            color: "#000",
            weight: 1,
            opacity: 1,
            fillOpacity: 0.8
          });
        }
      },
      // geojsonOptionsStrassenbeleuchtung: {
      //   "color": "yellow"
      // },
      geojsonOptionsStraßennetz: {
        "color": "green"
      },
      geojsonOptionsVelorouten: {
        "color": "blue"
      },
      geojsonOptionsBebauungsFlaeche: {
        onEachFeature: (feature, layer) => {
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 1000) {
            layer.setStyle({
              color: 'white'
            });
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 2000) {
            layer.setStyle({
              color: 'yellow'
            });
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 3000) {
            layer.setStyle({
              color: 'red'
            });
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 4000) {
            layer.setStyle({
              color: 'red'
            });
          }
        }
      },
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

.ml-filter-one {
  margin-left: 20vw !important;
}

.ml-filter-two {
  margin-left: 55vw !important;
}
</style>