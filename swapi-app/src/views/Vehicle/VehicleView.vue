<template>
  <div>
    <div v-if="loading" class="loading-overlay">
      <StarWarsLoadingSpinner message="May the force be with you..." />
    </div>
    <div v-else>
      <ul class="wide-list" :class="{ lighttile: !isDark, darktile: isDark }">
        <li v-for="vehicle in vehicles" :key="vehicle.url">
          <h3 class="person-info-label">{{ vehicle.name }}</h3>
          <div class="person-info">
            <div class="person-info-row">
              <div class="person-info-label">Model:</div>
              <div class="person-info-value">{{ vehicle.model }}</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Cost:</div>
              <div class="person-info-value">{{ vehicle.cost_in_credits }}</div>
            </div>
            <div class="person-info-row">
              <div class="person-info-label">Passengers:</div>
              <div class="person-info-value">{{ vehicle.passengers }}</div>
            </div>
            <div class="person-info-row" v-if="vehicle.filmsData?.length">
              <div class="person-info-label">Films:</div>
              <div class="person-info-value">
                <ul class="film-list">
                  <li v-for="film in vehicle.filmsData" :key="film.url">
                    {{ film.title }}
                  </li>
                </ul>
              </div>
            </div>
            <div class="person-info-row" v-if="vehicle.hyperdrive_rating">
              <div class="person-info-label">Hyperdrive Rating:</div>
              <div class="person-info-value">{{ vehicle.hyperdrive_rating }}</div>
            </div>
            <div class="person-info-row" v-if="vehicle.MGLT">
              <div class="person-info-label">MGLT:</div>
              <div class="person-info-value">{{ vehicle.MGLT }}</div>
            </div>
            <div class="person-info-row" v-if="vehicle.vehicle_class">
              <div class="person-info-label">Class:</div>
              <div class="person-info-value">{{ vehicle.vehicle_class }}</div>
            </div>
          </div>
          <div class="person-info-row">
            <h3 class="person-info-label">Created Date:</h3>
            <div class="person-info-value">{{ vehicle.created }}</div>
            <h3 class="person-info-label">Edited Date:</h3>
            <div class="person-info-value">{{ vehicle.edited }}</div>
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
      vehicles: [],
      loading: true,
      isDark: useDark()
    };
  },
  created() {
    // Extract the vehicle ID from the URL
    const segments = this.$route.path.split("/");
    const vehicleId = segments[segments.length - 1];
    const vehicleType = segments[segments.length - 2];
    // Make the API call to fetch vehicle data
    fetch(`https://swapi.dev/api/${vehicleType}/?search=${vehicleId}`)
      .then((response) => response.json())
      .then((data) => {
        data = data.results[0]
        const vehicle = {
          name: data.name,
          model: data.model,
          cost_in_credits: data.cost_in_credits,
          passengers: data.passengers,
          films: data.films,
          created: new Date(data.created).toLocaleDateString(),
          edited: new Date(data.edited).toLocaleDateString(),
          filmsData: [],
        };
        console.log(vehicle)
        if (vehicle.films?.length) {
          // Fetch film data
          const filmPromises = vehicle.films.map((filmurl) => {
            return fetch(filmurl)
              .then((response) => response.json())
              .then((filmData) => {
                return filmData;
              });
          });
          Promise.all(filmPromises).then((films) => {
            vehicle.filmsData = films;
            this.vehicles = [vehicle]; // Display the vehicle data as soon as it arrives
            this.loading = false;
          });
        }
        else {
          this.vehicles = [vehicle]; // Display the vehicle data as soon as it arrives
          this.loading = false;
        }
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



.film-list {
  list-style: none;
  padding-left: 0;
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

