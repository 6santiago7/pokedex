<template>
  <div id="app">
    <header class="header">
      <h1 class="title">Pokédex</h1>
      <input v-model="search" class="search-input" placeholder="Buscar Pokémon por nombre o número..." />
      <button class="add-pokemon-button" @click="addRandomPokemon">Agregar Pokémon Aleatorio</button>
    </header>
    <div v-if="loading" class="loading">Cargando...</div>
    <div v-if="!loading" class="pokemon-container">
      <div v-if="showSinglePokemon" class="pokemon-card">
        <img :src="getPokemonImageUrl(singlePokemon.id)" alt="Pokémon image" class="pokemon-image" />
        <div class="pokemon-info">
          <h2 class="pokemon-name">#{{ singlePokemon.id }} - {{ singlePokemon.name }}</h2>
          <div class="pokemon-details">
            <h3 class="details-heading">Habilidades:</h3>
            <ul class="details-list">
              <li v-for="ability in singlePokemon.abilities" :key="ability.name" class="details-item">{{ ability.name }}</li>
            </ul>
            <h3 class="details-heading">Estadísticas:</h3>
            <ul class="details-list">
              <li v-for="stat in singlePokemon.stats" :key="stat.stat.name" class="details-item">
                {{ stat.stat.name }}: <span class="stat-value">{{ stat.base_stat }}</span>
                <div class="progress-bar-container">
                  <div class="progress-bar" :style="{ width: getProgressWidth(stat.base_stat) + '%' }"></div>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </div>
      <div v-else>
        <div v-if="filteredPokemons.length === 0" class="no-results">
          <img src="https://as01.epimg.net/epik/imagenes/2018/11/16/portada/1542384053_864693_1542384302_noticia_normal.jpg" alt="Pokémon no encontrado" />
          <p>No se encontró ningún Pokémon. Intenta con otro nombre o número.</p>
        </div>
        <div v-else>
          <div v-for="pokemon in filteredPokemons" :key="pokemon.id" class="pokemon-card">
            <img :src="getPokemonImageUrl(pokemon.id)" alt="Pokémon image" class="pokemon-image" />
            <div class="pokemon-info">
              <h2 class="pokemon-name">#{{ pokemon.id }} - {{ pokemon.name }}</h2>
              <div class="pokemon-details">
                <h3 class="details-heading">Habilidades:</h3>
                <ul class="details-list">
                  <li v-for="ability in pokemon.abilities" :key="ability.name" class="details-item">{{ ability.name }}</li>
                </ul>
                <h3 class="details-heading">Estadísticas:</h3>
                <ul class="details-list">
                  <li v-for="stat in pokemon.stats" :key="stat.stat.name" class="details-item">
                    {{ stat.stat.name }}: <span class="stat-value">{{ stat.base_stat }}</span>
                    <div class="progress-bar-container">
                      <div class="progress-bar" :style="{ width: getProgressWidth(stat.base_stat) + '%' }"></div>
                    </div>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      pokemons: [],
      search: '',
      loading: true,
      singlePokemon: null,
      showSinglePokemon: false
    };
  },
  computed: {
    filteredPokemons() {
      const query = this.search.toLowerCase();
      return this.pokemons.filter(pokemon => {
        return pokemon.name.toLowerCase().includes(query) || pokemon.id.toString().includes(query);
      });
    }
  },
  methods: {
    getPokemonImageUrl(id) {
      return `https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/other/official-artwork/${id}.png`;
    },
    async fetchPokemonDetails(pokemon) {
      try {
        const response = await axios.get(pokemon.url);
        return {
          ...pokemon,
          ...response.data,
          abilities: response.data.abilities.map(ability => ({
            name: ability.ability.name
          })),
          stats: response.data.stats.map(stat => ({
            stat: stat.stat,
            base_stat: stat.base_stat
          }))
        };
      } catch (error) {
        console.error('Error fetching Pokémon details:', error);
        return pokemon;
      }
    },
    getProgressWidth(stat) {
      const maxStat = 200;
      return (stat / maxStat) * 100;
    },
    async addRandomPokemon() {
      const randomId = Math.floor(Math.random() * 250) + 1;
      try {
        const response = await axios.get(`https://pokeapi.co/api/v2/pokemon/${randomId}`);
        const randomPokemon = await this.fetchPokemonDetails({
          name: response.data.name,
          url: `https://pokeapi.co/api/v2/pokemon/${randomId}`,
          id: randomId
        });
        this.singlePokemon = randomPokemon;
        this.showSinglePokemon = true;
      } catch (error) {
        console.error('Error fetching random Pokémon:', error);
      }
    }
  },
  async created() {
    try {
      const response = await axios.get('https://pokeapi.co/api/v2/pokemon?limit=250');
      const detailedPokemons = await Promise.all(
        response.data.results.map(async (pokemon, index) => {
          return this.fetchPokemonDetails({ ...pokemon, id: index + 1 });
        })
      );
      this.pokemons = detailedPokemons;
      this.loading = false;
    } catch (error) {
      console.error('Error fetching Pokémon:', error);
    }
  }
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
  background: #f0f8ff;
  font-family: 'Arial', sans-serif;
}

.header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  background: #ffffff;
  padding: 10px 20px;
  border-bottom: 2px solid #ffcb05;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.title {
  color: #ffcb05;
  font-size: 2.5em;
  margin: 0;
  font-family: Georgia, 'Times New Roman', Times, serif;
}

.search-input {
  padding: 10px;
  font-size: 16px;
  border: 2px solid #ffcb05;
  border-radius: 5px;
  margin-left: 20px;
  width: 300px;
  background-color: white;
  color: black;
  font-family: 'Times New Roman', Times, serif;
}

.add-pokemon-button {
  padding: 10px 20px;
  background-color: #ffcb05;
  border: none;
  border-radius: 5px;
  font-size: 16px;
  color: white;
  cursor: pointer;
  margin-right: 42px;
}

.add-pokemon-button:hover {
  background-color: #f0a500;
}

#app {
  text-align: center;
  color: #2c3e50;
  margin-top: 80px;
  min-height: 100vh;
}

.loading {
  font-size: 1.2em;
  color: #ffcb05;
}

.pokemon-container {
  display: grid;
  gap: 20px;
  justify-items: center;
}

.pokemon-card {
  border: 2px solid #ffcb05;
  border-radius: 10px;
  padding: 15px;
  margin: 10px;
  width: 220px;
  background: #ffffff;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s, box-shadow 0.3s;
}

.pokemon-card:hover {
  transform: scale(1.05);
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.2);
}

.pokemon-image {
  width: 150px;
  height: 150px;
  border-radius: 10px;
}

.pokemon-info {
  text-align: left;
}

.pokemon-name {
  color: #ffcb05;
  font-size: 1.3em;
  margin-bottom: 10px;
  font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
}

.pokemon-details {
  color: black;
  font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
}

.details-heading {
  color: #ffcb05;
  font-size: 1.2em;
  margin-bottom: 5px;
  font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
}

.details-list {
  list-style-type: none;
  padding: 0;
}

.details-item {
  margin-bottom: 5px;
}

.stat-value {
  color: black;
  font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
}

.progress-bar-container {
  width: 100%;
  background-color: #e0e0e0;
  border-radius: 5px;
  margin-top: 5px;
  height: 10px;
}

.progress-bar {
  height: 100%;
  background-color: #ffcb05;
  border-radius: 5px;
}

.no-results {
  text-align: center;
  margin-top: 50px;
}

.no-results img {
  width: 300px;
  height: auto;
  margin-bottom: 20px;
}

.no-results p {
  font-size: 1.5em;
  color: #ff0000;
  font-family: Impact, Haettenschweiler, 'Arial Narrow Bold', sans-serif;
}
</style>
