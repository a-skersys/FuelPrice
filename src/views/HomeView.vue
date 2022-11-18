<template>
  <main class="container text-white">
    <!-- <template> -->
    <div class="pt-4 mb-8 relative">
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
          Sorry, something went wrong, please try again.
        </p>
        <p class="py-2" v-if="!searchError && mapboxSearchResults.length === 0">
          No results match your query, try a different term.
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
          min="0"
          max="25"
          v-model="fuelRange"
          step="5"
          class="w-4/5 h-1 bg-white rounded-lg appearance-none cursor-pointer"
        />
        <div class="w-1/5 min-w-fit text-center">{{ fuelRange }} km</div>
      </div>

      <p class="relative">
        <label class="block text-sm mt-4 mb-1" for="fuelType">Treibstoff</label>
        <RadioGroup v-model="fuelType">
          <div class="grid grid-cols-3 gap-4">
            <RadioGroupOption
              as="template"
              v-for="type in types"
              :key="type"
              :value="type"
              v-slot="{ active }"
            >
              <div
                :class="[
                  active
                    ? 'bg-fuel-secondary text-white ring-2 ring-fuel-secondary'
                    : 'text-gray-900',
                  'bg-white  cursor-pointer group relative border-fuel-primary rounded-md py-3 px-4 flex items-center justify-center hover:bg-fuel-secondary hover:border-fuel-primary hover:text-white focus:outline-none sm:flex-1',
                ]"
              >
                <RadioGroupLabel as="span" class="uppercase">{{
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
    <p class="text-xl">{{ fuelType }}, {{ fuelRange }}, {{ fuelPlace }}</p>
  </main>
</template>

<script setup lang="ts">
import { ref } from "vue";
import axios from "axios";
import VueAxios from "vue-axios";
import { RadioGroup, RadioGroupLabel, RadioGroupOption } from "@headlessui/vue";
import { useRouter } from "vue-router";
// import CityCardSkeleton from "../components/CityCardSkeleton.vue";
// import CityList from "../components/CityList.vue";

const types = ["e5", "e10", "diesel"];

const router = useRouter();
const previewCity = (chosenResult, fuelRange, fuelType) => {
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
      preview: true,
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
const mapboxSearchResults = ref(null);
const searchError = ref(false);

const fuelType = ref("");
const fuelRange = ref(5);
const chosenResult = ref(null);

const removeSearchResults = (searchResult) => {
  chosenResult.value = searchResult;
  mapboxSearchResults.value = null;
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
    mapboxSearchResults.value = null;
  }, 300);
};
</script>
