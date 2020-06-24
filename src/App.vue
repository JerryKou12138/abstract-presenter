<template>
  <v-app>
    <div class="row mt-5">
      <div class="col-8 offset-2">
        <FileImport @inputCSV="assignCSV" @inputHeader="assignHeader"></FileImport>
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
              :headers="headerData"
              :items="tableData"
              :item-per-page="5"
              :search="search"
              class="elevation-1"
            ></v-data-table>
        </v-card>
      </div>
    </div>
  </v-app>
</template>

<script>
// import HelloWorld from './components/HelloWorld';
import FileImport from './components/FileImport';

export default {
  name: 'App',

  components: {
    // HelloWorld,
    FileImport
  },

  data() {
    return {
      tableData: null,
      headerData: null,
      search: ''
      }
  },

  methods: {
    assignCSV(e) {
      this.tableData = e;
    },
    assignHeader(e) {
      this.headerData = e;
    }
  }
};
</script>
