<template>
  <v-app>
    <div class="row mt-5">
      <div class="col-8 offset-2">
        <FileImport @inputCSV="assignCSV" @inputHeader="assignHeader"></FileImport>
        <v-data-table
          v-if="tableData != null"
          :headers="headerData"
          :items="tableData"
          :item-per-page="5"
          class="elevation-1"
        ></v-data-table>
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
      headerData: null
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
