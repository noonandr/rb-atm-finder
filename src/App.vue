<template>
  <div id="app">

    <div v-if="loading">
      Loading...
    </div>

    <div v-for="(atm, index) in latSortedAtms" :key="atm.id" v-if="index <= 10">

      <p>{{ atm.Location.PostalAddress.AddressLine[0] }}</p>
      <p>{{ atm.Location.PostalAddress.Country }}</p>
      <p>{{ atm.distance }} Km</p>
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
          this.loading = true;

          var getPosition = function (options) {
            return new Promise(function (resolve, reject) {
              navigator.geolocation.getCurrentPosition(resolve, reject, options);
            });
          }

          getPosition()
          .then((position) => {
            const visitorLatitude = position.coords.latitude;
            const visitorLongitude = position.coords.longitude;

            let atms = [];
            let allAtms = response.data.data[0].Brand[0].ATM;
            for (var ix = 0; ix < allAtms.length; ix++) {
              let atm = allAtms[ix];
              let atmLatitude = atm.Location.PostalAddress.GeoLocation.GeographicCoordinates.Latitude;
              let atmLongitude = atm.Location.PostalAddress.GeoLocation.GeographicCoordinates.Longitude;

              const earthsRadius = 6371;
              let radsLatitude = (atmLatitude - visitorLatitude) * Math.PI / 180;
              let radsLongitude = (atmLongitude - visitorLongitude) * Math.PI / 180;

              var chordLength = Math.sin(radsLatitude / 2) * Math.sin(radsLatitude / 2) +
                Math.cos(visitorLatitude * Math.PI / 180) * Math.cos(atmLatitude * Math.PI / 180) *
                Math.sin(radsLongitude / 2) * Math.sin(radsLongitude / 2);

              var angularDistance = 2 * Math.atan2(Math.sqrt(chordLength), Math.sqrt(1 - chordLength));

              let distance = earthsRadius * angularDistance;

              atm.distance = distance;
              atms.push(atm);
            }

            function sortDistance(a,b) {
              if (a.distance < b.distance)
                return -1;
              if (a.distance > b.distance)
                return 1;
              return 0;
            }

            this.latSortedAtms = atms.sort(sortDistance);


            this.loading = false;

          })

        })
      }
    },
    beforeMount(){
      this.get_atms()
    },
  }
</script>

<style>

</style>
