# cgil-vue-autocomplete
A Vue.js Autocomplete Component to be used in Goeland

## Getting Started

Install `cgil-vue-autocomplete` in the shell

```bash
npm install cgil-vue-autocomplete --save
```

Then import the vue component and use it !

```javascript
import 'cgil-vue-autocomplete/dist/cgil-vue-autocomplete.css'
import cgilVueAutoComplete from 'cgil-vue-autocomplete'

const DEV = process.env.NODE_ENV === 'development'
const BASE_REST_API_URL = DEV ? 'https://mygolux.lausanne.ch/goapi/geo/' : '/goapi/geo/'

export default {
name: 'app',
components: {
  cgilVueAutoComplete
},
data: () => {
  return {
    adresse: null,
    pays: null,    
    geoCountryUrl: BASE_REST_API_URL + 'pays'
  }
}
```

and in your html template :

```html
<cgil-vue-auto-complete
  placeholder="choix du pays"
  :ajax-data-source="geoCountryUrl"
  :is-ajax-data-source-with-filter="false"
  id-field-name="value"
  v-model="pays"
>
</cgil-vue-auto-complete>
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