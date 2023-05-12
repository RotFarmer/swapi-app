<template>
  <div>
    <div v-if="loading" class="loading-overlay">
      <StarWarsLoadingSpinner message="May the force be with you..." />
    </div>
    <div v-else>
      <ul class="wide-list" :class="{ lighttile: !isDark, darktile: isDark }">
        <li v-for="person in people" :key="person.url">
          <h3 class="person-info-label">{{ person.name }}</h3>
          <div class="person-info">
            <div class="person-info-row">
              <div class="person-info-label">Birth Year:</div>
              <div class="person-info-value">{{ person.birth_year }}</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Eye Color:</div>
              <div class="person-info-value">{{ person.eye_color }}</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Gender:</div>
              <div class="person-info-value">{{ person.gender }}</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Hair Color:</div>
              <div class="person-info-value">{{ person.hair_color }}</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Height:</div>
              <div class="person-info-value">{{ person.height }}cm</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Mass:</div>
              <div class="person-info-value">{{ person.mass }}kg</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Skin Color:</div>
              <div class="person-info-value">{{ person.skin_color }}</div>
            </div>
            <h3 class="sub-list person-info-label" v-if="people[0].vehiclesData?.length">Vehicles:</h3>
        <li class="ship-link" v-for="ship in people[0].vehiclesData" :key="ship.url"
          @click="$router.push(`/vehicles/${ship.name}`)">
          <div class="person-info-row">
            <div class="person-info-label">{{ ship.name }}</div>
            <div class="person-info-value">{{ ship.model }}</div>
          </div>
        </li>
        <h3 class="sub-list person-info-label" v-if="people[0].starshipsData?.length">Starships:</h3>
        <li class="ship-link" v-for="ship in people[0].starshipsData" :key="ship.url"
          @click="$router.push(`/starships/${ship.name}`)">
          <div class="person-info-row">
            <div class="person-info-label">{{ ship.name }}</div>
            <div class="person-info-value">{{ ship.model }}</div>
          </div>
        </li>
        <h3 class="sub-list person-info-label" v-if="people[0].filmsData?.length">Films:</h3>
        <li v-for="film in people[0].filmsData" :key="film.url">
          <div class="person-info-row">
            <div class="person-info-label">{{ film.title }}</div>
          </div>
        </li>
    </div>
    <div class="person-info-row sub-list">
      <h3 class="person-info-label">Created Date:</h3>
      <div class="person-info-value">{{ person.created }}</div>
      <h3 class="person-info-label">Edited Date:</h3>
      <div class="person-info-value">{{ person.edited }}</div>
    </div>
    </li>
    </ul>
  </div>
  </div>
</template>

<script lang="ts">
import { useDark } from '@vueuse/core';
import { defineComponent } from 'vue';
import StarWarsLoadingSpinner from '../../components/StarWarsLoader/StarWarsLoader.vue';

export default defineComponent({
  components: {
    StarWarsLoadingSpinner
  },
  data() {
    return {
      people: [],
      loading: true,
      searchQuery: "",
      isDark: useDark()
    };
  },
  created() {
    // Extract the searchQuery from the URL
    const segments = this.$route.path.split("/");
    this.searchQuery = decodeURIComponent(segments[segments.length - 1]);
    // Make the API call to fetch person data
    fetch(`https://swapi.dev/api/people/?search=${this.searchQuery}`)
      .then((response) => response.json())
      .then((data) => {
        let person = data.results[0]
        // Fetch homeworld data
        fetch(`https://swapi.dev/api/planets/?search=${person.homeworld}`)
          .then((response) => response.json())
          .then((homeworldData) => {
            person.homeworldData = homeworldData;
            person.created = new Date(person.created).toLocaleDateString();
            person.edited = new Date(person.edited).toLocaleDateString();
            this.people = [person]; // Display the person data as soon as it arrives
            // Fetch film data
            const filmPromises = person.films.map((filmurl) => {
              return fetch(filmurl)
                .then((response) => response.json())
                .then((filmData) => {
                  return filmData;
                });
            });
            Promise.all(filmPromises).then((films) => {
              person.filmsData = films;
              // Fetch starship data
              const starshipPromises = person.starships.map((starshipUrl) => {
                return fetch(starshipUrl)
                  .then((response) => response.json())
                  .then((starshipData) => {
                    return starshipData;
                  });
              });
              Promise.all(starshipPromises).then((starships) => {
                person.starshipsData = starships;

                // Fetch vehicle data
                const vehiclePromises = person.vehicles.map((vehicleUrl) => {
                  return fetch(vehicleUrl)
                    .then((response) => response.json())
                    .then((vehicleData) => {
                      return vehicleData;
                    });
                });
                Promise.all(vehiclePromises).then((vehicles) => {
                  person.vehiclesData = vehicles;
                  this.people = [person]; // Update the person data with vehicle data
                  this.loading = false;
                });
              });
            })
          });
      });
  },
});
</script>


<style>
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

.person-info {
  display: flex;
  flex-direction: column;
  margin-left: 20px;
}

.person-info-row {
  display: flex;
  flex-direction: row;
  margin-top: 5px;
}

.person-info-label {
  font-weight: bold;
  width: 15rem;
}

.person-info-value {
  width: 15rem;
}

li {
  text-decoration: none;
  list-style: none;
}

.sub-list {
  margin-top: 1rem;
  margin-bottom: 1rem;
}

.wide-list {
  width: 100%;
}



.ship-link {
  cursor: pointer;
  border: rgb(119, 133, 133) solid 1px;
  border-radius: 0.25rem;
  margin-top: 0.25rem;
  margin-bottom: 0.25rem;
  padding-left: .5rem;
}

.ship-link:hover {
  cursor: pointer;
  border: rgb(119, 133, 133) solid 1px;
  border-radius: 0.25rem;
  margin-top: 0.25rem;
  margin-bottom: 0.25rem;
  padding-left: .5rem;
  background-color: rgb(119, 133, 133);
  color: #fff;
}

.dark {
  background: #16171d;
  color: #FFE81F;
}

.darktile {
  background: #333438;
  color: #FFE81F;
  border-radius: .5rem;
  padding: 1rem;
}

.lighttile {
  background: #FFE81F;
  color: #333438;
  border-radius: .5rem;
  padding: 1rem;
}
</style>
