<template>
  <v-app>
    <div class="row mt-5">
      <div class="col-8 offset-2">
        <FileImport 
          @inputCSV="assignCSV"
          @inputHeader="assignHeader"
          @detailedInput="assignDetailedInfo"></FileImport>
        <v-card v-if="tableData != null">
          <v-card-title>
            Abstract
            <v-spacer></v-spacer>
            <v-text-field
              v-model="search"
              append-icon="mdi-magnify"
              label="Search"
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
              show-select
              class="elevation-1"
              @input="update"
            ></v-data-table>
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
                  <p class="text--primary">
                    {{ card1Data.AB }}
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
                  <p class="text--primary">
                    {{ card2Data.AB }}
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
// import HelloWorld from './components/HelloWorld';
import FileImport from './components/FileImport';
import lunr from 'lunr';

export default {
  name: 'App',

  components: {
    // HelloWorld,
    FileImport
  },

  data() {
    return {
      index: null,
      tableData: null,
      detailedInfo: null,
      headerData: null,
      search: '',
      selected: [],
      card1Data: null,
      card2Data: null,
      cardClass: "col-12 mt-5"
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
      let documents = e;
      let counter = 0;
      documents.forEach(function (doc) {
        doc['id'] = counter++;
      });
      this.index = lunr(function () {
        this.ref('id');
        this.field('DO');

        documents.forEach(function (doc) {
          this.add(doc);
        }, this)
      });
    },
    update() {
      if (this.selected.length > 2) {
        this.selected.shift();
      } else if (this.selected.length > 0) {
        let doi = this.selected[0].DO;
        let result = this.index.search(doi);
        this.card1Data = this.detailedInfo[parseInt(result[0].ref)];
        if (this.selected.length > 1) {
            doi = this.selected[1].DO;
            result = this.index.search(doi);
            this.card2Data = this.detailedInfo[parseInt(result[0].ref)];
            this.cardClass = "col-6 mt-5";
        } else {
          this.card2Data = null;
          this.cardClass = "col-12 mt-5";
        }
      } else {
        this.card1Data = null;
        this.card2Data = null;
      }
    },
    goToEvents(url) {
      window.open(url);
    }
  },

};
</script>
