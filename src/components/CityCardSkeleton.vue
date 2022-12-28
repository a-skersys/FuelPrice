<template>
  <div class="flex py-6 px-3 min-h-35 bg-fuel-secondary rounded-md shadow-md">
    <div class="flex flex-row flex-1">
      <div class="w-3/4 ml-2">
        <h2 class="text-3x1 text-xl"><AnimatedPlaceholder class="w-1/2" /></h2>
        <h3 class="text-sm pt-3">
          <AnimatedPlaceholder class="w-1/3" />
        </h3>
      </div>
      <div class="w-1/4 flex flex-col items-end justify-center text-xl">
        <AnimatedPlaceholder class="w-20 mr-3" />
        <AnimatedPlaceholder class="w-20 mr-3 mt-3" />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import AnimatedPlaceholder from "../components/AnimatedPlaceholder.vue";
</script>
