<style>
input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 18px;
  height: 18px;
  border-radius: 10px;
  background-color: rgb(38, 70, 83);
  overflow: visible;
  cursor: pointer;
}
</style>

<template>
  <main class="container max-w-2xl text-white">
    <!-- <template> -->
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList />

        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
      <div
        class="h-0.5 my-4 w-full bg-fuel-secondary opacity-30 lg:w-1/3"
      ></div>
    </div>
    <div class="pt-4 mb-8 relative">
      <p class="mb-4">Neuen Standort hinzufügen:</p>
      <p class="relative">
        <label class="block text-sm mb-1" for="place">PLZ/Ort</label>
        <input
          type="text"
          id="place"
          v-model="searchQuery"
          @input="getSearchResults"
          class="p-2 w-full text-black rounded-md bg-white border focus:border-fuel-secondary focus:outline-none focus: shadow-[0px_1px_0_0_#004E71]"
        />
      </p>

      <ul
        class="absolute bg-white text-fuel-primary w-full shadow-md top-[75px] rounded-b-md z-20"
        id="cityList"
        v-if="mapboxSearchResults"
      >
        <p class="py-2" v-if="searchError">
          Entschuldigung, etwas ist schief gelaufen, bitte versuchen Sie es
          erneut.
        </p>
        <p
          class="py-2"
          v-if="
            !searchError &&
            mapboxSearchResults.length === 0 &&
            searchQuery != '' &&
            chosenResult == ''
          "
        >
          Keine Ergebnisse stimmen mit Ihrer Suchanfrage überein, versuchen Sie
          es mit einem anderen Begriff.
        </p>
        <template v-else>
          <li
            v-for="searchResult in mapboxSearchResults"
            @click="removeSearchResults(searchResult)"
            :key="searchResult.id"
            class="py-2 px-2 cursor-pointer hover:bg-fuel-secondary hover:text-white"
          >
            {{ searchResult.place_name }}
          </li>
        </template>
      </ul>

      <label class="block text-sm mb-1 mt-4" for="range">Umkreis</label>
      <div class="flex items-center justify-center">
        <input
          id="range"
          type="range"
          min="1"
          max="25"
          v-model="fuelRange"
          step="1"
          class="w-4/5 h-1 bg-white rounded-lg appearance-none cursor-pointer"
        />
        <div class="w-1/5 min-w-fit text-center text-fuel-secondary font-bold">
          {{ fuelRange }} km
        </div>
      </div>

      <p class="relative">
        <label class="block text-sm mt-4 mb-1" for="fuelType">Treibstoff</label>
        <RadioGroup v-model="fuelType">
          <div class="grid grid-cols-3 gap-4">
            <RadioGroupOption
              v-for="type in types"
              :key="type"
              :value="type"
              v-slot="{ active }"
              as="template"
            >
              <div
                :class="[
                  active
                    ? 'bg-fuel-secondary text-white ring ring-fuel-secondary'
                    : 'text-gray-900',
                  'bg-white  cursor-pointer group relative border-fuel-primary rounded-md py-3 px-4 flex items-center justify-center hover:bg-fuel-secondary hover:border-fuel-primary hover:text-white focus:outline-none focus:text-gray-900 sm:flex-1 focus:hover:text-white',
                ]"
              >
                <RadioGroupLabel as="span" class="capitalize text-lg">{{
                  type
                }}</RadioGroupLabel>
              </div>
            </RadioGroupOption>
          </div>
        </RadioGroup>
      </p>
      <div
        @click="previewCity(chosenResult, fuelRange, fuelType)"
        class="bg-white w-full cursor-pointer text-black text-center py-3 my-5 rounded-md hover:bg-fuel-secondary hover:border-fuel-primary hover:text-white"
      >
        Suchen
      </div>
    </div>
  </main>
</template>

<script setup lang="ts">
import { ref } from "vue";
import type { Ref } from "vue";
import axios from "axios";
import { RadioGroup, RadioGroupLabel, RadioGroupOption } from "@headlessui/vue";
import { useRouter } from "vue-router";
import CityList from "../components/CityList.vue";
import CityCardSkeleton from "@/components/CityCardSkeleton.vue";

const types = ["e5", "e10", "diesel"];

const router = useRouter();
const previewCity = (
  chosenResult: any,
  fuelRange: number,
  fuelType: string
) => {
  const [city, state] = chosenResult.place_name.split(",");
  router.push({
    name: "cityView",
    params: {
      state: state.replaceAll(" ", ""),
      city: city,
    },
    query: {
      lat: chosenResult.geometry.coordinates[1],
      lng: chosenResult.geometry.coordinates[0],
      preview: true as any,
      range: fuelRange,
      type: fuelType,
    },
  });
};

const mapboxAPIKey =
  "pk.eyJ1IjoiYS1za2Vyc3lzIiwiYSI6ImNsOWczZTFmazBidHczd2syYnYwZDZueHEifQ.Oeld8g7GuQWldb-xOHh_bQ";
const fuelPlace = ref("");
const searchQuery = ref(fuelPlace);
const queryTimeout = ref(0);
const mapboxSearchResults: Ref<any> = ref([]);
const searchError = ref(false);

const fuelType = ref("");
const fuelRange = ref(5);
const chosenResult = ref(null);

const removeSearchResults = (searchResult: any) => {
  chosenResult.value = searchResult;
  mapboxSearchResults.value = [];
  fuelPlace.value = searchResult.place_name;
};

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== "") {
      try {
        const result = await axios.get(
          `https://api.mapbox.com/geocoding/v5/mapbox.places/${searchQuery.value}.json?access_token=${mapboxAPIKey}&types=place,postcode&country=DE&language=de`
        );
        mapboxSearchResults.value = result.data.features;
      } catch {
        searchError.value = true;
      }

      return;
    }
    mapboxSearchResults.value = [];
  }, 300);
};
</script>
