<template>
  <div>
    <h1>E Scooty</h1>
        
    <div> <!-- Filter active / completed project partizipation -->
      <button @click="dropdownInformation = !dropdownInformation"  class="btn btn-outline-dropdown align-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow" data-bs-toggle="dropdown" aria-expanded="false" id="filter-active">Verkehr</button>
      <ul v-show="dropdownInformation" @:mouseleave="dropdownInformation = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 mx-0 shadow w-220px" id="traffic-options">
        <li><a class="dropdown-item rounded-2 active" @mousedown="noneLoaded = false" id="alle-projekte" v-on:mouseup="dropdownInformation = !dropdownInformation;" aria-current="page">Alle Filter</a></li>
        <li><a class="dropdown-item rounded-2" @mousedown="noneLoaded = false" v-on:mouseup="dropdownInformation = !dropdownInformation;" id="aktuelle">Parkplätze</a></li>
        <li><a class="dropdown-item rounded-2" @mousedown="noneLoaded = false" v-on:mouseup="dropdownInformation = !dropdownInformation;" id="abgeschlossene">Verbotszonen</a></li>
      </ul>

      <button @click="dropdownActuality = !dropdownActuality"  class="btn btn-outline-dropdown align-items-center mb-2 mb-lg-0 text-decoration-none dropdown-toggle button-size shadow" data-bs-toggle="dropdown" aria-expanded="false" id="filter-active" style="margin-left: 10vw">Anderer Filter</button>
      <ul v-show="dropdownActuality" @:mouseleave="dropdownActuality = false" class="vue-dropdown-menu list-style-none text-small gap-1 p-2 rounded-3 mx-0 shadow w-220px" id="active-options">
        <li><a class="dropdown-item rounded-2 active" @mousedown="noneLoaded = false" id="alle-projekte" v-on:mouseup="dropdownActuality = !dropdownActuality;" aria-current="page">Alle krams</a></li>
        <li><a class="dropdown-item rounded-2" @mousedown="noneLoaded = false" v-on:mouseup="dropdownActuality = !dropdownActuality;" id="aktuelle">confusion</a></li>
        <li><a class="dropdown-item rounded-2" @mousedown="noneLoaded = false" v-on:mouseup="dropdownActuality = !dropdownActuality;" id="abgeschlossene">?!</a></li>
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
      <l-geo-json :geojson="escooterStellplatz" :options="geojsonOptions2" />
      <l-geo-json :geojson="escooterParkverbot" :options="geojsonOptions" />
    </l-map>
  </div>
</template>

<script>
import "leaflet/dist/leaflet.css";
import { LMap, LTileLayer, LGeoJson } from "@vue-leaflet/vue-leaflet";
import Escooter_Stellplatz from "../assets/Escooter_Stellplatz.geojson";
import EScooter_Parkverbot_EL from "../assets/EScooter_Parkverbot_EL.geojson";

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
      geojsonOptions: {
        //"color": "#ffffff",
      },
      geojsonOptions2: {
        "stroke": "true",
        "color": "#ff0000",
        "weight": "3",
      }
    };
  },
};
</script>

<style>
@import "bootstrap";

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

/* ~~~~~~~~~~~ Button designs (Colors) ~~~~~~~~~~~ */
.btn-outline-dropdown {
  background-color: #FFFFFF;
  --bs-btn-border-color: #808080;
  --bs-btn-hover-border-color: #808080;
  --bs-gradient: none;
}

.accept, .dropdown-item.active {
  background-color: #90EE90;
  color: #000000;
  --bs-btn-border-color: #808080;
  --bs-gradient: none;
}
.dropdown-item.active:hover, #leichte-sprache .green:hover {
  background: #55CC55;
}

.accept:hover, .cancel {
  background-color: #D3D3D3;
  --bs-btn-hover-border-color: #6C757D;
  --bs-btn-border-color: #808080;
  --bs-gradient: none;
}

a, .accept:hover, .cancel {
  color: #000000;
}

.green {
  background: #90EE90;
}

#top-nav {
  border-top: #F1F1F1 1px solid;
}


/* ~~~~~~~~~~~~~~~~~~~~~~ Nav bars ~~~~~~~~~~~~~~~~~~~~~~ */
nav {
  position: fixed;
  display: inline-block;
  width: 100vw;
}

#top-nav {
  top: 56px;
  position: fixed;
}

#bottom-nav {
  bottom: 0;
}

.list-style-none {
  list-style: none;
}

.vue-dropdown-menu, .vue-dropdown-menu-settings {
  position: absolute;
  z-index: var(--bs-dropdown-zindex);
  margin: 7px 0 0;
  text-align: left;
  list-style: none;
  border: var(--bs-dropdown-border-width) solid var(--bs-dropdown-border-color);
  border-radius: var(--bs-dropdown-border-radius);
}
.vue-dropdown-menu li, .vue-dropdown-menu-settings li {
  margin: 3px;
}

.medium-width {
  min-width: 30vw;
}

.display-right, .display-simple-right {
  text-align: right;
}

img {
  object-fit: cover;
}

.settings-img {
  object-fit: contain;
  height: 4vh;
  width: 5vw;
}

.s-img {
  object-fit: contain;
  height: 3vh;
  width: 3vw;
}

.project-image {
  min-height: 10vh;
  max-height: 15vh;
  width: 100%;
}

.search-img {
  height: inherit;
  object-fit: contain;
}

.desktop-filter h3 {
  margin-right: 3vw;
}

.project-search {
  width: 85%;
}
</style>