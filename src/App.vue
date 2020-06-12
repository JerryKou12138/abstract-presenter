<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png">
    <!-- <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <div class="container">
            <div class="container">
              <section class="py-5">
                  <div class="row mt-5">
                      <div class="col-8 offset-2">
                          <vue-csv-import 
                            v-model="csv" 
                            autoMatchFields = "true"
                            :map-fields="['file', 'book', 'conference', 'DO', 'TY',
                             'TI', 'T2','SP', 'EP', 'PY', 'JO', 'IS', 'SN', 'VO',
                              'VL', 'JA', 'Y1', 'PB','UR', 'AU', 'KW', 'AB']">
                            </vue-csv-import>
                          <div class="mt-2">
                            {{ csv }}
                          </div>
                      </div>
                  </div>
              </section>
            </div>
        </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import VueCsvImport from 'vue-csv-import';

export default {
  name: 'App',
  data() {
    return {
      csv: null,
    }
  },
  components: {
    // HelloWorld,
    VueCsvImport
  }
}
</script>

<style>
    #app {
        font-family: "Avenir", Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }

    .container {
        text-align: left;
    }

    pre code {
        background-color: #eee;
        border: 1px solid #999;
        display: block;
        padding: 20px;
    }

    #app .form {
        text-align: left;
    }
</style>
