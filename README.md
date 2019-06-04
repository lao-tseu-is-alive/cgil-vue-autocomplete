# cgil-vue-autocomplete
A Vue.js Autocomplete Component to be used in Goeland

## Getting Started

Install `cgil-vue-autocomplete` in the shell

```bash
npm i -P cgil-vue-autocomplete 
```

Then import the vue component and use it as usual in your ES6 javascript code (with a bundler)
as explained here : https://cli.vuejs.org/guide/build-targets.html#app


Or directly inside an html page with the umd version of the lib

```html
<meta charset="utf-8">
<title>cgilVueAutoComplete demo</title>
<script src="https://unpkg.com/vue"></script>
<script src="./cgilVueAutoComplete.umd.min.js"></script>

<link rel="stylesheet" href="./cgilVueAutoComplete.css">


<div id="app">
  <demo placeholder="choix du pays"
        :ajax-data-source="geoCountryUrl"
        :is-ajax-data-source-with-filter="false"
        id-field-name="id"
        v-model="pays"
  />
  <template v-if="pays !== null">
    <p>Vous avez choisi le pays {{pays}}</p>
  </template>
</div>

<script>
new Vue({
  components: {
    demo: cgilVueAutoComplete
  },
  data: () => ({
    adresse: null,
    pays: null,
    geoCountryUrl: 'https://go.trouvl.info/api/geo/pays',
  }),
}).$mount('#app')
</script>
```




## Build Setup

``` bash
# clone this repository
git clone 

# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report

# run unit tests
npm run unit

# run all tests
npm test
```
