<script setup>
import { ref, watch } from 'vue'

const countryName = ref();
const nativeLongName = ref();
const flag = ref()
const currencyCode = ref();
const currencyName = ref();
const currencySymbol = ref();
const region = ref();
const capital = ref()
const area = ref();
const googleMapsUrl = ref();
const loading = ref(false)

const API = 'https://restcountries.com/v3.1/name/'

const currencies = {
  Colombia: 'COP',
  Venezuela: 'VES',
}

const DEFAULT = 'COP'

async function getItems(param){
  try {
    loading.value = true
    const res = await fetch(API + param)
    const [json] = await res.json()

    countryName.value = json?.name?.common
    nativeLongName.value = json?.nativeLongName?.spa?.official
    flag.value = json?.flag
    currencyName.value = json.currencies[currencies[param] ?? DEFAULT].name
    currencySymbol.value = json.currencies[currencies[param] ?? DEFAULT].symbol
    region.value = json.region
    capital.value = json.capital.join(' ')
    area.value = json.area
    googleMapsUrl.value = json.maps.googleMaps /* "https://goo.gl/maps/zix9qNFX69E9yZ2M6" */
    
  } catch (error) {
    console.log(error);
  } finally {
    loading.value = false
  }
} 

function debounce(func, wait) {
  let timeout;

  return function(...args) {
    const context = this;
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(context, args), wait);
  };
}

const searchTerm = ref('Colombia');

const search = (newTerm) => {
  getItems(newTerm)
};

watch(searchTerm, debounce((newTerm) => {
  search(newTerm);
}, 500));

function onSubmit(ev){
  ev.preventDefault();
  getItems(searchTerm.value)
}
</script>

<template>
  <main>
    <div class="card">
      <div class="card-search">
        <form @submit="onSubmit">
          <input v-model="searchTerm" type="text" class="input">
        </form>
      </div>
      <div class="card-header">
        <h2>
          <span>{{ flag }}</span>
          {{ countryName }}</h2>
        <p>{{ nativeLongName }}</p>
      </div>
      <div class="card-body">
        <template v-if="loading" >
          Loading ...
        </template>
        <template v-else>
          <p class="card-currency"><strong>Currency:</strong> {{ currencyName }} ({{ currencySymbol }})</p>
          <p class="card-region"><strong>Region:</strong> {{ region }}</p>
          <p class="card-capital"><strong>Capital:</strong> {{ capital }}</p>
          <p class="card-area"><strong>Area:</strong> {{ area }} km²</p>
          <div class="map-container">
            <iframe
              :src="googleMapsEmbedUrl"
              width="100%"
              height="300"
              frameborder="0"
              style="border:0"
              allowfullscreen
            ></iframe>
          </div>
        </template>
      </div>
    </div>
  </main>
</template>

<style scoped>
main{
  display: grid;
  place-items: center; 
}

.card {
  width: 300px;
  background-color: white;
  border-radius: 10px;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  transition: transform 0.3s ease;
  margin: 20px;
}

.card-search{
  padding: 2rem 0;
}

.input{
  height: 40px;
  border: rgba(0, 0, 0, 0.2 ) 1px solid;
  border-radius: 4px;
  padding: 0.25rem 0.5rem;
  width: 100%;
}


.card-header {
  background-color: #007bff;
  color: white;
  padding: 20px;
  text-align: center;
  min-height: 76px;
}

.card-body {
  padding: 20px;
  min-height: 477px;
}

.map-container {
  margin-top: 15px;
}

.card-body p {
  margin: 5px 0;
}

.card-currency, .card-currency, .card-region, .card-capital, .card-area{
  color: black;
}

@media (min-width: 1024px) {
  .card {
    width: 700px;
  }


  .card-search{
    display: flex;
    justify-content: center;
  }

  .input{
    width: 400px;
  }
}

</style>