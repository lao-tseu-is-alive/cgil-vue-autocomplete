<!-- eslint-disable max-len,no-lonely-if -->
<template>
  <div>
    <svg xmlns="http://www.w3.org/2000/svg" style="display:none">
      <symbol xmlns="http://www.w3.org/2000/svg" id="sbx-icon-search-6" viewBox="0 0 40 40">
        <path
          d="M28.295 32.517c-2.93 2.086-6.51 3.312-10.38 3.312C8.02 35.83 0 27.81 0 17.914 0 8.02 8.02 0 17.915 0 27.81 0 35.83 8.02 35.83 17.915c0 3.87-1.227 7.45-3.313 10.38l6.61 6.61c1.166 1.165 1.163 3.057 0 4.22-1.167 1.167-3.057 1.167-4.226-.002l-6.605-6.606zm-10.38.326c8.245 0 14.928-6.683 14.928-14.928 0-8.245-6.683-14.93-14.928-14.93-8.245 0-14.93 6.685-14.93 14.93 0 8.245 6.685 14.928 14.93 14.928zm0-26.573c-6.43 0-11.645 5.214-11.645 11.645 0 .494.4.895.896.895.495 0 .896-.4.896-.895 0-5.442 4.41-9.853 9.853-9.853.494 0 .895-.4.895-.896 0-.495-.4-.896-.895-.896z"
          fill-rule="evenodd"></path>
      </symbol>
      <symbol xmlns="http://www.w3.org/2000/svg" id="sbx-icon-clear-5" viewBox="0 0 20 20">
        <path
          d="M10 20c5.523 0 10-4.477 10-10S15.523 0 10 0 0 4.477 0 10s4.477 10 10 10zm1.35-10.123l3.567 3.568-1.225 1.226-3.57-3.568-3.567 3.57-1.226-1.227 3.568-3.568-3.57-3.57 1.227-1.224 3.568 3.568 3.57-3.567 1.224 1.225-3.568 3.57zM10 18.272c4.568 0 8.272-3.704 8.272-8.272S14.568 1.728 10 1.728 1.728 5.432 1.728 10 5.432 18.272 10 18.272z"
          fill-rule="evenodd"></path>
      </symbol>
    </svg>
    <div id='formFeedBack' class="alert alert-danger" v-if="isError">
      <strong>{{errMsg}}</strong>
    </div>
    <form novalidate="novalidate" onsubmit="return false;" class="searchbox sbx-custom">
      <div role="search" class="sbx-google__wrapper">
        <input type="search" name="search" ref="search" v-model='searchText' @keydown.enter="enter"
               @keydown.tab="close" @keydown.up="up"
               @keydown.down="down" @keydown.esc="close" @keyup='update'
               @focus="gotFocus()"
               class="sbx-custom__input"
               v-bind:class="{isloading : isAjaxCallRunning}"
               :placeholder="placeholder"
               autocomplete="off" required="required">
        <button type="submit" title="Lancer votre recherche." class="sbx-custom__submit">
          <svg role="img" aria-label="Search">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#sbx-icon-search-6"></use>
          </svg>
        </button>
        <button type="reset" @click="reset()" title="Effacer les critères de votre recherche"
                class="sbx-custom__reset">
          <svg role="img" aria-label="Reset">
            <use xmlns:xlink="http://www.w3.org/1999/xlink" xlink:href="#sbx-icon-clear-5"></use>
          </svg>
        </button>
        <ul v-if="showSugestions" ref='suggestion' class="suggestion" tabindex="0">
          <template v-for="(result, key) in arrResults">
            <li :key="key" :data="result.id" @click.prevent="itemSelected(result, $event)"
                class="suggestion-item"
                :class="{'suggestionitem__selected' : isSelected(key) }">{{ result[labelFieldName]
              }}
            </li>
          </template>
        </ul>
      </div>
    </form>
  </div>
</template>

<script>
import axios from 'axios';
import { isNullOrUndefined, debounce } from 'cgil-html-utils/src/cgHtmlUtils';
import Log from 'cgil-log';

const log = new Log('', 2); // for now limit to warning and error
const debounceDelay = 350; // in ms
const minChars = 2;
// const cache = {}
axios.defaults.timeout = 4000;
export default {
  name: 'cgilVueAutoComplete',
  components: {}, // end of components section
  data: () => ({
    searchText: '',
    previousSearch: '',
    selectedIndex: null,
    isError: false,
    errMsg: '',
    selected: '',
    isAjaxCallRunning: false,
    showSugestions: false,
    eventListener: null, // pour detecter click en dehor suggestion
    arrAxiosSource: [],
    arrAjaxResults: [], // when isAjaxDataSourceWithFilter is false we keep data here
    arrResults: [{
      id: 0,
      label: 'aucun résultats trouvés...',

    }],
  }), // end of data section
  props: {
    placeholder: {
      type: String,
      default: '',
    },
    idFieldName: {
      type: String,
      default: 'id',
    },
    labelFieldName: {
      type: String,
      default: 'label',
    },
    ajaxDataSource: {
      type: String,
      default: '',
    },
    isAjaxDataSourceWithFilter: {
      type: Boolean,
      default: true,
    },
  }, // end of props section
  watch: {
    searchText(val) {
      const search = val.trim();
      // if (search !== this.selected) {
      // new search begins so reset selected
      // this.selected = ''
      // this.selectedIndex = null
      // }
      if (this.isAjaxDataSourceWithFilter) {
        if (search.length < minChars) {
          this.arrResults = [];
        }
        if (search !== this.previousSearch && this.selected.length < 1) {
          this.previousSearch = search;
          if (search.length > minChars) {
            this.getData(search);
          }
        } else {
          // probably user pressed space
          // eslint-disable-next-line
          if (!this.showSugestions && (this.arrResults.length > 1)) {
            this.showSugestions = true;
          }
        }
      } else {
        // here isAjaxDataSourceWithFilter = false
        // so we are only filtering existing data with search criteria
        // eslint-disable-next-line
        if (search !== this.previousSearch) {
          this.previousSearch = search;
          const searchNormalized = search.normalize('NFD')
            .replace(/[\u0300-\u036f]/g, '')
            .toLowerCase();
          this.arrResults = this.arrAjaxResults.filter((o) => o[`${this.labelFieldName}_normalized`].includes(searchNormalized));
        } else {
          // nothing
        }
      }
    },
  }, // end of watch section
  created() {
    log.t(`### created src: ${this.ajaxDataSource} , with filter? ${this.isAjaxDataSourceWithFilter}`);
    if (!this.isAjaxDataSourceWithFilter) {
      // we can load once the data from remote server and forget about it
      this.getData();
    }
  }, // end of created section
  mounted() {
    log.t(`### mounted src: ${this.ajaxDataSource} , with filter? ${this.isAjaxDataSourceWithFilter}`);
  }, // end of created section
  methods: {
    getData: debounce(function (query = '') {
      const that = this;
      log.l(`## In getData debounce CALLBACK : "${query}"`);
      if (that.arrAxiosSource.length > 0) {
        // eslint-disable-next-line
        that.arrAxiosSource.map((s) => {
          s.cancel();
        });
        that.arrAxiosSource = [];
      }
      const { CancelToken } = axios;
      const source = CancelToken.source();
      that.arrAxiosSource.push(source);
      that.arrResults = [];
      this.selectedIndex = null;
      that.isAjaxCallRunning = true;
      that.isError = false;
      axios.get(this.ajaxDataSource + query, {
        cancelToken: source.token,
      })
        .then((response) => {
          log.l(`#### AJAX CALL SUCCESS: "${query}" => num results = ${response.data.length}`);
          response.data.forEach((record) => {
            that.arrResults.push(record);
          });
          that.isAjaxCallRunning = false;
          if (that.arrResults.length > 0) {
            if (that.isAjaxDataSourceWithFilter) {
              that.showSugestions = true;
              that.setEventListener();
            } else {
              that.arrAjaxResults = that.arrResults.map((val) => {
                const o = {};
                const labelNormalized = val[that.labelFieldName].normalize('NFD')
                  .replace(/[\u0300-\u036f]/g, '')
                  .toLowerCase();
                o[`${that.labelFieldName}_normalized`] = labelNormalized;
                o[that.labelFieldName] = val[that.labelFieldName];
                o[that.idFieldName] = val[that.idFieldName];
                return o;
              });
            }
          }
          that.isError = false;
        })
        .catch((thrown) => {
          if (axios.isCancel(thrown)) {
            log.w(`#### AJAX CALL CANCELED : "${query}" : ${thrown.message}`);
          } else {
            log.e(`#### AJAX ERROR on GET  ${that.ajaxDataSource}${query}`, thrown);
            that.errMsg = `Un problème est survenu pendant l'appel réseau AJAX sur : ${that.ajaxDataSource}${query} Message d'erreur reçu : ${thrown.message}`;
            if (!window.navigator.onLine) {
              that.errMsg += 'EN CE MOMENT VOUS N\'AVEZ PAS DE RESEAU ! veuillez réessayez quand votre connexion sera rétablie ';
            }
            log.e(that.errMsg);
            that.$emit('errorajax', that.errMsg);
            that.isError = true;
          }
          that.isAjaxCallRunning = false;
        });
    }, debounceDelay), // end of getData
    update(evt) {
      log.t(`#Update Event ==> key: ${evt.key}, query: "${this.searchText}"`);
      // eslint-disable-next-line
      switch (evt.key) {
        case 'ArrowDown':
          // this.$refs.suggestion.focus()
      }
    },
    itemSelected(result, evt) {
      log.t('## itemSelected', result, evt);
      let tempObj = {};
      if (!isNullOrUndefined(result)) {
        tempObj = { ...result };
      } else {
        tempObj[this.idFieldName] = evt.srcElement.data;
        tempObj[this.labelFieldName] = evt.srcElement.innerText;
      }
      // Emit the selected value through the input event
      this.$emit('input', tempObj);
      this.searchText = tempObj[this.labelFieldName];
      this.previousSearch = tempObj[this.labelFieldName];
      // this.selected = tempObj[this.labelFieldName]
      this.selectedIndex = null;
      log.l(` data selected is : ${tempObj[this.idFieldName]}`);
      this.close();
    },
    isSelected(key) {
      return key === this.selectedIndex;
    },
    up() {
      if (this.showSugestions) {
        log.t(`## up  selectedIndex:${this.selectedIndex} hasVerticalScrollbar: ${this.hasVerticalScrollbar(this.$refs.suggestion)}`);
        if (this.selectedIndex === 0) {
          this.selectedIndex = this.arrResults.length - 1;
        } else {
          this.selectedIndex -= 1;
          if (this.hasVerticalScrollbar(this.$refs.suggestion)) {
            this.scrollVertical(this.$refs.suggestion, -1);
          }
        }
      }
    },
    down() {
      if (this.showSugestions) {
        log.t(`## down   selectedIndex:${this.selectedIndex} hasVerticalScrollbar: ${this.hasVerticalScrollbar(this.$refs.suggestion)}`);
        if (this.selectedIndex === null) {
          this.selectedIndex = 0;
          return;
        }
        if (this.selectedIndex === this.arrResults.length - 1) {
          this.selectedIndex = 0;
        } else {
          this.selectedIndex += 1;
          if (this.hasVerticalScrollbar(this.$refs.suggestion) && this.selectedIndex > 5) {
            this.scrollVertical(this.$refs.suggestion, 1);
          }
        }
      }
    },
    enter() {
      if (!isNullOrUndefined(this.selectedIndex)) {
        this.itemSelected(this.arrResults[this.selectedIndex]);
      }
    },
    gotFocus() {
      log.t('## gotFocus  ');
      if (!this.isAjaxDataSourceWithFilter) {
        this.showSugestions = true;
        this.setEventListener();
      }
    },
    close() {
      log.t('## close  ');
      this.showSugestions = false;
      this.arrResults = [];
      this.removeEventListener();
    },
    reset() {
      log.t('## reset  ');
      this.searchText = '';
      this.arrResults = [];
      // let tempObj = {}
      // reset the selected value through the input event
      this.$emit('input', null);
    },
    setEventListener() {
      log.t('## setEventListener  ');
      if (this.eventListener) {
        return false;
      }
      this.eventListener = true;
      document.addEventListener('click', this.clickOutsideListener, true);
      return true;
    },
    removeEventListener() {
      log.t('## removeEventListener  :');
      this.eventListener = false;
      document.removeEventListener('click', this.clickOutsideListener, true);
    },
    clickOutsideListener(evt) {
      const test = (evt.target !== this.$refs.suggestion) && (evt.target !== this.$refs.search);
      log.t('## clickOutsideListener  :', evt, evt.target, test);
      if (test) {
        this.close();
      }
    },
    hasVerticalScrollbar(el) {
      return el.scrollHeight > el.clientHeight;
    },
    scrollVertical(el, numLines) {
      log.t(`## scrollVertical  numLines: ${numLines}`);
      log.l(`element scrollTop before: ${el.scrollTop}`);
      const offsetY = el.children[0].clientHeight || 12;
      // eslint-disable-next-line
      el.scrollTop += (numLines * offsetY);
      log.l(`element scrollTop after : ${el.scrollTop}`);
    },
  }, // end of methods section
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

  input[type=text]::-ms-clear {
    display: none;
    width: 0;
    height: 0;
  }

  input[type=text]::-ms-reveal {
    display: none;
    width: 0;
    height: 0;
  }

  input[type=search]::-ms-clear {
    display: none;
    width: 0;
    height: 0;
  }

  input[type=search]::-ms-reveal {
    display: none;
    width: 0;
    height: 0;
  }

  .suggestion {
    margin: 0;
    padding: 0;
    list-style-type: none;
    z-index: 1000;
    position: absolute;
    max-height: 400px;
    overflow-y: auto;
    background: #fff;
    width: 100%;
    border: 1px solid #ccc;
    border-top: 0;
    color: #000;
    text-align: left;

  }

  .suggestion-item {
    cursor: pointer;
    list-style-type: none;
    padding-left: 10px;
  }

  .suggestion-item:hover {
    background: rgba(0, 180, 255, .075);
  }

  .suggestionitem__selected {
    background: rgba(0, 150, 255, .075);
    border: #3E82F7 solid 1px;
  }

  .isloading {
    background: white url("./ui-anim_basic_16x16.gif") center no-repeat;
    background-position-x: 80%;
  }

  /* pour l'apparence de la recherche */

  .sbx-google__wrapper {
    width: 100%;
    height: 100%;
  }

  .searchbox-demo {
    transition: all .5s linear;
    margin-bottom: 0;
    padding: 18px;
    width: 100%;
    text-align: center;
    box-sizing: border-box;
    margin-right: 12px;
    margin-bottom: 20px;
    border-radius: 3px;
    box-shadow: 0 0 5px #cdd4d7;
    background: #fff;
  }

  .sbx-custom {
    display: inline-block;
    position: relative;
    width: 100%;
    height: 41px;
    white-space: nowrap;
    box-sizing: border-box;
    font-size: 16px;
  }

  .sbx-custom__wrapper {
    width: 100%;
    height: 100%;
  }

  .sbx-custom__input {
    display: inline-block;
    -webkit-transition: box-shadow .4s ease, background .4s ease;
    transition: box-shadow .4s ease, background .4s ease;
    border: 0;
    border-radius: 6px;
    box-shadow: inset 0 0 0 1px #CCCCCC;
    background-color: #FFFFFF;
    padding: 0;
    padding-right: 83px;
    padding-left: 13px;
    width: 100%;
    height: 100%;
    vertical-align: middle;
    white-space: normal;
    font-size: inherit;
    -webkit-appearance: none;
    -moz-appearance: none;
    appearance: none;
  }

  .sbx-custom__input::-webkit-search-decoration,
  .sbx-custom__input::-webkit-search-cancel-button,
  .sbx-custom__input::-webkit-search-results-button,
  .sbx-custom__input::-webkit-search-results-decoration {
    display: none;
  }

  .sbx-custom__input:hover {
    box-shadow: inset 0 0 0 1px #b3b3b3;
  }

  .sbx-custom__input:focus,
  .sbx-custom__input:active {
    outline: 0;
    box-shadow: inset 0 0 0 1px #3E82F7;
    background-color: #FFFFFF;
  }

  .sbx-custom__input::-webkit-input-placeholder {
    color: #AAAAAA;
  }

  .sbx-custom__input::-moz-placeholder {
    color: #AAAAAA;
  }

  .sbx-custom__input:-ms-input-placeholder {
    color: #AAAAAA;
  }

  .sbx-custom__input::placeholder {
    color: #AAAAAA;
  }

  .sbx-custom__submit {
    position: absolute;
    top: 0;
    right: 0;
    left: inherit;
    margin: 0;
    border: 0;
    border-radius: 0 5px 5px 0;
    background-color: #337ab7;
    padding: 0;
    width: 50px;
    height: 100%;
    vertical-align: middle;
    text-align: center;
    font-size: inherit;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
  }

  .sbx-custom__submit::before {
    display: inline-block;
    margin-right: -4px;
    height: 100%;
    vertical-align: middle;
    content: '';
  }

  .sbx-custom__submit:hover,
  .sbx-custom__submit:active {
    cursor: pointer;
  }

  .sbx-custom__submit:focus {
    outline: 0;
  }

  .sbx-custom__submit svg {
    width: 21px;
    height: 21px;
    vertical-align: middle;
    fill: #FFFFFF;
  }

  .sbx-custom__reset {
    display: none;
    position: absolute;
    top: 8px;
    right: 58px;
    margin: 0;
    border: 0;
    background: none;
    cursor: pointer;
    padding: 0;
    font-size: inherit;
    -webkit-user-select: none;
    -moz-user-select: none;
    -ms-user-select: none;
    user-select: none;
    fill: rgba(0, 0, 0, 0.5);
  }

  .sbx-custom__reset:focus {
    outline: 0;
  }

  .sbx-custom__reset svg {
    display: block;
    margin: 4px;
    width: 17px;
    height: 17px;
  }

  .sbx-custom__input:valid ~ .sbx-custom__reset {
    display: block;
    -webkit-animation-name: sbx-reset-in;
    animation-name: sbx-reset-in;
    -webkit-animation-duration: .15s;
    animation-duration: .15s;
  }

  @-webkit-keyframes sbx-reset-in {
    0% {
      -webkit-transform: translate3d(-20%, 0, 0);
      transform: translate3d(-20%, 0, 0);
      opacity: 0;
    }
    100% {
      -webkit-transform: none;
      transform: none;
      opacity: 1;
    }
  }

  @keyframes sbx-reset-in {
    0% {
      -webkit-transform: translate3d(-20%, 0, 0);
      transform: translate3d(-20%, 0, 0);
      opacity: 0;
    }
    100% {
      -webkit-transform: none;
      transform: none;
      opacity: 1;
    }
  }
</style>
