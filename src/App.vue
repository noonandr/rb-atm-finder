<template>
  <div id="app">

    <div v-if="loading">
      Loading...
    </div>

    <div v-for="(atm, index) in orderedAtms" :key="atm.id" v-if="index <= 10">

      <p>{{ atm.Location.PostalAddress.AddressLine[0] }}</p>
      <p>{{ atm.Location.PostalAddress.Country }}</p>
      <p>{{ atm.distance }} Km</p>
      <p><a v-bind:href="atm.url" target="_blank">show on map</a></p>
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
        orderedAtms: [],
        loading: false
      }
    },
    methods: {
      get_atms: function () {
        return axios("https://api.lloydsbank.com/open-banking/v2.1/atms", {
          method: 'GET'
        }).then(response => {
          this.loading = true;

          // promise to make location accessible to distance calculation 
          var getPosition = function (options) {
            return new Promise(function (resolve, reject) {
              navigator.geolocation.getCurrentPosition(resolve, reject, options);
            });
          }

          getPosition()
          .then((position) => {
            // sets visitors latitude and longitude
            const visitorLatitude = position.coords.latitude;
            const visitorLongitude = position.coords.longitude;

            // ATM array from response
            let allAtms = response.data.data[0].Brand[0].ATM;

            let atms = [];

            for (var ix = 0; ix < allAtms.length; ix++) {
              // atm
              let atm = allAtms[ix];
              let atmLatitude = atm.Location.PostalAddress.GeoLocation.GeographicCoordinates.Latitude;
              let atmLongitude = atm.Location.PostalAddress.GeoLocation.GeographicCoordinates.Longitude;

              // distance calculation
              const earthsRadius = 6371;
              let radsLatitude = (atmLatitude - visitorLatitude) * Math.PI / 180;
              let radsLongitude = (atmLongitude - visitorLongitude) * Math.PI / 180;

              var chordLength = Math.sin(radsLatitude / 2) * Math.sin(radsLatitude / 2) +
                Math.cos(visitorLatitude * Math.PI / 180) * Math.cos(atmLatitude * Math.PI / 180) *
                Math.sin(radsLongitude / 2) * Math.sin(radsLongitude / 2);

              var angularDistance = 2 * Math.atan2(Math.sqrt(chordLength), Math.sqrt(1 - chordLength));

              let distance = earthsRadius * angularDistance;

              // sets ATMs distance
              atm.distance = distance;

              // sets ATMs Google Maps API
              atm.url = 'https://www.google.com/maps/search/?api=1&query=' + atmLatitude + ',' + atmLongitude;

              // push updated ATM objection into new array
              atms.push(atm);
            }

            function sortDistance(a,b) {
              if (a.distance < b.distance)
                return -1;
              if (a.distance > b.distance)
                return 1;
              return 0;
            }

            this.orderedAtms = atms.sort(sortDistance);

            //  disables loading state
            this.loading = false;

          })

        })
      }
    },
    beforeMount(){
      // run main function on load
      this.get_atms()
    },
  }
</script>

<style>

</style>
