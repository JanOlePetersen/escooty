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
      <button @click="checkIntersections" class="btn btn-primary shadow" style="margin-left: 10vw">
        Prüfe Überschneidungen
      </button>
      <button @click="centreOfMass" class="btn btn-primary shadow" style="margin-left: 10vw">
        Massezentrum bidde?
      </button>
      <button @click="tinning" class="btn btn-primary shadow" style="margin-left: 10vw">
        Masse wird klasse
      </button>
    </div>
  </div>


  <div class = "format center">
    <l-map ref="map" v-model:zoom="zoom" :center="[53.75153044444902, 9.664619800111053]">
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
        minZoom="12"
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
      <l-geo-json v-if="layerVisibility[8].visible && tinurf" :geojson="tinurf" :options="geojsonOptionsTin"/>

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
import StellplatzZentrum from "../assets/StellplatzZentrum.geojson";
import BebauungsFlaeche from "../assets/BebauungsFlaeche.geojson";
import Denkmal from "../assets/Denkmal.geojson";
import L from "leaflet";

import * as turf from "@turf/turf";

import CheckBox from "../assets/square-check-regular.svg";
import CheckBoxOff from "../assets/square-regular.svg";

import busStopIconUrl from "../assets/bus-solid.svg"; 
const busStopIcon = L.icon({
  iconUrl: busStopIconUrl,
  iconSize: [15, 15],
  className: 'custom-icon-bus', // Größe des Icons
  // iconAnchor: [12, 25], // Punkt des Icons, der auf den Marker zeigt
  // popupAnchor: [0, -25] // Punkt, von dem aus das Popup relativ zum Icon geöffnet wird
});
import denkmalIconUrl from "../assets/landmark-solid.svg"; 
const denkmalIcon = L.icon({
  iconUrl: denkmalIconUrl,
  iconSize: [15, 15],
  className: 'custom-icon-denk', // Größe des Icons
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
        { id: 7, name: "Denkmäler", visible: true },
        { id: 8, name: "Tinurf", visible: true }
      ],
      zoom: 13,
      tinurf: null,
      escooterStellplatz: Escooter_Stellplatz,
      escooterParkverbot: EScooter_Parkverbot_EL,
      elmshornHaltestellen: Haltestellen,
      elmshornStrassenbeleuchtung: Strassenbeleuchtung,
      elmshornStraßennetz: Straßennetz,
      elmshornVelorouten: Velorouten,
      elmshornBebauungsFlaeche: BebauungsFlaeche,
      elmDenkmal: Denkmal,
      escooterStellplatzZentrum: StellplatzZentrum,
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

      geojsonOptionsTin: {
        onEachFeature: (feature, layer) => {

          layer.on({
            mouseover: () => {
              const area = turf.area(feature); // Berechne die Fläche
              let color = "white";
              if (area < 25000) color = "#ee3e32";
              else if (area < 50000) color = "#f68838";
              else if (area < 75000) color = "#fbb021";
              else if (area < 100000) color = "#1b8a5a";
              else if (area < 150000) color = "	#00544d";
              //else if (area < 200000) color = "blue";
              else color = "#1d4877";

              layer.setStyle({
                color: color,
                weight: 1,
                fillOpacity: 0.5,
              })
            }
          })
        }
      },

      geojsonOptionsParkverbot: {
        color: 'red'
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
              color: '#7bb35d'
            });
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 2000) {
            layer.setStyle({
              color: '#6cc0df'
            });
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 3000) {
            layer.setStyle({
              color: '#a8afb1'
            });
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 4000) {
            layer.setStyle({
              color: '#a8afb1'
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
    async filterLayers(id) {
      if (this.layerVisibility[id].visible) {
        this.layerVisibility[id].visible = !this.layerVisibility[id].visible;
        return;
      }
      
      var wasNotLoaded = [];
      for (let i = id-1; i >= 0; i--) {
        if (this.layerVisibility[i].visible) this.layerVisibility[i].visible = !this.layerVisibility[i].visible;
        else wasNotLoaded.push(this.layerVisibility[i].id);
      }
      this.layerVisibility[id].visible = !this.layerVisibility[id].visible;
      for (let i = id-1; i >= 0; i--) {
        if (!wasNotLoaded.includes(i)) {
          this.layerVisibility[i].visible = !this.layerVisibility[i].visible;
          await this.delay(0.1);
        }
      }
    },
    filterLayers2(id) {
      this.layerVisibility[id].visible = !this.layerVisibility[id].visible;
    },
    delay(ms) {
      return new Promise(resolve => setTimeout(resolve, ms));

    },

    centreOfMass() {
      var centres = [];
      const stellplatzFeatures = this.escooterStellplatz.features;
      stellplatzFeatures.forEach((stellplatzFeature) => {
        centres.push(turf.centerOfMass(stellplatzFeature).geometry.coordinates);
      })
      console.log("Centres: " + centres);
    },

    tinning() {
      //console.log(turf.tin(this.elmshornHaltestellen));
      const newTin = turf.tin(this.elmshornHaltestellen);
      this.tinurf = JSON.parse(JSON.stringify(newTin));
  console.log("TIN GeoJSON:", this.tinurf);
    },

    checkIntersections() {
      var i = 0;
      var j = 0;
      const intersections = [];
      const parkverbotFeatures = this.escooterParkverbot.features;
      const stellplatzFeatures = this.escooterStellplatz.features;

      stellplatzFeatures.forEach((stellplatzFeature) => {
        i++;
        j = 0;
        parkverbotFeatures.forEach((parkverbotFeature) => {
          
          const isIntersecting = turf.booleanIntersects(stellplatzFeature, parkverbotFeature);
          if (isIntersecting) {
            intersections.push({
              stellplatzId: i,
              parkverbotId: j,
            });
            return;
          }
          j++;
        });
      });

      if (intersections.length > 0) {
        intersections.forEach((intersection) => {
          console.log(`{ ${intersection.stellplatzId}, ${intersection.parkverbotId} }`);
        });
        console.log("Überschneidungen gefunden:", intersections);
        alert(`Überschneidungen gefunden: ${intersections.length}`);
      } else {
        console.log("Keine Überschneidungen gefunden.");
        alert("Keine Überschneidungen gefunden.");
      }
    },
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

.custom-icon-denk{
  filter: invert(49%) sepia(6%) saturate(448%) hue-rotate(305deg) brightness(103%) contrast(89%);
}

.custom-icon-bus{
  filter: invert(43%) sepia(5%) saturate(4177%) hue-rotate(140deg) brightness(89%) contrast(86%);
}
</style>