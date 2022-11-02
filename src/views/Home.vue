<template>
  <div class="flex flex-col h-screen max-h-screen">
    <!-- Search / Result -->
    <div class="z-20 flex justify-center relative bg-hero-pattern bg-cover px-4 pt-8 pb-32">
      <!-- Search Input -->
      <div class="w-full max-w-screen-sm">
        <h1 class="text-white text-center text-3xl pb-4 font-bold">IP Address Tracker</h1>
        <div class="flex">
          <input v-model="queryIp" type="text" class="flex-1 py-3 px-3 rounded-tl-md rounded-bl-md focus:outline-none"
            placeholder="Search for any IP address or leave empty to get your ip info">
          <i @click="getIpInfo" class="cursor-pointer bg-black text-white px-4 rounded-tr-md rounded-br-md flex items-center fas fa-chevron-right"></i>
        </div>
      </div>
      <!-- IP Info -->
      <IPInvo v-if="ipInfo" v-bind:ipInfo="ipInfo" />
    </div>

    <!-- Map -->
    <div id="map" class="h-full z-10"></div>
  </div>
</template>

<script>
import IPInvo from '../components/IPInfo.vue';
import leaflet from 'leaflet';
import { onMounted, ref } from 'vue';
import axios from 'axios';

export default {
  name: "Home",
  components: { IPInvo },
  setup() {
    let map;
    const queryIp = ref("")
    const ipInfo = ref(null)

    onMounted(() => {
      map = leaflet.map('map').setView([-6.21462, 106.84513], 16);
      leaflet.marker([-6.21462, 106.84513]).addTo(map);


      L.tileLayer(`https://api.mapbox.com/styles/v1/{id}/tiles/{z}/{x}/{y}?access_token=pk.eyJ1IjoibHZ4eHl6IiwiYSI6ImNsOXpvcHZ1azBnaDAzcW8zeTZyYnpkYXEifQ.hep-pQ9QDIcxO7g7KbM21g`, {
        attribution: '© <a href="https://www.mapbox.com/about/maps/">Mapbox</a> © <a href="http://www.openstreetmap.org/copyright">OpenStreetMap</a> <strong><a href="https://www.mapbox.com/map-feedback/" target="_blank">Improve this map</a></strong>',
        tileSize: 512,
        maxZoom: 18,
        zoomOffset: -1,
        id: 'mapbox/streets-v11',
        accessToken: 'pk.eyJ1IjoibHZ4eHl6IiwiYSI6ImNsOXpvcHZ1azBnaDAzcW8zeTZyYnpkYXEifQ.hep-pQ9QDIcxO7g7KbM21g'
      }).addTo(map);
    });

    const getIpInfo = async () => {
      try {
        const data = await axios.get(
          `https://geo.ipify.org/api/v2/country,city,vpn?apiKey=at_9gkVu3oSei7fKxyzIILHvaFxAk8Sg&ipAddress=${queryIp.value}`
        )

        const result = data.data;
        ipInfo.value = {
          address: result.ip,
          state: result.location.region,
          timezone: result.location.timezone,
          isp: result.isp,
          lat: result.location.lat,
          lng: result.location.lng
        }
        leaflet.marker([ipInfo.value.lat, ipInfo.value.lng]).addTo(map);
        map.setView([ipInfo.value.lat, ipInfo.value.lng], 22);
      } catch(err) {
        alert(err.message)
      }
    }

    return { queryIp, ipInfo, getIpInfo }
  },
}
</script>