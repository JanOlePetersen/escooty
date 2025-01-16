<script setup>
  var filterOneOptions = ["Straßennetz", "Veloruten", "Stellplätze", "Bushaltestellen", "Parkverbot", "Bebauungsfläche", "Straßenbeleuchtung", "Denkmäler"];
  var filterTwoOptions = [2023, 2024, 2025];
</script>


<template>
  <div class="nav justify-content-center">
    <h1>E-Scooty</h1>
  </div>

  <div>
    <div> <!-- Filter active / completed project partizipation -->
      <button @click="dropdownInformation = !dropdownInformation" class="btn btn-outline-dropdown align-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow w-25" data-bs-toggle="dropdown" aria-expanded="false" id="filter-layers">Ebenen</button>
      <ul v-show="dropdownInformation" @mouseleave="dropdownInformation = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 shadow w-25 z-2 ml-filter-one" id="layer-options">
        <li v-for="(option, index) in filterOneOptions" :key="index">
          <a 
            class="dropdown-item rounded-2 active layer-option" 
            @mousedown="noneLoaded = false; handleFilter('layer-' + index)" 
            :id="'layer-' + index" 
            @mouseup="filterLayers(index)" 
            aria-current="page">
              <img class="s-img me-2" v-if="layerVisibility[index].visible" :src="CheckBox" alt="Checkbox" />
              <img class="s-img me-2" v-else :src="CheckBoxOff" alt="Checkbox" />
              {{ option }}
          </a>
        </li>
      </ul>

      <button @click="dropdownActuality = !dropdownActuality" class="btn btn-outline-dropdown align-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow w-25" data-bs-toggle="dropdown" aria-expanded="false" id="filter-active" style="margin-left: 10vw">Anderer Filter</button>
      <ul v-show="dropdownActuality" @mouseleave="dropdownActuality = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 shadow w-25 z-2 ml-filter-two" id="active-options">
        <li v-for="(option, index) in filterTwoOptions" :key="index">
          <a 
            class="dropdown-item rounded-2 active active-option" 
            @mousedown="noneLoaded = false; handleFilter('active-' + index)" 
            :id="'active-' + index" 
            @mouseup="filterLayers2()" 
            aria-current="page">
            {{ option }}
          </a>
        </li>
      </ul>
    </div>
  </div>


  <div class = "format center">
    <l-map ref="map" v-model:zoom="zoom" :center="[53.75153044444902, 9.664619800111053]">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
        minZoom="14"
      maxZoom="20"
      ></l-tile-layer>
      <l-geo-json v-if="layerVisibility[6].visible" :geojson="elmshornStrassenbeleuchtung" :options="geojsonOptionsStrassenbeleuchtung" />
      <l-geo-json v-if="layerVisibility[0].visible" :geojson="elmshornStraßennetz" :options="geojsonOptionsStraßennetz" />
      <l-geo-json v-if="layerVisibility[1].visible" :geojson="elmshornVelorouten" :options="geojsonOptionsVelorouten" />
      <l-geo-json v-if="layerVisibility[4].visible" :geojson="escooterParkverbot" :options="geojsonOptionsParkverbot" />
      <l-geo-json v-if="layerVisibility[2].visible" :geojson="escooterStellplatz" :options="geojsonOptionsStellplatz" />
      <l-geo-json v-if="layerVisibility[3].visible" :geojson="elmshornHaltestellen" :options="geojsonOptionsHaltestellen" />
      <l-geo-json v-if="layerVisibility[5].visible" :geojson="elmshornBebauungsFlaeche" :options="geojsonOptionsBebauungsFlaeche" />
      <l-geo-json v-if="layerVisibility[7].visible" :geojson="elmDenkmal" :options="geojsonOptionsDenkmal" />

    </l-map>
  </div>
</template>

<script>
import '../assets/main.css';
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LGeoJson } from "@vue-leaflet/vue-leaflet";
import Strassenbeleuchtung from "../assets/Strassenbeleuchtung.geojson";
import Straßennetz from "../assets/Straßennetz.geojson";
import Velorouten from "../assets/Velorouten.geojson";
import Escooter_Stellplatz from "../assets/Escooter_Stellplatz.geojson";
import EScooter_Parkverbot_EL from "../assets/EScooter_Parkverbot_EL.geojson";
import Haltestellen from "../assets/Haltestellen.geojson";
import BebauungsFlaeche from "../assets/BebauungsFlaeche.geojson";
import Denkmal from "../assets/Denkmal.geojson";
import L from "leaflet";

import CheckBox from "../assets/square-check-regular.svg";
import CheckBoxOff from "../assets/square-regular.svg";

import busStopIconUrl from "../assets/bus-solid.svg"; 
const busStopIcon = L.icon({
  iconUrl: busStopIconUrl,
  iconSize: [15, 15], // Größe des Icons
  // iconAnchor: [12, 25], // Punkt des Icons, der auf den Marker zeigt
  // popupAnchor: [0, -25] // Punkt, von dem aus das Popup relativ zum Icon geöffnet wird
});
import denkmalIconUrl from "../assets/landmark-solid.svg"; 
const denkmalIcon = L.icon({
  iconUrl: denkmalIconUrl,
  iconSize: [15, 15], // Größe des Icons
  // iconAnchor: [12, 25], // Punkt des Icons, der auf den Marker zeigt
  // popupAnchor: [0, -25] // Punkt, von dem aus das Popup relativ zum Icon geöffnet wird
});

export default {
  components: {
    LMap,
    LTileLayer,
    LGeoJson,
  },
  data() {
    return {
      dropdownInformation: false,
      dropdownActuality: false,
      layerVisibility: [
        { id: 0, name: "Straßennetz", visible: true },
        { id: 1, name: "Veloruten", visible: true },
        { id: 2, name: "Stellplätze", visible: true },
        { id: 3, name: "Straßenbeleuchtung", visible: true },
        { id: 4, name: "Parkverbot", visible: true },
        { id: 5, name: "Bebauungsfläche", visible: true },
        { id: 6, name: "Straßenbeleuchtung", visible: true },
        { id: 7, name: "Denkmäler", visible: true }
      ],
      zoom: 13,
      escooterStellplatz: Escooter_Stellplatz,
      escooterParkverbot: EScooter_Parkverbot_EL,
      elmshornHaltestellen: Haltestellen,
      elmshornStrassenbeleuchtung: Strassenbeleuchtung,
      elmshornStraßennetz: Straßennetz,
      elmshornVelorouten: Velorouten,
      elmshornBebauungsFlaeche: BebauungsFlaeche,
      elmDenkmal: Denkmal,
      geojsonOptionsStellplatz: {
      // Färbt die jeweilige genannte EScooterID ein
      color: 'blue',
      onEachFeature: (feature, layer) => {
          // if (feature.properties && feature.properties.Escoote_ID === 1) {
          //   layer.setStyle({
          //     color: 'red'
          //   });
          // }
          layer.on({
            click: () => {
              layer.bindPopup(`Stellplatz ID: ${feature.properties.Escoote_ID}`).openPopup();
              layer.setStyle({
              color: 'red'
            });
            },
            mouseover: () => {
              layer.setStyle({
                weight: 7
              })
            },
            mouseout: () => {
              layer.setStyle({
                weight: 2
              })
            },
            popupclose: () => {
              layer.setStyle({
                color: 'blue'
              })
            }
          })
        }
      },
      geojsonOptionsParkverbot: {
        //"color": "#ffffff",
      },
      geojsonOptionsHaltestellen: {
        pointToLayer: (feature, latlng) => {
          return L.marker(latlng, { icon: busStopIcon });
        }
      },
      geojsonOptionsStrassenbeleuchtung: {
        pointToLayer: (feature, latlng) => {
          return L.circleMarker(latlng, {
            radius: 5,
            fillColor: "yellow",
            color: "#000",
            weight: 1,
            opacity: 0.01,
            fillOpacity: 0.8
          });
        }
      },
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
      geojsonOptionsDenkmal: {
        pointToLayer: (feature, latlng) => {
          return L.marker(latlng, { icon: denkmalIcon });
        }
      },
    };
  },
  methods: {
    handleFilter(itemId) {
      const clickedElement = document.getElementById(itemId);

      //activate if wasn't active before
      if (!clickedElement) {
        console.error(`Element with id ${itemId} not found.`);
        return;
      }
      clickedElement.classList.toggle('active');
    },
    filterLayers(id) {
      this.layerVisibility[id].visible = !this.layerVisibility[id].visible;
    },
    filterLayers2(id) {
      this.layerVisibility[id].visible = !this.layerVisibility[id].visible;
    }
  }
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