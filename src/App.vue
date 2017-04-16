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
    <div class="select-control"><button @click="selectInputs('all')" class="select-all" type="button" name="button">ALL</button> <button @click="selectInputs('none')" class="select-none" type="button" name="button">NONE</button></div>
    <div class="languages small">
      <label v-for="(language, index) in supportedLanguages" v-bind:class="index">
        <input class="language" type="checkbox" v-bind:value="index" v-model="selectedLanguages" @change="translate">{{ language.language }}
      </label>
    </div>
    <div class="input-container">
    <input ref="butt" @keyup.enter="translate" v-model="word" class="word-input" type="text" name="word" placeholder="Enter word" autocomplete="off">
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
        <transition name="fade">
        <tbody v-if="output">
        <tr v-bind:class="index" v-for="(language, index) in supportedLanguages" v-if="selectedLanguages.includes(index) && supportedLanguages[index].translation">
          <td>{{ language.language }}<span class="lang-label">{{ index }}</span></td>
          <td><input @click="selectText" v-bind:class="index" class="translation" type="text" v-bind:value="language.translation" readonly></td>
          <td>3</td>
        </tr>
        </tbody>
        </transition>
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
        'nl': {
          'language': 'Dutch',
          'selected': true,
          'translation': ''
        },
        'en': {
          'language': 'English',
          'selected': true,
          'translation': ''
        },
        'fr': {
          'language': 'French',
          'selected': true,
          'translation': ''
        },
        'de': {
          'language': 'German',
          'selected': true,
          'translation': ''
        },
        'it': {
          'language': 'Italian',
          'selected': true,
          'translation': ''
        },
        'pl': {
          'language': 'Polish',
          'selected': true,
          'translation': ''
        },
        'pt': {
          'language': 'Portuguese',
          'selected': true,
          'translation': ''
        },
        'ru': {
          'language': 'Russian',
          'selected': true,
          'translation': ''
        },
        'es': {
          'language': 'Spanish',
          'selected': true,
          'translation': ''
        },
        'tr': {
          'language': 'Turkish',
          'selected': true,
          'translation': ''
        },
        'vi': {
          'language': 'Vietnamese',
          'selected': true,
          'translation': ''
        },
        'ar': {
          'language': 'Arabic',
          'selected': true,
          'translation': ''
        },
        'zh': {
          'language': 'Chinese',
          'selected': true,
          'translation': ''
        },
        'ja': {
          'language': 'Japanese',
          'selected': true,
          'translation': ''
        },
        'ko': {
          'language': 'Korean',
          'selected': true,
          'translation': ''
        },
        'th': {
          'language': 'Thai',
          'selected': true,
          'translation': ''
        },
        'af': {
          'language': 'Afrikaans',
          'selected': false,
          'translation': ''
        },
        'sq': {
          'language': 'Albanian',
          'selected': false,
          'translation': ''
        },
        'am': {
          'language': 'Amharic',
          'selected': false,
          'translation': ''
        },
        'hy': {
          'language': 'Armenian',
          'selected': false,
          'translation': ''
        },
        'az': {
          'language': 'Azerbaijan',
          'selected': false,
          'translation': ''
        },
        'ba': {
          'language': 'Bashkir',
          'selected': false,
          'translation': ''
        },
        'eu': {
          'language': 'Basque',
          'selected': false,
          'translation': ''
        },
        'be': {
          'language': 'Belarusian',
          'selected': false,
          'translation': ''
        },
        'bn': {
          'language': 'Bengali',
          'selected': false,
          'translation': ''
        },
        'bs': {
          'language': 'Bosnian',
          'selected': false,
          'translation': ''
        },
        'bg': {
          'language': 'Bulgarian',
          'selected': false,
          'translation': ''
        },
        'ca': {
          'language': 'Catalan',
          'selected': false,
          'translation': ''
        },
        'ceb': {
          'language': 'Cebuano',
          'selected': false,
          'translation': ''
        },
        'hr': {
          'language': 'Croatian',
          'selected': false,
          'translation': ''
        },
        'cs': {
          'language': 'Czech',
          'selected': false,
          'translation': ''
        },
        'da': {
          'language': 'Danish',
          'selected': false,
          'translation': ''
        },
        'eo': {
          'language': 'Esperanto',
          'selected': false,
          'translation': ''
        },
        'et': {
          'language': 'Estonian',
          'selected': false,
          'translation': ''
        },
        'fi': {
          'language': 'Finnish',
          'selected': false,
          'translation': ''
        },
        'gl': {
          'language': 'Galician',
          'selected': false,
          'translation': ''
        },
        'ka': {
          'language': 'Georgian',
          'selected': false,
          'translation': ''
        },
        'el': {
          'language': 'Greek',
          'selected': false,
          'translation': ''
        },
        'gu': {
          'language': 'Gujarati',
          'selected': false,
          'translation': ''
        },
        'ht': {
          'language': 'Haitian (Creole)',
          'selected': false,
          'translation': ''
        },
        'he': {
          'language': 'Hebrew',
          'selected': false,
          'translation': ''
        },
        'mrj': {
          'language': 'Hill Mari',
          'selected': false,
          'translation': ''
        },
        'hi': {
          'language': 'Hindi',
          'selected': false,
          'translation': ''
        },
        'hu': {
          'language': 'Hungarian',
          'selected': false,
          'translation': ''
        },
        'is': {
          'language': 'Icelandic',
          'selected': false,
          'translation': ''
        },
        'id': {
          'language': 'Indonesian',
          'selected': false,
          'translation': ''
        },
        'ga': {
          'language': 'Irish',
          'selected': false,
          'translation': ''
        },
        'jv': {
          'language': 'Javanese',
          'selected': false,
          'translation': ''
        },
        'kn': {
          'language': 'Kannada',
          'selected': false,
          'translation': ''
        },
        'kk': {
          'language': 'Kazakh',
          'selected': false,
          'translation': ''
        },
        'ky': {
          'language': 'Kyrgyz',
          'selected': false,
          'translation': ''
        },
        'la': {
          'language': 'Latin',
          'selected': false,
          'translation': ''
        },
        'lv': {
          'language': 'Latvian',
          'selected': false,
          'translation': ''
        },
        'lt': {
          'language': 'Lithuanian',
          'selected': false,
          'translation': ''
        },
        'mk': {
          'language': 'Macedonian',
          'selected': false,
          'translation': ''
        },
        'mg': {
          'language': 'Malagasy',
          'selected': false,
          'translation': ''
        },
        'ms': {
          'language': 'Malay',
          'selected': false,
          'translation': ''
        },
        'ml': {
          'language': 'Malayalam',
          'selected': false,
          'translation': ''
        },
        'mt': {
          'language': 'Maltese',
          'selected': false,
          'translation': ''
        },
        'mi': {
          'language': 'Maori',
          'selected': false,
          'translation': ''
        },
        'mr': {
          'language': 'Marathi',
          'selected': false,
          'translation': ''
        },
        'mhr': {
          'language': 'Mari',
          'selected': false,
          'translation': ''
        },
        'mn': {
          'language': 'Mongolian',
          'selected': false,
          'translation': ''
        },
        'ne': {
          'language': 'Nepali',
          'selected': false,
          'translation': ''
        },
        'no': {
          'language': 'Norwegian',
          'selected': false,
          'translation': ''
        },
        'pap': {
          'language': 'Papiamento',
          'selected': false,
          'translation': ''
        },
        'fa': {
          'language': 'Persian',
          'selected': false,
          'translation': ''
        },
        'pa': {
          'language': 'Punjabi',
          'selected': false,
          'translation': ''
        },
        'ro': {
          'language': 'Romanian',
          'selected': false,
          'translation': ''
        },
        'gd': {
          'language': 'Scottish Gaelic',
          'selected': false,
          'translation': ''
        },
        'sr': {
          'language': 'Serbian',
          'selected': false,
          'translation': ''
        },
        'si': {
          'language': 'Sinhala',
          'selected': false,
          'translation': ''
        },
        'sk': {
          'language': 'Slovakian',
          'selected': false,
          'translation': ''
        },
        'sl': {
          'language': 'Slovenian',
          'selected': false,
          'translation': ''
        },
        'su': {
          'language': 'Sundanese',
          'selected': false,
          'translation': ''
        },
        'sw': {
          'language': 'Swahili',
          'selected': false,
          'translation': ''
        },
        'sv': {
          'language': 'Swedish',
          'selected': false,
          'translation': ''
        },
        'tl': {
          'language': 'Tagalog',
          'selected': false,
          'translation': ''
        },
        'tg': {
          'language': 'Tajik',
          'selected': false,
          'translation': ''
        },
        'ta': {
          'language': 'Tamil',
          'selected': false,
          'translation': ''
        },
        'tt': {
          'language': 'Tatar',
          'selected': false,
          'translation': ''
        },
        'te': {
          'language': 'Telugu',
          'selected': false,
          'translation': ''
        },
        'udm': {
          'language': 'Udmurt',
          'selected': false,
          'translation': ''
        },
        'uk': {
          'language': 'Ukrainian',
          'selected': false,
          'translation': ''
        },
        'ur': {
          'language': 'Urdu',
          'selected': false,
          'translation': ''
        },
        'uz': {
          'language': 'Uzbek',
          'selected': false,
          'translation': ''
        },
        'cy': {
          'language': 'Welsh',
          'selected': false,
          'translation': ''
        },
        'xh': {
          'language': 'Xhosa',
          'selected': false,
          'translation': ''
        },
        'yi': {
          'language': 'Yiddish',
          'selected': false,
          'translation': ''
        }
      },
      selectedLanguages: ['nl','en','fr','de','it','pl','pt','ru','es','tr','vi','ar','zh','ja','ko','th'],
      word: '',
      isLoading: false,
      output: [],
      hasOutput: false,
      errorMsg: ''
    }
  },
  computed: {
    sortOutput: function() {
      return _.orderBy(this.output, 'characterCount', 'desc');
      this.hasOutput = true;
    },
  },
  watch: {
    word: function() {
      this.translate();
    }
  },
  methods: {
    init: function() {
      this.$refs.butt.focus();
    },
    translate: _.debounce(
      function() {
        if (this.word == '') {
          this.output = [];
          this.hasOutput = false;
        } else {
          this.output = [];

          var self = this;

          var getTranslation = function(i, code) {
            self.isLoading = true;
            var text = self.word;
            self.$http.get('https://translate.yandex.net/api/v1.5/tr.json/translate?key=trnsl.1.1.20161205T032544Z.7e1492088f252553.93da07ad618b8fef174afcdf00b72efa811e0388&text=' + text + '&lang=en-' + code + '')
            .then(function(response) {
              // self.supportedLanguages[code].push({
              //   //characterCount: response.data.text[0].length,
              //   translation: response.data.text[0]
              // });
              self.supportedLanguages[code].translation = response.data.text[0];
              if (i === self.selectedLanguages.length - 1) {
                self.isLoading = false;
                //console.log('false');
              }
            }, function(response) {
              console.log('ERROR!!!!!!!');
            });
          }

          // for each code in selectedLanguages array
          for (var i = 0; i < self.selectedLanguages.length; i++) {
            var code = self.selectedLanguages[i];
            //console.log(self.supportedLanguages[self.selectedLanguages[i]]);
                //lang = self.supportedLanguages[code];
            getTranslation(i, code);
          }
          this.hasOutput = true;
        }
      }, 1000),
    selectInputs: function(which) {
      var self = this;
      if (which == "all") {
        _.forEach(this.supportedLanguages, function(key, value) {
          var code = value;
          self.selectedLanguages.push(code);
        });
      } else {
        this.selectedLanguages = [];
      }
      this.translate();
    },
    selectText: function(e) {
      console.log(e.target.value);
    },
    clearInput: function() {
      this.word = '';
    }
  },
  mounted() {
    this.init();
  }
}

//TODO make a table with every language and show if has translation, hide if nah

</script>
