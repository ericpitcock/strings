<style lang='scss'>
  @import '../node_modules/html5-reset/assets/css/reset';
  @import 'assets/_fonts';

  @mixin block() {
    background: #fff;
    border: 1px solid #ccc;
    box-shadow: 0 1px 10px rgba(0,0,0,.1);
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
    background: #f6f6f6;
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
    background: url('http://www.ericpitcock.com/assets/img/e.svg') left center no-repeat;
  }

  .select-control {
    position: absolute;
    top: 60px;
    left: 30px;
    width: 200px;
    height: 30px;
    @include block();
    border-bottom-color: $light-gray;
    box-shadow: none;
    padding-left: 30px;
    z-index: 2;
    button {
      border: 1px solid $light-gray;
      background: none;
      font-size: 10px;
      color: red;
      letter-spacing: 1px;
    }
  }

  .languages {
    position: absolute;
    top: 90px;
    bottom: 30px;
    left: 30px;
    width: 200px;
    overflow: scroll;
    padding: 20px 30px 30px 30px;
    @include block();
    border-top: none;
    label {
      display: block;
      margin-bottom: 10px;
      &:hover {
        //color: red;
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
      &::placeholder {
        color: black;
      }
      &:focus {
        outline: none;
        border-color: #999;
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
      top: 10px;
      right: 10px;
    }
    table.output {
      width: 100%;
      margin: 30px 0;
      text-align: left;
      thead {
        text-transform: uppercase;
        letter-spacing: 1px;
        color: red;
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
    transition: opacity 1s ease-in-out;
  }
  .fade-leave-active {
    transition: opacity .3s ease-in-out;
  }
  .fade-enter, .fade-leave-to {
    opacity: 0;
  }
</style>

<template>
  <div id="app">
    <h1>Translate</h1>
    <div class="select-control"><button @click="selectLanguages('all')" class="select-all" type="button" name="button">ALL</button> <button @click="selectLanguages('none')" class="select-none" type="button" name="button">NONE</button></div>
    <div class="languages small">
      <label v-for="(language, index) in supportedLanguages" v-bind:class="index">
        <input class="language" type="checkbox" v-bind:value="index" v-model="selectedLanguages" @change="translate">{{ language }}
      </label>
    </div>
    <div class="input-container">
    <input ref="input" @keyup.enter="translate" v-model="word" class="word-input" type="text" name="word" placeholder="Enter word" autocomplete="off">
    <div @click="clearInput" class="clear-input">×</div>
    </div>
    <div class="translation-cont">
      <div class="loading" v-show="isLoading"><img src="static/img/loading.svg" /></div>
      <table class="output">
        <thead class="small">
          <tr>
            <th>Language</th>
            <th>Translation</th>
            <th>Characters</th>
          </tr>
        </thead>
        <tbody>
        <tr v-for="item in sortOutput" v-model="sortOutput">
          <td>{{ item.lang }}<span class="lang-label">{{ item.code }}</span></td>
          <td><input @click="selectText" v-bind:class="item.code" class="translation" type="text" v-bind:value="item.translation" readonly></td>
          <td>{{ item.characterCount }}</td>
        </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'app',
  data() {
    return {
      supportedLanguages: {
        "nl": "Dutch",
        "en": "English",
        "fr": "French",
        "de": "German",
        "it": "Italian",
        "pl": "Polish",
        "pt": "Portuguese",
        "ru": "Russian",
        "es": "Spanish",
        "tr": "Turkish",
        "vi": "Vietnamese",
        "ar": "Arabic",
        "zh": "Chinese",
        "ja": "Japanese",
        "ko": "Korean",
        "th": "Thai",
        "af": "Afrikaans",
        "sq": "Albanian",
        "am": "Amharic",
        "hy": "Armenian",
        "az": "Azerbaijan",
        "ba": "Bashkir",
        "eu": "Basque",
        "be": "Belarusian",
        "bn": "Bengali",
        "bs": "Bosnian",
        "bg": "Bulgarian",
        "ca": "Catalan",
        "ceb": "Cebuano",
        "hr": "Croatian",
        "cs": "Czech",
        "da": "Danish",
        "eo": "Esperanto",
        "et": "Estonian",
        "fi": "Finnish",
        "gl": "Galician",
        "ka": "Georgian",
        "el": "Greek",
        "gu": "Gujarati",
        "ht": "Haitian (Creole)",
        "he": "Hebrew",
        "mrj": "Hill Mari",
        "hi": "Hindi",
        "hu": "Hungarian",
        "is": "Icelandic",
        "id": "Indonesian",
        "ga": "Irish",
        "jv": "Javanese",
        "kn": "Kannada",
        "kk": "Kazakh",
        "ky": "Kyrgyz",
        "la": "Latin",
        "lv": "Latvian",
        "lt": "Lithuanian",
        "mk": "Macedonian",
        "mg": "Malagasy",
        "ms": "Malay",
        "ml": "Malayalam",
        "mt": "Maltese",
        "mi": "Maori",
        "mr": "Marathi",
        "mhr": "Mari",
        "mn": "Mongolian",
        "ne": "Nepali",
        "no": "Norwegian",
        "pap": "Papiamento",
        "fa": "Persian",
        "pa": "Punjabi",
        "ro": "Romanian",
        "gd": "Scottish Gaelic",
        "sr": "Serbian",
        "si": "Sinhala",
        "sk": "Slovakian",
        "sl": "Slovenian",
        "su": "Sundanese",
        "sw": "Swahili",
        "sv": "Swedish",
        "tl": "Tagalog",
        "tg": "Tajik",
        "ta": "Tamil",
        "tt": "Tatar",
        "te": "Telugu",
        "udm": "Udmurt",
        "uk": "Ukrainian",
        "ur": "Urdu",
        "uz": "Uzbek",
        "cy": "Welsh",
        "xh": "Xhosa",
        "yi": "Yiddish"
      },
      selectedLanguages: ['nl','en','fr','de','it','pl','pt','ru','es','tr','vi','ar','zh','ja','ko','th'],
      word: '',
      output: [],
      isLoading: false
    }
  },
  computed: {
    sortOutput: function() {
      //return _.orderBy(this.output, 'characterCount', 'desc');
      return this.output.sort(function(a, b) {
        return b.characterCount - a.characterCount;
      });
    },
  },
  watch: {
    selectedLanguages: function() {
      // if just removing, don't re-translate
    },
    word: function() {
      this.translate();
    }
  },
  methods: {
    init: function() {
      this.$refs.input.focus();
    },
    translate: _.debounce(
      function() {
        if (this.word == '') {
          this.output = [];
        } else {
          this.output = [];

          var self = this,
              tempStore = [];

          var getTranslation = function(i, code) {
            self.isLoading = true;
            var text = self.word;
            self.$http.get('https://translate.yandex.net/api/v1.5/tr.json/translate?key=trnsl.1.1.20161205T032544Z.7e1492088f252553.93da07ad618b8fef174afcdf00b72efa811e0388&text=' + text + '&lang=en-' + code + '')
            .then(function(response) {
              tempStore.push({
                characterCount: response.data.text[0].length,
                code: code,
                lang: self.supportedLanguages[code],
                translation: response.data.text[0]
              });

              if (i === self.selectedLanguages.length - 1) {
                self.isLoading = false;
                self.output = tempStore;
              }
            }, function(response) {
              console.log('ERROR!!!!!!!');
            });
          }

          // translate for each code in selectedLanguages array
          for (var i = 0; i < self.selectedLanguages.length; i++) {
            getTranslation(i, self.selectedLanguages[i]);
          }
        }
      }, 1000),
    selectLanguages: function(which) {
      //var self = this;
      this.selectedLanguages = [];
      this.output = [];
      if (which == 'all') {
        // _.forEach(this.supportedLanguages, function(key, value) {
        //   self.selectedLanguages.push(value);
        // });
        for (var lang in this.supportedLanguages) {
          if (this.supportedLanguages.hasOwnProperty(lang)) {
            this.selectedLanguages.push(lang);
          }
        }
      }
      this.translate();
    },
    selectText: function(e) {
      console.log(e.target.value);
    },
    clearInput: function() {
      this.word = '';
      this.output = [];
    }
  },
  mounted() {
    this.init();
  }
}
</script>
