<style lang='scss'>
  @import '../node_modules/html5-reset/assets/css/reset';
  @import 'assets/_fonts';

  html::-webkit-scrollbar {
    display: none;
  }

  body, input, textarea {
    font-family: 'Noto Sans', sans-serif;
    font-size: 14px;
    line-height: 1;
    color: #555;
  }

  .small {
    font-size: 12px;
  }

  .controls {
    background: #555;
  }

  .language-list {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 200px;
    overflow: scroll;
    padding: 30px;
    background: #e6e6e6;
    &::-webkit-scrollbar {
      display: none;
    }
    .select-control {
      margin-bottom: 20px;
    }
    label {
      margin-bottom: 10px;
      input {
        margin-right: 5px;
        vertical-align: baseline;
        &:focus {
          outline: none;
        }
      }
    }
    .th {
      padding-bottom: 20px;
      border-bottom: 1px solid #ccc;
      margin-bottom: 20px;
    }
  }

  .translation-cont {
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 200px;
    overflow: scroll;
    background: #fff;
    &::-webkit-scrollbar {
      display: none;
    }
    input {
      position: fixed;
      width: 100%;
      height: 60px;
      border: none;
      padding-left: 30px;
      background: #f3f3f3;
      &:focus {
        outline: none;
      }
    }
    table.output {
      width: 100%;
      //max-width: 600px;
      margin-top: 90px;
      text-align: left;
      thead {
        text-transform: uppercase;
        letter-spacing: 1px;
        color: red;
        tr th {
          padding-bottom: 10px;
        }
        tr {
          border-bottom: 1px solid #e6e6e6;
        }
        th {
          width: 33.333%;
        }
      }
      tbody {
        tr {
          border-bottom: 1px solid #e6e6e6;
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
          }
        }
      }
      th:first-child,
      td:first-child {
        padding-left: 30px;
      }
    }
  }

  input[type=text], label {
    display: block;
    margin-bottom: 10px;
  }

  .slide-fade-enter-active {
    transition: opacity 1s ease-in-out;
  }
  .slide-fade-leave-active {
    transition: opacity .3s ease-in-out;
  }
  .slide-fade-enter, .slide-fade-leave-to
  /* .slide-fade-leave-active for <2.1.8 */ {
    //transform: translateX(10px);
    opacity: 0;
  }
</style>

<template>
  <div id="app">
    <div class="language-list small">
      <div class="select-control">Select: <button @click="selectInputs('all')" class="select-all" type="button" name="button">All</button> <button @click="selectInputs('none')" class="select-none" type="button" name="button">None</button></div>
      <label v-for="(language, index) in languages" v-bind:class="index">
        <input class="language" type="checkbox" v-bind:value="index" v-model="languages[index].selected" @change="translate">{{ language.language }}
      </label>
    </div>
    <div class="translation-cont">
      <input @keyup.enter="translate" v-model="word" class="word-input" type="text" name="word" placeholder="Enter word" autocomplete="off">
      <transition name="slide-fade">
      <table class="output" v-if="hasOutput">
        <thead class="small">
          <tr>
            <th class="language">Language</th>
            <th class="translation">Translation</th>
            <th class="character-count">Character Count</th>
          </tr>
        </thead>
        <tbody>
        <tr v-for="item in sortOutput" v-model="output">
          <td>{{ item.language }}<span class="lang-label">{{ item.code }}</span></td>
          <td><span v-bind:class="item.code">{{ item.translation }}</span></td>
          <td>{{ item.characterCount }}</td>
        </tr>
        </tbody>
      </table>
      </transition>
    </div>
  </div>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'app',
  data() {
    return {
      languages: {
        'nl': {
          'language': 'Dutch',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'en': {
          'language': 'English',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'fr': {
          'language': 'French',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'de': {
          'language': 'German',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'it': {
          'language': 'Italian',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'pl': {
          'language': 'Polish',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'pt': {
          'language': 'Portuguese',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'ru': {
          'language': 'Russian',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'es': {
          'language': 'Spanish',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'tr': {
          'language': 'Turkish',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'vi': {
          'language': 'Vietnamese',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'ar': {
          'language': 'Arabic',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'zh': {
          'language': 'Chinese',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'ja': {
          'language': 'Japanese',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'ko': {
          'language': 'Korean',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'th': {
          'language': 'Thai',
          'tier': 1,
          'selected': true,
          'translation': ''
        },
        'af': {
          'language': 'Afrikaans',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'sq': {
          'language': 'Albanian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        // 'am': {
        //   'language': 'Amharic',
        //   'tier': 2,
        //   'selected': false,
        //   'translation': ''
        // },
        'hy': {
          'language': 'Armenian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'az': {
          'language': 'Azerbaijan',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ba': {
          'language': 'Bashkir',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'eu': {
          'language': 'Basque',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'be': {
          'language': 'Belarusian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'bn': {
          'language': 'Bengali',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'bs': {
          'language': 'Bosnian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'bg': {
          'language': 'Bulgarian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ca': {
          'language': 'Catalan',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ceb': {
          'language': 'Cebuano',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'hr': {
          'language': 'Croatian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'cs': {
          'language': 'Czech',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'da': {
          'language': 'Danish',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'eo': {
          'language': 'Esperanto',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'et': {
          'language': 'Estonian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'fi': {
          'language': 'Finnish',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'gl': {
          'language': 'Galician',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ka': {
          'language': 'Georgian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'el': {
          'language': 'Greek',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'gu': {
          'language': 'Gujarati',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ht': {
          'language': 'Haitian (Creole)',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'he': {
          'language': 'Hebrew',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'mrj': {
          'language': 'Hill Mari',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'hi': {
          'language': 'Hindi',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'hu': {
          'language': 'Hungarian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'is': {
          'language': 'Icelandic',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'id': {
          'language': 'Indonesian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ga': {
          'language': 'Irish',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'jv': {
          'language': 'Javanese',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'kn': {
          'language': 'Kannada',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'kk': {
          'language': 'Kazakh',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ky': {
          'language': 'Kyrgyz',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'la': {
          'language': 'Latin',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'lv': {
          'language': 'Latvian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'lt': {
          'language': 'Lithuanian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'mk': {
          'language': 'Macedonian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'mg': {
          'language': 'Malagasy',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ms': {
          'language': 'Malay',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        // 'ml': {
        //   'language': 'Malayalam',
        //   'tier': 2,
        //   'selected': false,
        //   'translation': ''
        // },
        'mt': {
          'language': 'Maltese',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'mi': {
          'language': 'Maori',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'mr': {
          'language': 'Marathi',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'mhr': {
          'language': 'Mari',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'mn': {
          'language': 'Mongolian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ne': {
          'language': 'Nepali',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'no': {
          'language': 'Norwegian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'pap': {
          'language': 'Papiamento',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'fa': {
          'language': 'Persian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        // 'pa': {
        //   'language': 'Punjabi',
        //   'tier': 2,
        //   'selected': false,
        //   'translation': ''
        // },
        'ro': {
          'language': 'Romanian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'gd': {
          'language': 'Scottish Gaelic',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'sr': {
          'language': 'Serbian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'si': {
          'language': 'Sinhala',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'sk': {
          'language': 'Slovakian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'sl': {
          'language': 'Slovenian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'su': {
          'language': 'Sundanese',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'sw': {
          'language': 'Swahili',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'sv': {
          'language': 'Swedish',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'tl': {
          'language': 'Tagalog',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'tg': {
          'language': 'Tajik',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ta': {
          'language': 'Tamil',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'tt': {
          'language': 'Tatar',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'te': {
          'language': 'Telugu',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'udm': {
          'language': 'Udmurt',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'uk': {
          'language': 'Ukrainian',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'ur': {
          'language': 'Urdu',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'uz': {
          'language': 'Uzbek',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'cy': {
          'language': 'Welsh',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'xh': {
          'language': 'Xhosa',
          'tier': 2,
          'selected': false,
          'translation': ''
        },
        'yi': {
          'language': 'Yiddish',
          'tier': 2,
          'selected': false,
          'translation': ''
        }
      },
      word: '',
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
    // https://vuejs.org/v2/guide/computed.html
    translate: _.debounce(
      function() {
        if (this.word == '') {
          // clear the output
          this.output = [];
          this.hasOutput = false;
          console.log('the input is empty');
        } else {
          // clear the output
          this.output = [];

          // get the word to translate
          var self = this,
              text = this.word,
              lang;

          // translate and add to object
          _.forIn(this.languages, function(key, value) {
            var code = [value][0],
                language = key.language;

            if (key.selected === true) {
              self.$http.get('https://translate.yandex.net/api/v1.5/tr.json/translate?key=trnsl.1.1.20161205T032544Z.7e1492088f252553.93da07ad618b8fef174afcdf00b72efa811e0388&text=' + text + '&lang=en-' + code + '')
              .then(function(response) {
                self.output.push({
                  language: language,
                  code: code,
                  translation: response.data.text[0],
                  characterCount: response.data.text[0].length
                });
              }, function(response) {
                console.log('Error');
              });
            }
          });
          this.hasOutput = true;
          console.log(this.output);
        }
      }, 500),
    selectInputs: function(which) {
      var self = this;
      if (which == "all") {
        //console.log('you selected all');
        _.forIn(this.languages, function(key, value) {
          var code = [value][0];
          self.languages[code].selected = true;
        });
      } else {
        //console.log('you selected none');
        _.forIn(this.languages, function(key, value) {
          var code = [value][0];
          self.languages[code].selected = false;
        });
      }
      this.translate();
    }
  }
}
</script>
