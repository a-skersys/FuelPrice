<template>
  <header class="sticky top-0 bg-fuel-primary shadow-lg z-10">
    <nav
      class="container flex flex-col sm:flex-row items-center gap-4 text-white py-6"
    >
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3">
          <p class="text-4xl font-light">TankenSpaß</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <i
          class="fa-solid fa-circle-info text-xl hover:text-fuel-secondary duration-150 cursor-pointer"
          @click="toggleModal"
        ></i>
        <i
          v-if="route.query.preview"
          class="fa-solid fa-circle-plus text-xl hover:text-fuel-secondary duration-150 cursor-pointer"
          @click="addCity"
        ></i>
        <i
          v-else
          class="fa-solid fa-trash text-xl hover:text-fuel-secondary duration-150 cursor-pointer"
          @click="removeCity"
        ></i>
      </div>

      <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
        <div class="text-black">
          <h1 class="text-2xl mb-1">Über:</h1>
          <p class="mb-4">
            Mit TankenSpaß können Sie die Kraftstoffpreise der Städte Ihrer Wahl
            verfolgen.
          </p>
          <h2 class="text-2xl">Wie es funktioniert:</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>
              Suchen Sie nach Ihrer Stadt, indem Sie ihren Namen in die
              Suchleiste eingeben und die richtige auswählen.
            </li>
            <li>
              Wählen Sie den Kraftstofftyp und die Entfernung vom
              Standortzentrum aus.
            </li>
            <li>
              Verfolgen Sie die Stadt, indem Sie oben rechts auf das
              <i class="fa-solid fa-circle-plus"></i>
              Symbol klicken. Dadurch wird die Stadt gespeichert, um sie zu
              einem späteren Zeitpunkt auf der Startseite anzuzeigen.
            </li>
          </ol>

          <h2 class="text-2xl">Eine Stadt entfernen</h2>
          <p>
            Wenn Sie eine Stadt nicht mehr verfolgen möchten, wählen Sie einfach
            die Stadt auf der Startseite aus. Oben auf der Seite wird anstelle
            des <i class="fa-solid fa-circle-plus"></i> Symbol ein
            <i class="fa-solid fa-trash"></i> Symbol angezeigt, um den Eintrag
            zu entfernen.
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup lang="ts">
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import { ref } from "vue";
import type { Ref } from "vue";
import BaseModal from "./BaseModal.vue";

const savedCities: Ref<any> = ref([]);
const route = useRoute();
const router = useRouter();
const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(
      localStorage.getItem("savedCities") as string
    );
  }

  const locationObj: any = {
    id: uid() as string,
    state: route.params.state as string,
    city: route.params.city as string,
    coords: {
      lat: route.query.lat as string,
      lng: route.query.lng as string,
    },
    type: route.query.type as string,
    range: route.query.range as string,
  };

  savedCities.value.push(locationObj);
  localStorage.setItem("savedCities", JSON.stringify(savedCities.value));

  let query = Object.assign({}, route.query);
  delete query.preview;
  query.id = locationObj.id;
  router.replace({ query });
};

const removeCity = () => {
  savedCities.value = JSON.parse(localStorage.getItem("savedCities") as string);
  const updatedCities = savedCities.value.filter(
    (city: any) => city.id !== route.query.id
  );
  localStorage.setItem("savedCities", JSON.stringify(updatedCities));
  router.push({
    name: "home",
  });
};

const modalActive = ref(false);
const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>
