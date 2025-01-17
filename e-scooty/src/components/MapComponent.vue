<script setup>
  var filterOneOptions = ["Straßennetz", "Veloruten", "E-Scooter Abstellplätze", "Bushaltestellen", "E-Scooter Parkverbot", "Bebauungsfläche", "Straßenbeleuchtung", "Denkmäler"];
  var filterTwoOptions = ["E-Scooter Abstellplätze", "Buslinien", "Nahverkehr", "Überschneidungen", "Gebietsansicht"];
</script>


<template>
  <div class="nav justify-content-center">
    <img src="../assets/logo.png" alt="Logo" class="logo settings-img mt-1">
    <h1 class="ms-2 me-2">E-Scooty</h1>
    <img src="../assets/logo.png" alt="Logo" class="logo settings-img mt-1">
  </div>

  <div>
    <div> <!-- Filter active / completed project partizipation -->
      <button @click="dropdownInformation = !dropdownInformation; callOnce()" class="btn btn-outline-dropdown align-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow w-25" data-bs-toggle="dropdown" aria-expanded="false" id="filter-layers">Ebenen</button>
      <ul v-show="dropdownInformation" @mouseleave="dropdownInformation = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 shadow w-25 z-2 ml-filter-one" id="layer-options">
        <li v-for="(option, index) in filterOneOptions" :key="index">
          <a 
            class="dropdown-item rounded-2 layer-option" 
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

      <button @click="dropdownHeat = !dropdownHeat" class="btn btn-outline-dropdown a%lign-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow w-25" data-bs-toggle="dropdown" aria-expanded="false" id="filter-heat" style="margin-left: 10vw">Planungshilfen</button>
      <ul v-show="dropdownHeat" @mouseleave="dropdownHeat = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 shadow w-25 z-2 ml-filter-two" id="heat-options">
        <li v-for="(option, index) in filterTwoOptions" :key="index">
          <a 
            class="dropdown-item rounded-2 heat-option" 
            @mousedown="noneLoaded = false; handleFilter('heat-' + index); SetupHeatmaps(index)" 
            :id="'heat-' + index" 
            @mouseup="filterLayers2((index + 8))" 
            aria-current="page">
              <img class="s-img me-2" v-if="layerVisibility[(index + 8)].visible" :src="CheckBox" alt="Checkbox" />
              <img class="s-img me-2" v-else :src="CheckBoxOff" alt="Checkbox" />
              {{ option }}
          </a>
        </li>
      </ul>
    </div>
  </div>

  <div class = "format center">
    <l-map ref="map" v-model:zoom="zoom" :center="[53.75153044444902, 9.664619800111053]" @click="checkPlace" :bounds="bounds" :max-bounds="maxBounds" >
      <l-tile-layer
        url="https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"
        layer-type="base"
        name="OpenStreetMap"
        minZoom="12"
        maxZoom="18"
      ></l-tile-layer>
      <button v-if="layerVisibility[10].visible" @click="enablePointPlacement" class="btn btn-primary shadow me-5" style="z-index: 1000; position: relative; top: 10px;">
        Platziere Punkt
      </button>
      <button v-if="layerVisibility[10].visible" @click="resetPoints" :disabled="!newPoints || newPoints.features.length === 0" class="btn btn-primary shadow" style="z-index: 1000; position: relative; top: 10px;">
        Zurücksetzen
      </button>
      <button v-if="layerVisibility[8].visible" @click="enablePointPlacementLot" class="btn btn-primary shadow me-5" style="z-index: 1000; position: relative; top: 10px;">
        Platziere Punkt
      </button>
      <button v-if="layerVisibility[8].visible" @click="resetPointsLot" :disabled="!newPointsLot || newPointsLot.features.length === 0" class="btn btn-primary shadow" style="z-index: 1000; position: relative; top: 10px;">
        Zurücksetzen
      </button>
      <button v-if="layerVisibility[9].visible" @click="enablePointPlacementBus" class="btn btn-primary shadow me-5" style="z-index: 1000; position: relative; top: 10px;">
        Platziere Punkt
      </button>
      <button v-if="layerVisibility[9].visible" @click="resetPointsBus" :disabled="!newPointsBus || newPointsBus.features.length === 0" class="btn btn-primary shadow" style="z-index: 1000; position: relative; top: 10px;">
        Zurücksetzen
      </button>
      <l-geo-json :geojson="cityLimits" :options="cityLimitOptions"/>
      <l-geo-json v-if="layerVisibility[6].visible" :geojson="elmshornStrassenbeleuchtung" :options="geojsonOptionsStrassenbeleuchtung" />
      <l-geo-json v-if="layerVisibility[0].visible" :geojson="elmshornStraßennetz" :options="geojsonOptionsStraßennetz" />
      <l-geo-json v-if="layerVisibility[1].visible" :geojson="elmshornVelorouten" :options="geojsonOptionsVelorouten" />
      <l-geo-json v-if="layerVisibility[4].visible" :geojson="escooterParkverbot" :options="geojsonOptionsParkverbot" />
      <l-geo-json v-if="layerVisibility[2].visible" :geojson="escooterStellplatz" :options="geojsonOptionsStellplatz" />
      <l-geo-json v-if="layerVisibility[3].visible" :geojson="elmshornHaltestellen" :options="geojsonOptionsHaltestellen" />
      <l-geo-json v-if="layerVisibility[5].visible" :geojson="elmshornBebauungsFlaeche" :options="geojsonOptionsBebauungsFlaeche" />
      <l-geo-json v-if="layerVisibility[7].visible" :geojson="elmDenkmal" :options="geojsonOptionsDenkmal" />
      <l-geo-json v-if="layerVisibility[8].visible && tinurfLot" :geojson="tinurfLot" :options="geojsonOptionsTin"/>
      <l-geo-json v-if="layerVisibility[9].visible && tinurfBus" :geojson="tinurfBus" :options="geojsonOptionsTin"/>
      <l-geo-json v-if="layerVisibility[10].visible && tinurf" :geojson="tinurf" :options="geojsonOptionsTin"/>
      <l-geo-json v-if="layerVisibility[11].visible" :geojson="bufferedgeojson" :options="bufferStyle"/>
      <l-geo-json v-if="layerVisibility[10].visible && addIcons" :geojson="addIcons" :options="bufferStyle"/>
      <l-geo-json v-if="layerVisibility[8].visible && addIconsLot" :geojson="addIconsLot" :options="bufferStyle"/>
      <l-geo-json v-if="layerVisibility[9].visible && addIconsBus" :geojson="addIconsBus" :options="bufferStyle"/>
      <l-geo-json :geojson="cityLimitsMask" :options="cityLimitMaskOptions"/>
      <div v-if="layerVisibility[8].visible || layerVisibility[9].visible || layerVisibility[10].visible" class="legend" style="z-index: 1000; position: relative; top: -3vh;">
        <h4>Abdeckung</h4>
        <div class="gradient-bar"></div>
        <div class="legend-labels">
          <span>hoch</span>
          <span>niedrig</span>
        </div>
  </div>
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
import HeatmapTest from "../assets/HeatmapTest.geojson";
import CityLimits from "../assets/FP_Bereich.geojson";
import CityLimitsMask from "../assets/FP_Bereich_Mask.geojson";
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
      dropdownHeat: false,
      layerVisibility: [
        { id: 0, name: "Straßennetz", visible: false },
        { id: 1, name: "Veloruten", visible: false },
        { id: 2, name: "Stellplätze", visible: true },
        { id: 3, name: "Bushaltestellen", visible: true },
        { id: 4, name: "Parkverbot", visible: true },
        { id: 5, name: "Bebauungsfläche", visible: false },
        { id: 6, name: "Straßenbeleuchtung", visible: false },
        { id: 7, name: "Denkmäler", visible: false },
        { id: 8, name: "Tinurf", visible: false },
        { id: 9, name: "TinBus", visible: false },
        { id: 10, name: "TinBoth", visible: false },
        { id: 11, name: "reUnion", visible: false },
        { id: 12, name: "gebiete", visible: false }
      ],
      zoom: 13,
      bounds: [
        [53.7, 9.8],
        [53.8, 9.5]
      ],
      maxBounds: [
        [53.7, 9.8],
        [53.8, 9.5]
      ],
      tinurfBus: null,
      tinurfLot: null,
      tinurf: null,
      once: true,
      newPoints: {
        type: "FeatureCollection", // GeoJSON-Typ
        features: [], // Features übernehmen
      },
      newPointsLot: {
        type: "FeatureCollection", // GeoJSON-Typ
        features: [], // Features übernehmen
      },
      newPointsBus: {
        type: "FeatureCollection", // GeoJSON-Typ
        features: [], // Features übernehmen
      },
      pointPlacementEnabled: false,
      pointPlacementEnabledLot: false,
      pointPlacementEnabledbus: false,
      bufferedgeojson: null,
      escooterStellplatz: Escooter_Stellplatz,
      escooterParkverbot: EScooter_Parkverbot_EL,
      elmshornHaltestellen: Haltestellen,
      elmshornStrassenbeleuchtung: Strassenbeleuchtung,
      elmshornStraßennetz: Straßennetz,
      elmshornVelorouten: Velorouten,
      elmshornBebauungsFlaeche: BebauungsFlaeche,
      elmDenkmal: Denkmal,
      cityLimits: CityLimits,
      cityLimitsMask: CityLimitsMask,
      escooterStellplatzZentrum: StellplatzZentrum,
      heatmapTesting: HeatmapTest,
      geojsonOptionsStellplatz: {
      // Färbt die jeweilige genannte EScooterID ein
        color: 'blue',
       onEachFeature: (feature, layer) => {
           layer.on({
      //       click: () => {
      //         layer.bindPopup(`Stellplatz ID: ${feature.properties.Escoote_ID}`).openPopup();
      //         layer.setStyle({
      //         color: 'red'
      //       });
      //       },
             mouseover: () => {
      //         layer.setStyle({
      //           weight: 7
      //         })
              layer.bindPopup("E-Scooter Abstellplatz").openPopup();
             },
             mouseout: () => {
      //         layer.setStyle({
      //           weight: 2
      //         })
      layer.closePopup();
             },
      //       popupclose: () => {
      //         layer.setStyle({
      //           color: 'blue'
      //         })
      //       }
           })
         }
      },

      geojsonOptionsTin: {
        onEachFeature: (feature, layer) => {          
          const area = turf.area(feature); // Berechne die Fläche
          let color = "white";
          if (area < 25000) color = "#1d4877";
          else if (area < 50000) color = "#00544d";
          else if (area < 75000) color = "#1b8a5a";
          else if (area < 100000) color = "#fbb021";
          else if (area < 150000) color = "	#f68838";
          //else if (area < 200000) color = "blue";
          else color = "#ee3e32";

          layer.setStyle({
            color: color,
            weight: 1,
            fillOpacity: 0.5,    
          })
        }
      },

      geojsonOptionsParkverbot: {
        color: 'red',
        onEachFeature: (feature, layer) => {
          layer.on({
              mouseover: () => {
                layer.bindPopup("Parkverbot für E-Scooter").openPopup();
              },
              mouseout: () => {
                layer.closePopup();
              },
            })
        }
      },
      geojsonOptionsHaltestellen: {
        pointToLayer: (feature, latlng) => {
          return L.marker(latlng, { icon: busStopIcon });
        },
        onEachFeature: (feature, layer) => {
          layer.on({
            mouseover: () => {
              layer.bindPopup(`Bushaltestelle: ${feature.properties.NAME}`).openPopup();
            },
            mouseout: () => {
              layer.closePopup();
            },
          })
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
      cityLimitOptions: {
        color: 'black',
        opacity: 0,
        fillOpacity: 0.1
      },
      cityLimitMaskOptions: {
        color: 'black',
        opacity: 0,
        fillOpacity: 0.6
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
            layer.on({
              mouseover: () => {
                layer.bindPopup("Wohngebiet").openPopup();
              },
              mouseout: () => {
                layer.closePopup();
              },
            })
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 2000) {
            layer.setStyle({
              color: '#6cc0df'
            });
            layer.on({
              mouseover: () => {
                layer.bindPopup("Gewerbegebiet").openPopup();
              },
              mouseout: () => {
                layer.closePopup();
              },
            })
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 3000) {
            layer.setStyle({
              color: '#a8afb1'
            });
            layer.on({
              mouseover: () => {
                layer.bindPopup("Industriegebiet").openPopup();
              },
              mouseout: () => {
                layer.closePopup();
              },
            })
          }
          if (feature.properties && feature.properties.allgArtDerBaulNutzung === 4000) {
            layer.setStyle({
              color: '#a8afb1'
            });
            layer.on({
              mouseover: () => {
                layer.bindPopup("Sondergebiet").openPopup();
              },
              mouseout: () => {
                layer.closePopup();
              },
            })
          }
        }
      },
      geojsonOptionsDenkmal: {
        pointToLayer: (feature, latlng) => {
          return L.marker(latlng, { icon: denkmalIcon });
        },
        onEachFeature: (feature, layer) => {
          layer.on({
            mouseover: () => {
              layer.bindPopup(`Denkmal: ${feature.properties.NAME}`).openPopup();
            },
            mouseout: () => {
              layer.closePopup();
            },
          })
        }
      },
    };
  },
  methods: {
    callOnce () {
      if (this.once) {
        document.getElementById('layer-' + 2).classList.toggle('active');
        document.getElementById('layer-' + 3).classList.toggle('active');
        document.getElementById('layer-' + 4).classList.toggle('active');
        this.once = false;
      }
    },
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
      if (this.layerVisibility[id].visible) {
        for (var i = 8; i < 13 ; i++) {
          if (this.layerVisibility[i].visible && i != id) {
            this.handleFilter('heat-' + (i-8));
            this.layerVisibility[i].visible = false;
          }
        }
      }
      if (id === 8 && this.layerVisibility[id].visible) {
        for (i = 0; i < 8 ; i++) {
          if (this.layerVisibility[i].visible && i != 2) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
          else if (!this.layerVisibility[i].visible && i == 2) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
        }
      } else if (id === 9 && this.layerVisibility[id].visible) {
        for (i = 0; i < 8 ; i++) {
          if (this.layerVisibility[i].visible && i != 3) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
          else if (!this.layerVisibility[i].visible && i == 3) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
        }
      } else if (id === 10 && this.layerVisibility[id].visible) {
        for (i = 0; i < 8 ; i++) {
          if (this.layerVisibility[i].visible && i != 3 && i != 2) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
          else if (!this.layerVisibility[i].visible && (i == 2 || i == 3)) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
        }
      } else if (id === 11 && this.layerVisibility[id].visible) {
        for (i = 0; i < 8 ; i++) {
          if (this.layerVisibility[i].visible && i != 2 && i != 4) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
          else if (!this.layerVisibility[i].visible && (i == 2 || i == 4)) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
        }
      } else if (id === 12 && this.layerVisibility[id].visible) {
        for (i = 0; i < 8 ; i++) {
          if (this.layerVisibility[i].visible && i != 2 && i != 5) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
          else if (!this.layerVisibility[i].visible && (i == 2 || i == 5)) {
            this.handleFilter('layer-' + i);
            this.filterLayers(i);
          }
        }
      }
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

    createBuffer() {
      const bufferDistance = 25;
      const buffered = turf.buffer(this.elmshornHaltestellen, bufferDistance, { units: "metres" });
      this.bufferedgeojson = buffered;
      console.log("Buffered GeoJSON: ", buffered);
    },

    SetupHeatmaps(id){
      if (id === 0) this.tinningLot();
      if (id === 1) this.tinningBus();
      if (id === 2) this.tinning();
    },
    tinningLot() {
      var combinedBlot = {
        type: "FeatureCollection", // GeoJSON-Typ
        features: [...this.escooterStellplatzZentrum.features, ...this.newPointsLot.features], // Features übernehmen
      };
      const newTin = turf.tin(combinedBlot);
      this.tinurfLot = JSON.parse(JSON.stringify(newTin));
    },
    tinningBus() {
      var combinedBlot = {
        type: "FeatureCollection", // GeoJSON-Typ
        features: [...this.elmshornHaltestellen.features, ...this.newPointsBus.features], // Features übernehmen
      };
      const newTin = turf.tin(combinedBlot);
      this.tinurfBus = JSON.parse(JSON.stringify(newTin));
    },
    tinning() {
      var combinedBlot = {
        type: "FeatureCollection", // GeoJSON-Typ
        features: [...this.elmshornHaltestellen.features, ...this.escooterStellplatzZentrum.features, ...this.newPoints.features], // Features übernehmen
      };
      const newTin = turf.tin(combinedBlot);
      this.tinurf = JSON.parse(JSON.stringify(newTin));
    },
    checkPlace(event) {
      const latlng = event.latlng;
      if (this.layerVisibility[10].visible) {
        this.placeNewPoint(latlng);
      } else if (this.layerVisibility[8].visible) {
        this.placeNewPointLot(latlng);
      } else if (this.layerVisibility[9].visible) {
        this.placeNewPointBus(latlng);
      }
    },
    enablePointPlacement(event) {
      event.stopPropagation();
      this.pointPlacementEnabled = true;
    },
    resetPoints(event) {
      event.stopPropagation();
      this.newPoints.features = [];
      this.addIcons = JSON.parse(JSON.stringify(""));
      //this.escooterStellplatzZentrum.features = this.escooterStellplatzZentrum.features.filter(feature => !this.newPoints.includes(feature));
      this.tinning();
    },
    placeNewPoint(event) {
      if (this.pointPlacementEnabled) { // Linksklick
        const latlng = event;
        
        const newPoint = {
          type: "Feature",
          geometry: {
            type: "Point",
            coordinates: [latlng.lng, latlng.lat],
          },
          properties: {},
        };
        
        if (turf.pointsWithinPolygon(newPoint, this.cityLimits).features.length > 0) {	
          this.newPoints.features.push(newPoint);

          var combinedIcons = {
            type: "FeatureCollection", // GeoJSON-Typ
            features: [...this.newPoints.features], // Features übernehmen
          };
          const newTin = turf.combine(combinedIcons);
          this.addIcons = JSON.parse(JSON.stringify(newTin));

          this.tinning();
          this.pointPlacementEnabled = false;
        }
      }
    },
    enablePointPlacementLot(event) {
      event.stopPropagation();
      this.pointPlacementEnabledLot = true;
    },
    resetPointsLot(event) {
      event.stopPropagation();
      this.newPointsLot.features = [];
      this.addIconsLot = JSON.parse(JSON.stringify(""));
      //this.escooterStellplatzZentrum.features = this.escooterStellplatzZentrum.features.filter(feature => !this.newPoints.includes(feature));
      this.tinningLot();
    },
    placeNewPointLot(event) {

      if (this.pointPlacementEnabledLot) { // Linksklick
        const latlng = event;
        
        const newPoint = {
          type: "Feature",
          geometry: {
            type: "Point",
            coordinates: [latlng.lng, latlng.lat],
          },
          properties: {},
        };

        console.log(turf.pointsWithinPolygon(newPoint, this.cityLimits).features.length > 0);
        
        if (turf.pointsWithinPolygon(newPoint, this.cityLimits).features.length > 0) {	
          this.newPointsLot.features.push(newPoint);

          var combinedIcons = {
            type: "FeatureCollection", // GeoJSON-Typ
            features: [...this.newPointsLot.features], // Features übernehmen
          };
          const newTin = turf.combine(combinedIcons);
          this.addIconsLot = JSON.parse(JSON.stringify(newTin));

          this.tinningLot();
          this.pointPlacementEnabledLot = false;
        }
      }
    },
    enablePointPlacementBus(event) {
      event.stopPropagation();
      this.pointPlacementEnabledBus = true;
    },
    resetPointsBus(event) {
      event.stopPropagation();
      this.newPointsBus.features = [];
      this.addIconsBus = JSON.parse(JSON.stringify(""));
      //this.escooterStellplatzZentrum.features = this.escooterStellplatzZentrum.features.filter(feature => !this.newPoints.includes(feature));
      this.tinningBus();
    },
    placeNewPointBus(event) {
      if (this.pointPlacementEnabledBus) { // Linksklick
        const latlng = event;
        
        const newPoint = {
          type: "Feature",
          geometry: {
            type: "Point",
            coordinates: [latlng.lng, latlng.lat],
          },
          properties: {},
        };
        console.log(turf.pointsWithinPolygon(newPoint, this.cityLimits).features.length > 0);
        
        if (turf.pointsWithinPolygon(newPoint, this.cityLimits).features.length > 0) {	
          this.newPointsBus.features.push(newPoint);

          var combinedIcons = {
            type: "FeatureCollection", // GeoJSON-Typ
            features: [...this.newPointsBus.features], // Features übernehmen
          };
          const newTin = turf.combine(combinedIcons);
          this.addIconsBus = JSON.parse(JSON.stringify(newTin));

          this.tinningBus();
          this.pointPlacementEnabledBus = false;
        }
      }
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
  mounted() {
    this.createBuffer();
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

.custom-icon-denk{
  filter: invert(49%) sepia(6%) saturate(448%) hue-rotate(305deg) brightness(103%) contrast(89%);
}

.custom-icon-bus{
  filter: invert(43%) sepia(5%) saturate(4177%) hue-rotate(140deg) brightness(89%) contrast(86%);
}
</style>

<style scoped>
/* ...existing code... */
.legend {
  position: absolute;
  bottom: 10px;
  left: 10px;
  background: white;
  padding: 10px;
  border-radius: 5px;
  box-shadow: 0 0 15px rgba(0, 0, 0, 0.2);
  width: 300px;
}
.legend h4 {
  margin: 0 0 10px;
}
.gradient-bar {
  height: 20px;
  background: linear-gradient(to right, #1d4877, #00544d, #1b8a5a, #fbb021, #f68838, #ee3e32);
  border: 1px solid #000;
  margin-bottom: 10px;
}
.legend-labels {
  display: flex;
  justify-content: space-between;
}
.legend-labels span {
  font-size: 12px;
}
</style>
