<template>
  <div class="flex flex-col flex-1 items-center">
    <!-- Banner -->
    <div
      v-if="route.query.preview"
      class="text-white p-4 bg-weather-secondary w-full text-center"
    >
      <p>
        You are currently previewing this city, click the
        <i class="fa-solid fa-circle-plus"></i> icon to start tracking this
        city.
      </p>
    </div>
    <!-- Fuel Overview -->
    <div class="flex flex-col items-center text-white py-12">
      <h1 class="text-4xl mb-2">{{ route.params.city }}</h1>
      <p class="text-sm mb-12">
        {{ new Date(fuelData) }}
      </p>
    </div>
  </div>
</template>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";

const route = useRoute();
const getFuelData = async () => {
  try {
    const fuelData = await axios.get(
      `https://creativecommons.tankerkoenig.de/json/list.php?lat=${route.query.lat}&lng=${route.query.lng}&rad=25&sort=dist&type=all&apikey=2ec47aaf-4e2f-1262-ab18-bfaec1e47136`
    );
    return fuelData.data;
  } catch (err) {
    console.log(err);
  }
};
const fuelData = await getFuelData();
</script>
