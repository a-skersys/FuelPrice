<template>
  <div
    class="flex py-6 px-3 bg-fuel-secondary rounded-md shadow-md cursor-pointer"
  >
    <div class="flex flex-row flex-1">
      <div class="w-3/4">
        <h2 class="text-3x1 text-xl">{{ city.city }}</h2>
        <h3 class="text-sm">
          <span class="capitalize">{{ city.type }}</span> in {{ city.range }} km
          Umkreis
        </h3>
      </div>
      <div class="w-1/4 flex flex-col items-center justify-center text-2xl">
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
