<script setup>
import { ref, onMounted, watch, computed } from 'vue';

const loading = ref(false);
const error = ref(null);
const index = ref(1);
const pokemon = ref(null);
const favorites = ref(JSON.parse(localStorage.getItem('pokemonFavorites') || '[]'));

watch(favorites, (newFavorites) => {
  localStorage.setItem('pokemonFavorites', JSON.stringify(newFavorites));
}, { deep: true });

const isFavorite = computed(() => {
  return pokemon.value && favorites.value.includes(pokemon.value.id);
});

const fetchPokemon = async () => {
  loading.value = true;
  error.value = null;
  try {
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${index.value}`);
    if (!res.ok) throw new Error('Pokemon not found');
    pokemon.value = await res.json();
  } catch (err) {
    error.value = err.message;
  } finally {
    loading.value = false;
  }
};

const toggleFavorite = () => {
  if (!pokemon.value) return;
  const pokemonId = pokemon.value.id;
  const currentIndex = favorites.value.indexOf(pokemonId);
  if (currentIndex === -1) {
    favorites.value.push(pokemonId);
  } else {
    favorites.value.splice(currentIndex, 1);
  }
};

const increaseIndex = () => {
  index.value++;
};

const decreaseIndex = () => {
  if (index.value > 1) index.value--;
};

onMounted(() => {
  fetchPokemon();
});

watch(index, () => {
  fetchPokemon();
});
</script>

<template>
  <div class="min-h-screen bg-gradient-to-br from-gray-900 to-gray-800 flex items-center justify-center p-4">
    <div class="bg-gray-800 rounded-3xl shadow-2xl p-8 w-96 border border-gray-700">
      <div v-if="loading" class="flex justify-center items-center h-64">
        <div class="w-8 h-8 border-4 border-blue-400 border-t-transparent rounded-full animate-spin"></div>
      </div>

      <div v-else-if="error" class="text-red-400 text-center p-4 bg-red-900/20 rounded-lg">
        {{ error }}
      </div>

      <div v-else-if="pokemon" class="flex flex-col items-center">
        <div class="relative w-full">
          <h2 class="text-4xl font-bold text-center mb-6 text-white capitalize">
            {{ pokemon.name }}
          </h2>
          <button
            @click="toggleFavorite"
            class="absolute right-0 top-0 p-2 transition-transform hover:scale-110"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="w-6 h-6"
              :class="isFavorite ? 'fill-red-500 text-red-500' : 'text-gray-400'"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
            >
              <path
                stroke-linecap="round"
                stroke-linejoin="round"
                d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"
              />
            </svg>
          </button>
        </div>

        <div class="relative group">
          <div class="w-48 h-48 flex items-center justify-center bg-gray-700/30 rounded-2xl mb-6 overflow-hidden transition-transform group-hover:scale-105">
            <img
              :src="pokemon.sprites.front_default"
              :alt="pokemon.name"
              class="w-full h-full object-contain"
            />
          </div>
        </div>

        <div class="flex gap-4 w-full justify-center">
          <button
            @click="decreaseIndex"
            :disabled="index <= 1"
            class="flex items-center gap-2 px-4 py-2 rounded-lg transition-all duration-200"
            :class="index <= 1 ? 'bg-gray-700 text-gray-500 cursor-not-allowed' : 'bg-blue-600 hover:bg-blue-500 text-white hover:shadow-lg hover:shadow-blue-500/20'"
          >
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="w-5 h-5"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <polyline points="15 18 9 12 15 6" />
            </svg>
            Prev
          </button>

          <button
            @click="increaseIndex"
            class="flex items-center gap-2 px-4 py-2 rounded-lg bg-blue-600 hover:bg-blue-500 text-white transition-all duration-200 hover:shadow-lg hover:shadow-blue-500/20"
          >
            Next
            <svg
              xmlns="http://www.w3.org/2000/svg"
              class="w-5 h-5"
              viewBox="0 0 24 24"
              fill="none"
              stroke="currentColor"
              stroke-width="2"
              stroke-linecap="round"
              stroke-linejoin="round"
            >
              <polyline points="9 18 15 12 9 6" />
            </svg>
          </button>
        </div>

        <div class="mt-6 w-full">
          <div class="grid grid-cols-2 gap-2 text-sm">
            <div
              v-for="type in pokemon.types"
              :key="type.type.name"
              class="bg-blue-500/20 text-blue-300 px-3 py-1 rounded-full text-center"
            >
              {{ type.type.name }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>