<template>
  <div>
    <Suspense>
      <AsyncCityView />

      <template #fallback>
        <p class="text-white text-center p-2">Loading...</p>
      </template>
    </Suspense>
  </div>
</template>

<script setup lang="ts">
import AsyncCityView from "../components/AsyncCityView.vue";
</script>
