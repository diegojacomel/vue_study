<template>
  <div class="search-bar">
    <input
      class="input-search"
      @input="(e) => fetchHeroes(e.target.value)"
      v-model="textTyped"
      placeholder="Busque um Herói"
    />
    <div v-if="isLoading">
      Carregando ...
    </div>
    <div class="card">
      <div
        v-for="(heroe, index) in heroes"
        v-bind:key="index"
        class="card-item"
      >
        <figure class="card-item-image">
          <img :src="heroe.thumbnail.path + '.' + heroe.thumbnail.extension" />
        </figure>
        <div class="crad-item-description">
          {{ heroe.name }}
        </div>
        <p v-if="heroe.description">
          {{ heroe.description }}
        </p>
        <p v-else>
          No descriptions
        </p>
        <div>
          <p>
            Movies:
          </p>
          <ul>
            <li v-for="(movie, movieIndex) in heroe.stories.items" v-bind:key="movieIndex">
              {{ movie.name }}
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import MarvelService from '../services/marvel'
import { privateKey, publicKey } from '../utils/keys'
import CryptoJS from 'crypto-js'
import { debounce } from 'lodash'

export default {
  name: 'SearchBar',
  data() {
    return {
      textTyped: '',
      heroes: []
    }
  },
  created: function() {
    this.isLoading = true
    this.fetchHeroes()
  },
  methods: {
    fetchHeroes: debounce(function(value) {
      this.isLoading = true;

      const timestamp = '1'
      const infoJoined = timestamp + privateKey + publicKey
      const hash = CryptoJS.MD5(infoJoined).toString(CryptoJS.enc.Hex)
      const offset = this.heroes.length;
      const limit = this.heroes.length + 33;

      MarvelService.getCharacters(timestamp, publicKey, hash, value ? value : '', limit, offset)
        .then(response => {
          if (response.data) {
            this.heroes = response.data.data.results
            this.isLoading = false
          }
        })
        .finally(() => {
          this.isLoading = false
        })
    }, 1000)
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.input-search {
  padding: 8px 15px;
  border: 1px solid #cccccc;
  font-size: 16px;
  border-radius: 3px;
  margin-bottom: 30px;
}

.card {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  align-items: stretch;
}

.card-item {
  flex-basis: calc(33.33% - 50px);
  margin-left: 20px;
  margin-bottom: 20px;
  border: 1px solid #cccccc;
  padding: 15px;
  border-radius: 3px;
}

.card-item:nth-child(3n + 1) {
  margin-left: 0;
}

.card-item-image {
  height: 200px;
  background: #333333;
  border-radius: 3px;
  overflow: hidden;
  margin: 0;
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}

.card-item-image > img {
  max-width: 100%;
}
</style>
