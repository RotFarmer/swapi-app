<template>
  <div :class="{ light: !isDark, dark: isDark }">
    <h1 class="title">Star Wars Characters</h1>
    <div class="pagination" v-if="!loading">
      <button :disabled="prevUrl === null" @click="getPeople(prevUrl)" :class="{ 'disabled': prevUrl === null }">Previous
        Page</button>
      <ul class="page-list">
        <li v-for="page in pages" :key="page" :class="{ active: page === currentPage }" class="page-button-container">
          <button v-if="page" class="page-button" @click="getPeople(`https://swapi.dev/api/people/?page=${page}`)">{{ page
          }}</button>
          <span v-else class="ellipsis">&hellip;</span>
        </li>
      </ul>
      <button :disabled="nextUrl === null" @click="getPeople(nextUrl)" :class="{ 'disabled': nextUrl === null }">Next
        Page</button>
    </div>
    <ul v-if="!loading" class="fade-in">
      <li v-for="person in people" :key="person.url" class="person" :class="{ lighttile: !isDark, darktile: isDark }">
        <div @click="$router.push(`/person/${person.name}`)" class="person-link">
          <h2 class="name">{{ person.name }}</h2>
          <p class="gender">Gender: {{ person.gender }}</p>
          <p class="birth-year">Birth Year: {{ person.birth_year }}</p>
          <p class="vehicles" v-if="person.vehicles?.length">Total Number of Vehicles: {{ person.vehicles.length }}</p>
          <p class="vehicles" v-if="person.starships?.length">Total Number of Starships: {{ person.starships.length }}</p>
        </div>
      </li>
    </ul>
    <div v-if="loading" class="loading-overlay">
      <StarWarsLoadingSpinner message="May the force be with you..." />
    </div>
    <div class="pagination" v-if="!loading">
      <button :disabled="prevUrl === null" @click="getPeople(prevUrl)" :class="{ 'disabled': prevUrl === null }">Previous
        Page</button>
      <ul class="page-list">
        <li v-for="page in pages" :key="page" :class="{ active: page === currentPage }" class="page-button-container">
          <button v-if="page" class="page-button" @click="getPeople(`https://swapi.dev/api/people/?page=${page}`)">{{ page
          }}</button>
          <span v-else class="ellipsis">&hellip;</span>
        </li>
      </ul>
      <button :disabled="nextUrl === null" @click="getPeople(nextUrl)" :class="{ 'disabled': nextUrl === null }">Next
        Page</button>
    </div>
  </div>
</template>

<script lang="ts">
import { reactive, onMounted } from "vue";
import axios from "axios";
import { useDark, useToggle } from "@vueuse/core";
import StarWarsLoadingSpinner from '../StarWarsLoader/StarWarsLoader.vue';

export default {
  name: "PersonList",
  components:{
    StarWarsLoadingSpinner
  },
  data() {
    return {
      people: [],
      currentPage: 1,
      pages: [],
      prevUrl: null,
      nextUrl: null,
      totalPages: 0,
      loading: false,
      isDark: useDark()
    };
  },
  async mounted() {
    this.loading = true;
    await this.getPeople("https://swapi.dev/api/people/");
    this.loading = false;
  },
  methods: {
    async getPeople(url: string) {
      try {
        this.loading = true;
        const response = await axios.get(url);
        this.people = response.data.results;
        this.prevUrl = response.data.previous;
        this.nextUrl = response.data.next;
        this.totalPages = Math.ceil(response.data.count / 10);
        this.pages = this.getPages();
        this.currentPage = this.getCurrentPage(url);
        this.loading = false;
      } catch (error) {
        console.log(error);
        this.loading = false;
      }
    },
    getPages() {
      const pages = [];
      for (let i = 1; i <= this.totalPages; i++) {
        pages.push(i);
      }
      return pages;
    },
    getCurrentPage(url: string) {
      const page = Number(url.match(/page=(\d+)/)?.[1]);
      return page || 1;
    },
  },
};
</script>

<style scoped>
.loading-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 9999;
  opacity: 0;
  animation: fade-in 0.5s ease forwards;
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}


.title {
  font-size: 2.5rem;
  margin-bottom: 1rem;
  text-align: center;
  color: #FFE81F;
  text-shadow: 1px 1px #000;
}

.person {
  margin-bottom: 1rem;
  margin-left: 0%;
  padding: 1rem;
  border: 1px solid #FFE81F;
  border-radius: 3px;
  background-color: #333438;
  list-style-type: none;
}

.name {
  margin-top: 0;
  margin-bottom: 0.5rem;
  font-size: 1.5rem;
  text-shadow: 1px 1px #000;
}

.gender,
.birth-year,
.vehicles {
  margin-top: 0;
  margin-bottom: 0.5rem;
}

.pagination {
  display: flex;
  justify-content: space-between;
  margin-top: 1rem;
  margin-bottom: 1rem;
}

.pagination button {
  margin-right: 0.5rem;
  padding: 0.25rem 0.5rem;
  font-size: 1rem;
  border: 1px solid #FFE81F;
  border-radius: 3px;
  background-color: #333438;
  color: #FFE81F;
  cursor: pointer;
}

.pagination .active button {
  background-color: #FFE81F;
  color: #333438;
  cursor: pointer;
}

.pagination span {
  margin-right: 0.5rem;
  padding: 0.25rem 0.5rem;
  font-size: 1rem;
  border: none;
  background-color: transparent;
  color: #FFE81F;
}

.page-list {
  display: flex;
  list-style: none;
  padding: 0;
  margin: 0;
}

.page-button {
  margin-right: 0.5rem;

}

.ellipsis {
  margin-right: 0.5rem;
}

ul {
  padding: 0;
  list-style-type: none;
}

.page-button-container {
  border: none;
}

.dark {
  background: #16171d;
  color: #FFE81F;
}

.darktile {
  background: #333438;
  color: #FFE81F;
}

.lighttile {
  background: #FFE81F;
  color: #333438;
}

.person-link {
  cursor: pointer;
}

.disabled {
  background: #c3c5cc !important;
  color: #272525 !important;
  cursor: default !important;
}

button:hover {
  background-color: #FFE81F;
  color: #333438;
  border-radius: 0.25rem;
}

.lighttile:hover{
  background-color: #333438;
  color: #FFE81F;
  border-radius: 0.25rem;
}
.darktile:hover{
  color: #333438;
  background-color: #FFE81F;
  border-radius: 0.25rem;
}
</style>
