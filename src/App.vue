<template>
  <div id="app">

    <button v-on:click="get_atms">Get atms</button>

    <div v-if="loading">
      Loading...
    </div>
 
    <div v-for="(atm, index) in latSortedAtms" :key="atm.id" v-if="index <= 4">

        <p>{{ atm.Location.PostalAddress.AddressLine[0] }}</p>
        <p>{{ atm.Location.PostalAddress.Country }}</p>
        <br>

    </div>
  </div>
</template>

<script>
  import axios from 'axios';
  
  export default {
    name: 'app',
    data () {
      return {
        latSortedAtms: [],
        loading: false
      }
    }, 
    methods: {
      get_atms: function () {
         return axios("https://api.lloydsbank.com/open-banking/v2.1/atms", {
          method: 'GET'
        }).then(response => {
          this.loading = false;
          let atms = response.data.data[0].Brand[0].ATM;
          this.latSortedAtms = atms.sort(function(a, b){
            return a.Location.PostalAddress.GeoLocation.GeographicCoordinates.Latitude > b.Location.PostalAddress.GeoLocation.GeographicCoordinates.Latitude;
          })
        })
      }
    }
  }
</script>

<style>

</style>
