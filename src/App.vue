<template>
  <div id="app">
    <h4>Test de cgil-vue-autocomplete.vue</h4>
    <cgil-vue-auto-complete
      placeholder="choix du pays"
      :ajax-data-source="geoCountryUrl"
      :is-ajax-data-source-with-filter="false"
      id-field-name="id"
      v-model="pays"
    />
    <template v-if="pays !== null">
      <p>Cous avez choisi le pays {{pays}}</p>
    </template>
  </div>
</template>

<script>
import cgilVueAutoComplete from './components/cgil-vue-autocomplete.vue';

const BASE_REST_API_URL = 'https://golux.lausanne.ch/goapi/geo/';
export default {
  name: 'App',
  components: {
    cgilVueAutoComplete,
  },
  data: () => ({
    adresse: null,
    pays: null,
    geoCountryUrl: `${BASE_REST_API_URL}pays`,
  }),
};
</script>

<style lang="scss">
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
