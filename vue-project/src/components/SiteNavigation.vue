<template>
  <header class="sticky top-0 bg-gradient-to-r from-weather-primary to-weather-secondary shadow-lg">
    <nav class="container flex flex-col sm:flex-row items-center justify-between gap-4 text-white py-6">
      <RouterLink :to="{ name: 'home' }">
        <div class="flex items-center gap-3">
          <i class="fa-solid fa-sun text-4xl"></i>
          <p class="text-2xl font-bold tracking-wide text-white">Weather Realtime</p>
        </div>
      </RouterLink>

      <div class="flex gap-3 flex-1 justify-end">
        <RouterLink :to="{ name: 'Pert1' }">
          <button class="nav-button">Pertemuan 1</button>
        </RouterLink>
        <RouterLink :to="{ name: 'Pert2' }">
          <button class="nav-button">Pertemuan 2</button>
        </RouterLink>
        <RouterLink :to="{ name: 'Pert3' }">
          <button class="nav-button">Pertemuan 3</button>
        </RouterLink>
        <RouterLink :to="{ name: 'Pert4' }">
          <button class="nav-button">Pertemuan 4</button>
        </RouterLink>
        <RouterLink :to="{ name: 'Pert5' }">
          <button class="nav-button">Pertemuan 5</button>
        </RouterLink>
        <RouterLink :to="{ name: 'Pert6' }">
          <button class="nav-button">Pertemuan 6</button>
        </RouterLink>
        <i class="fa-solid fa-circle-info icon-button" @click="toggleModal"></i>
        <i class="fa-solid fa-plus icon-button" @click="addCity" v-if="route.query"></i>
      </div>

      <BaseModal :modalActive="modalActive" @close-modal="toggleModal">
        <div class="text-black">
          <h1 class="text-2xl mb-1">About:</h1>
          <p class="mb-4">
            The Local Weather allows you to track the current and future weather of cities of your choosing.
          </p>
          <h2 class="text-2xl">How it works:</h2>
          <ol class="list-decimal list-inside mb-4">
            <li>Search for your city by entering the name into the search bar.</li>
            <li>Select a city within the results, this will take you to the current weather for your selection.</li>
            <li>Track the city by clicking on the "+" icon in the top right. This will save the city to view at a later time on the home page.</li>
          </ol>

          <h2 class="text-2xl">Removing a city</h2>
          <p>
            If you no longer wish to track a city, simply select the city within the home page. At the bottom of the page, there will be an option to delete the city.
          </p>
        </div>
      </BaseModal>
    </nav>
  </header>
</template>

<script setup>
import { RouterLink, useRoute, useRouter } from "vue-router";
import { uid } from "uid";
import { ref } from "vue";
import BaseModal from "./BaseModal.vue";

const savedCities = ref([]);
const route = useRoute();
const router = useRouter();

const addCity = () => {
  if (localStorage.getItem("savedCities")) {
    savedCities.value = JSON.parse(localStorage.getItem("savedCities"));
  }

  const locationObj = {
    id: uid(),
    state: route.params.state,
    city: route.params.city,
    coords: {
      lat: route.query.lat,
      lng: route.query.lng,
    },
  };

  savedCities.value.push(locationObj);
  localStorage.setItem("savedCities", JSON.stringify(savedCities.value));

  let query = { ...route.query };
  delete query.preview;
  query.id = locationObj.id;
  router.replace({ query });
};

const modalActive = ref(false);

const toggleModal = () => {
  modalActive.value = !modalActive.value;
};
</script>

<style scoped>
nav.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

.fa-sun {
  color: #FFD700;
  animation: rotate 6s linear infinite;
}

@keyframes rotate {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}

.nav-button {
  background-color: rgba(255, 255, 255, 0.1);
  border: none;
  padding: 10px 20px;
  border-radius: 5px;
  transition: background-color 0.3s ease, transform 0.3s ease;
}

.nav-button:hover {
  background-color: rgba(255, 255, 255, 0.2);
  transform: scale(1.05);
}

.icon-button {
  color: #fff;
  transition: color 0.2s ease, transform 0.2s ease;
}

.icon-button:hover {
  color: #FFD700;
  transform: scale(1.2);
}

.BaseModal {
  z-index: 999;
  background-color: rgba(0, 0, 0, 0.75);
  padding: 20px;
  border-radius: 10px;
}

.text-2xl {
  font-size: 2rem;
  line-height: 1.5;
  text-align: center;
  color: #fff;
}

.font-bold {
  font-weight: 700;
}

.tracking-wide {
  letter-spacing: 1px;
}

.bg-gradient-to-r {
  background-image: linear-gradient(to right, #A9A9A9, #2d2d2d);
}
</style>
