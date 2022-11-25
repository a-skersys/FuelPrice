<template>
  <div v-for="city in savedCities" :key="city.id">
    <CityCard :city="city" @click="goToCityView(city)" />
  </div>

  <p v-if="savedCities.length === 0">
    No locations added. To start tracking a locations, search in the field
    above.
  </p>
</template>

<script setup lang="ts">
import axios from "axios";
import { ref } from "vue";
import { useRouter } from "vue-router";
import type { Ref } from "vue";
import CityCard from "./CityCard.vue";

const savedCities: Ref<any> = ref([]);
const getCities = async () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities") as string
    );

    console.log(savedCities.value);

    const requests = [] as any;

    console.log(requests);
    savedCities.value.forEach((city: any) => {
      requests.push(
        axios.get(
          `https://creativecommons.tankerkoenig.de/json/list.php?lat=${city.coords.lat}&lng=${city.coords.lng}&rad=${city.range}&sort=price&type=${city.type}&apikey=2ec47aaf-4e2f-1262-ab18-bfaec1e47136`
        )
      );
    });

    console.log(requests);

    const fuelData = await Promise.all(requests);

    fuelData.forEach((value, index) => {
      savedCities.value[index].fuel = value.data;
    });
  }
};

await getCities();

const router = useRouter();
const goToCityView = (city: any) => {
  router.push({
    name: "cityView",
    params: { state: city.state, city: city.city },
    query: {
      id: city.id,
      lat: city.coords.lat,
      lng: city.coords.lng,
      type: city.type,
      range: city.range,
    },
  });
};
</script>
