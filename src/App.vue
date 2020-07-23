<template>
  <v-app>
    <div class="row mt-5">
      <div class="col-8 offset-2">
        <!-- <FileImport 
          @inputCSV="assignCSV"
          @inputHeader="assignHeader"
          @detailedInput="assignDetailedInfo"
          :searchIndex="index"></FileImport> -->
        <v-file-input ref="idx" name="idx" label="File input" @change="selectFile"></v-file-input>
        <v-card v-if="tableData != null">
          <v-card-title>
            Abstract
            <v-spacer></v-spacer>
            <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Search in Abstracts, Titles, and Key Words"
              single-line
              hide-details
            ></v-text-field>
          </v-card-title>
            <v-data-table
              v-model="selected"
              :headers="headerData"
              :items="tableData"
              item-key="DO"
              :item-per-page="5"
              :search="search"
              :custom-filter="customFilter"
              show-select
              class="elevation-1"
              @input="update"
            >
              <!-- <template v-slot:top>
                <div class="text-right mr-4">
                  <v-menu
                    v-model="menu"
                    :close-on-content-click="false"
                    :nudge-width="200"
                    offset-x
                  >
                    <template v-slot:activator="{ on, attrs }">
                      <v-btn
                        fab dark small
                        color="#c8cbcf"
                        v-bind="attrs"
                        v-on="on"
                      >
                        <v-icon dark>mdi-format-list-bulleted-square</v-icon>
                      </v-btn>
                    </template>

                    <v-card>
                      <v-list>
                        <v-list-item>
                          <v-list-item-title>Search Your Text in</v-list-item-title>
                        </v-list-item>
                      </v-list>

                      <v-divider></v-divider>

                      <div class="ml-4">
                        <v-form>
                          <v-checkbox v-model="searchFields"
                            value="AB"
                            label="Abstracts"
                            hide-details
                            :rules="rules"
                            ></v-checkbox>
                          <v-checkbox
                            v-model="searchFields"
                            value="KW"
                            label="Key Words"
                            hide-details
                            :rules="rules"
                            ></v-checkbox>
                          <v-checkbox
                            v-model="searchFields"
                            value="TI"
                            label="Titles"
                            :rules="rules"
                            ></v-checkbox>
                        </v-form>
                      </div>
                    </v-card>
                  </v-menu>
                </div>
              </template> -->
            </v-data-table>
        </v-card>
        <div class="container">
          <div class="row">
            <div :class="cardClass" v-if="card1Data != null">
              <v-card
                class="mx-auto"
                max-width="700"
              >
                <v-card-text>
                  <p>doi: {{ card1Data.DO }}</p>
                  <p class="display-1 text--primary">
                    {{ card1Data.TI }}
                  </p>
                  <p>by {{ card1Data.AU }}</p>
                  <p>{{ card1Data.Y1 }}</p>
                  <p class="font-weight-bold text--primary">
                    Key Words: {{ card1Data.KW }}
                  </p>
                  <p class="h6 text--primary">
                    Abstract:
                  </p>
                  <p class="text--primary" v-html="highlight(card1Data.AB)">
                  </p>
                </v-card-text>
                <v-card-actions>
                  <v-btn
                    text
                    color="deep-purple accent-4"
                    @click="goToEvents(card1Data.UR)"
                  >
                    Learn More
                  </v-btn>
                </v-card-actions>
              </v-card>
            </div>
            <div :class="cardClass" v-if="card2Data != null">
              <v-card
                class="mx-auto"
                max-width="500"
              >
                <v-card-text>
                  <p>doi: {{ card2Data.DO }}</p>
                  <p class="display-1 text--primary">
                    {{ card2Data.TI }}
                  </p>
                  <p>by {{ card2Data.AU }}</p>
                  <p>{{ card2Data.Y1 }}</p>
                  <p class="font-weight-bold text--primary">
                    Key Words: {{ card2Data.KW }}
                  </p>
                  <p class="h6 text--primary">
                    Abstract:
                  </p>
                  <p class="text--primary" v-html="highlight(card2Data.AB)">
                  </p>
                </v-card-text>
                <v-card-actions>
                  <v-btn
                    text
                    color="deep-purple accent-4"
                    @click="goToEvents(card2Data.UR)"
                  >
                    Learn More
                  </v-btn>
                </v-card-actions>
              </v-card>
            </div>
          </div>
        </div>
      </div>
    </div>
  </v-app>
</template>

<script>
// import FileImport from './components/FileImport';
import lunr from 'lunr';
import data from '../data.json';
import fullData from '../fullData.json'
import headers from '../headers.json'
// import $ from 'jquery'
// import preidx from '../index.json'

export default {
  name: 'App',

  // components: {
  //   FileImport
  // },

  data() {
    return {
      index: null,
      tableData: data,
      detailedInfo: fullData,
      headerData: headers,
      search: '',
      searchResult: [],
      awaitingSearch: false,
      menu: false,
      selected: [],
      card1Data: null,
      card2Data: null,
      cardClass: "col-12 mt-5",
      searchFields: ["AB"],
    }
  },

  methods: {
    assignCSV(e) {
      this.tableData = e;
    },
    assignHeader(e) {
      this.headerData = e;
    },
    assignDetailedInfo(e) {
      this.detailedInfo = e;

      this.index = lunr(function () {
        this.ref('id');
        this.field('DO');
        this.field('AB');
      });
      // this.index = lunr(function () {
      //   this.ref('id');
      //   this.field('DO');
      //   this.field('AB');
      //   this.field('TI');
      //   this.field('KW');

      //   for (let i = 0; i < documents.length; i++) {
      //     this.add(documents[i]);
      //   }
      // });
    },
    update() {
      // if (this.selected.length > 2) {
      //   this.selected.shift();
      // } else if (this.selected.length > 0) {
      //   let doi = this.selected[0].DO;
      //   let result = this.index.search(doi);
      //   this.card1Data = this.detailedInfo[parseInt(result[0].ref)];
      //   if (this.selected.length > 1) {
      //       doi = this.selected[1].DO;
      //       result = this.index.search(doi);
      //       this.card2Data = this.detailedInfo[parseInt(result[0].ref)];
      //       this.cardClass = "col-6 mt-5";
      //   } else {
      //     this.card2Data = null;
      //     this.cardClass = "col-12 mt-5";
      //   }
      // } else {
      //   this.card1Data = null;
      //   this.card2Data = null;
      // }
    },
    goToEvents(url) {
      window.open(url);
    },
    customFilter(value, search, item) {
      this.selected = [];
      this.card1Data = null;
      this.card2Data = null;
      let id = item.id.toString();

      if (this.searchResult.indexOf(id) > -1) {
        return value != null &&
          search != null &&
          typeof value === 'string';
      }
      return false;
    },
    highlight(text) {
      let idx = text.toLowerCase().indexOf(this.search.toLowerCase());
      let ans = '';
      while (this.search != '' && idx >= 0) {
        ans = ans + text.substring(0, idx) + "<span class='highlight'>"
         + text.substring(idx, idx + this.search.length)
         + "</span>";
        text = text.substring(idx + this.search.length);
        idx = text.toLowerCase().indexOf(this.search.toLowerCase());
      }
      ans = ans + text;
      return ans;
    },
    selectFile() {
      let file = this.$refs.idx.files[0];

      if (file) {
          let reader = new FileReader();
          reader.readAsText(file, "UTF-8");

          reader.onload = function (event) {
            this.index = lunr.Index.load(JSON.parse(event.target.result));
          }

          reader.onerror = function () {
          };
      }
      // $(document).ready(function() {
      //   $.getJSON("abstract-presenter-new/index.json", function(data) {
      //     this.index = lunr.Index.load(JSON.parse(data))
      //     console.log('here')
      //   })
      // })
      // fs.readFile('index.json', function(err, data) {
      //   if (!err) {
      //     this.index = lunr.Index.load(JSON.parse(data));
      //   }
      // })
    }
  },
  computed: {
    rules() {
      return [
        this.searchFields.length > 0 || "At least one field should be selected"
      ];
    }
  },

  watch: {
    search: function () {
      let temp = [];
      if (!this.awaitingSearch) {
        setTimeout(() => {
          if (this.search != '') {
            this.searchResult = this.index.search(this.search);
          } else {
            this.searchResult = [];
          }
          temp = [];
          for (let i = 0; i < this.searchResult.length; i++) {
            temp[i] = this.searchResult[i].ref;
          }
          this.searchResult = temp;
          this.awaitingSearch = false;
        }, 2000);
      }
      this.awaitingSearch = true;
    },
  }
};
</script>

<style>
  .highlight {
    background-color: yellow;
  }
  .v-btn.focus {
    box-shadow: none;
  }
</style>
