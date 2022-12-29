<template>
  <div
    class="flex py-4 px-4 min-h-35 bg-fuel-secondary rounded-md shadow-md cursor-pointer hover:shadow-lg hover:opacity-90"
  >
    <div class="flex flex-row flex-1">
      <div class="w-3/4">
        <h2 class="text-3x1 text-xl">
          <i class="fa-solid fa-location-dot pr-1"></i> {{ city.city }}
        </h2>
        <h3 class="text-sm pl-6">
          <span class="capitalize">{{ city.type }}</span> in {{ city.range }} km
          Umkreis
        </h3>
      </div>
      <div class="w-1/4 flex flex-col items-end text-md">
        <span class="text-fuel-primary"
          >{{ city.fuel.stations[0].price }} €</span
        >
        <span class="text-fuel-wrong"
          >{{ city.fuel.stations[city.fuel.stations.length - 1].price }} €</span
        >
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
defineProps({
  city: {
    type: Object,
    default: () => {},
  },
});
</script>
