<template>
  <div id="app">
    <h1>Strings</h1>
    <div class="select-control"><button @click="selectLanguages('all')" class="select-all" type="button" name="button">All</button> <button @click="selectLanguages('none')" class="select-none" type="button" name="button">None</button></div>
    <div class="languages small">
      <label v-for="(language, code) in supportedLanguages" :key="`lang-${code}`" :class="code">
        <input class="language" type="checkbox" :value="code" v-model="selectedLanguages">{{ language }}
      </label>
    </div>
    <div class="input-container">
      <input ref="input" v-model="word" class="word-input" type="text" name="word" placeholder="Enter word" autocomplete="off">
      <div v-if="word" @click="clearInput" class="clear-input">×</div>
    </div>
    <div class="translation-cont" ref="translationCont">
      <div v-show="isLoading" class="loading"></div>
      <table class="output">
        <thead class="small">
          <tr>
            <th>Language</th>
            <th>Translation</th>
            <th>Characters</th>
          </tr>
        </thead>
        <!-- <tbody> -->
        <transition-group name="fade" tag="tbody">
        <tr v-for="(item, index) in sortOutput" :key="`translation-${index}`">
          <td>{{ item.lang }}<span class="lang-label">{{ item.code }}</span></td>
          <td><input @click="selectText" :class="item.code" class="translation" type="text" :value="item.translation" readonly></td>
          <td>{{ item.characterCount }}</td>
        </tr>
        </transition-group>
        <!-- </tbody> -->
      </table>
    </div>
  </div>
</template>

<script>
  import { debounce, difference, pull } from 'lodash'
  import supportedLanguages from '../static/languages.json'

  export default {
    name: 'app',
    data() {
      return {
        supportedLanguages,
        // selectedLanguages: ['nl','en','fr','de','it','pl','pt','ru','es','tr','vi','ar','zh','ja','ko','th'],
        selectedLanguages: ['fr','de','it','vi'],
        word: '',
        output: [],
        isLoading: false,
        loadingWidth: 0
      }
    },
    computed: {
      sortOutput: function() {
        return this.output.sort(function(a, b) {
          return b.characterCount - a.characterCount
        })
      }
    },
    watch: {
      selectedLanguages: function(newValue, oldValue) {
        // calculate change
        // console.log(`old: ${oldValue}`)
        // console.log(`new: ${newValue}`)
        this.handleSelectedLanguagesChange(newValue, oldValue)
        
        
        // this.isLoading = true
        // this.debouncedRun()
      },
      word: function(newValue, oldValue) {
        if (this.word != '') {
          this.isLoading = true
          if (newValue) this.debouncedRun(this.selectedLanguages)
        }
      }
    },
    methods: {
      getTranslation(code) {
        return fetch(
          `https://translate.yandex.net/api/v1.5/tr.json/translate?key=trnsl.1.1.20161205T032544Z.7e1492088f252553.93da07ad618b8fef174afcdf00b72efa811e0388&text=${this.word}&lang=en-${code}`
        )
        .then(response => response.json())
        .then(json => {
          return json.text[0]
        })
        .catch(error => console.log(error))
      },
      handleSelectedLanguagesChange(newValue, oldValue) {
        if (newValue.length < oldValue.length) {
          console.log(`You removed: ${difference(oldValue, newValue)}`)
          this.output = this.output.filter(lang => lang.code != difference(oldValue, newValue))
        } else if (newValue.length > oldValue.length) {
          console.log(`You added: ${difference(newValue, oldValue)}`)
          this.run(difference(newValue, oldValue))
        }
      },
      run(codes) {
        codes.forEach(lang => {
          this.getTranslation(lang).then(translation => {
            this.output.push({
              characterCount: translation.length,
              code: lang,
              lang: this.supportedLanguages[lang],
              translation: translation
            })
          })
        })
        this.isLoading = false
        console.log(this.output)
      },
      selectLanguages: function(which) {
        //var self = this;
        this.selectedLanguages = []
        this.output = []
        if (which == 'all') {
          // _.forEach(this.supportedLanguages, function(key, value) {
          //   self.selectedLanguages.push(value);
          // });
          for (var lang in this.supportedLanguages) {
            if (this.supportedLanguages.hasOwnProperty(lang)) {
              this.selectedLanguages.push(lang)
            }
          }
        }
        this.debouncedRun()
      },
      selectText: function(e) {
        console.log(e.target.value)
      },
      clearInput: function() {
        this.word = ''
        this.output = []
        this.isLoading = false
      }
    },
    created() {
      this.debouncedRun = debounce(this.run, 1000)
    },
    mounted() {
      this.$refs.input.focus()
    }
  }
</script>

<style lang='scss'>
  @import '../node_modules/html5-reset/assets/css/reset';
  @import 'assets/_fonts';

  @mixin block() {
    background: #fff;
    // border: 1px solid #ccc;
    box-shadow: 0 5px 10px rgba(0,0,0,0.05);
    &::-webkit-scrollbar {
      display: none;
    }
  }

  $light-gray: #e6e6e6;

  html { overflow: hidden; }

  body, input, textarea {
    font-family: 'Noto Sans', sans-serif;
    font-size: 14px;
    font-weight: 400;
    line-height: 1;
    background: #eaeaec;
    color: black;
  }

  #app {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
  }

  .small {
    font-size: 12px;
  }

  .controls {
    background: #555;
  }

  h1 {
    position: absolute;
    top: 0;
    left: 32px;
    font-size: 14px;
    line-height: 60px;
    padding-left: 30px;
    background: url('https://www.ericpitcock.com/static/img/e.svg') left center no-repeat;
  }

  .select-control {
    position: absolute;
    top: 67px;
    left: 20px;
    text-align: right;
    width: 200px;
    z-index: 1;
    button {
      border: none;
      background: none;
      font-size: 10px;
      padding: 0;
      margin-left: 5px;
      color: red;
      text-transform: uppercase;
      letter-spacing: 1px;
      &:hover {
        color: black;
      }
      &:focus {
        outline: none;
      }
    }
  }

  .languages {
    position: absolute;
    top: 60px;
    bottom: 30px;
    left: 30px;
    width: 200px;
    overflow: scroll;
    padding: 26px 30px 30px 30px;
    @include block();
    label {
      display: block;
      margin-bottom: 10px;
      &:hover {
      }
      input {
        margin-right: 5px;
        vertical-align: baseline;
        &:hover {
          cursor: pointer;
        }
        &:focus {
          outline: none;
        }
      }
    }
    .th {
      padding-bottom: 20px;
      border-bottom: 1px solid $light-gray;
      margin-bottom: 20px;
    }
  }

  .input-container {
    position: absolute;
    top: 60px;
    left: 260px;
    width: 100%;
    max-width: 450px;
    height: 60px;
    input.word-input {
      width: 100%;
      height: 100%;
      border: none;
      padding-left: 30px;
      line-height: 60px;
      @include block();
      transition: all .2s ease-in-out;
      -webkit-appearance: none;
      &::placeholder {
        color: black;
      }
      &:focus {
        outline: none;
        border-color: #999;
        box-shadow: 0px 6px 15px rgba(0,0,0,.1);
      }
    }
    .clear-input {
      position: absolute;
      top: 0;
      right: 0;
      width: 60px;
      height: 60px;
      cursor: pointer;
      font-size: 24px;
      line-height: 56px;
      text-align: center;
      color: #ccc;
      &:hover {
        color: red;
      }
    }
  }

  .translation-cont {
    position: absolute;
    top: 150px;
    right: 30px;
    bottom: 30px;
    left: 260px;
    max-width: 700px;
    overflow: scroll;
    @include block();
    .loading {
      position: absolute;
      top: 0;
      right: 0;
      bottom: 0;
      left: 0;
      z-index: 2;
      background: white url('/static/img/loading.svg') center center no-repeat;
    }
    table.output {
      width: 100%;
      margin: 30px 0;
      text-align: left;
      thead {
        text-transform: uppercase;
        letter-spacing: 1px;
        // color: blue;
        tr {
          border-bottom: 1px solid $light-gray;
        }
        th {
          width: 33.333%;
          padding-bottom: 10px;
        }
      }
      tbody {
        tr {
          border-bottom: 1px solid $light-gray;
          &:nth-child(odd) {
            background: #f9f9f9;
          }
          td {
            height: 40px;
            vertical-align: middle;
            span.lang-label {
              display: inline-block;
              font-size: 10px;
              text-transform: uppercase;
              letter-spacing: 1px;
              color: #999;
              padding-left: 5px;
            }
            input.translation {
              border: none;
              background: none;
              cursor: pointer;
              &:hover {
                color: red;
              }
            }
          }
        }
      }
      th:first-child,
      td:first-child {
        padding-left: 30px;
      }
    }
  }

  .fade-enter-active {
    transition: opacity .2s ease-in-out;
  }
  .fade-leave-active {
    transition: opacity .2s ease-in-out;
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>
