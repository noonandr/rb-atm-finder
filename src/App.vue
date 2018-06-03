<template>
  <div id="app">

    <div class="page-width">

      <div class="feed">

        <div class="title">Your closest 10 ATM locations</div>

        <div v-if="loading">
          Loading...
        </div>

        <div v-for="(atm, index) in orderedAtms" :key="atm.id" v-if="index < 10">

          <div class="item">
            <div class="details">
              <div class="bank"><span class="label">Bank </span><span class="value">Lloyds Bank</span></div>
              <div class="country"><span class="label">Country </span><span class="value">{{ atm.Location.PostalAddress.Country }}</span></div>
              <div class="address"><span class="label">Address </span><span class="value">{{ atm.Location.PostalAddress.AddressLine[0] }}</span></div>
              <div class="distance"><span class="label">Distance </span><span class="value">{{ atm.distance }} Km</span></div>
            </div>
            <div class="geo-location">
              <a class="link" v-bind:href="atm.url" target="_blank"><img src="./assets/pin.png"></a>
            </div>
          </div>

        </div>
      </div>
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
              atm.distance = distance.toFixed(2);

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

body {
  background: #FAFAFA;
  font-family: Helvetica, Arial, Sans-Serif;
  color: #1c242b;
}

.page-width {
    max-width: 800px;
    margin: 0 auto;
    padding: 0 12px;

    @media (min-width: $screen-small) {
        padding: 0 u(2);
    }
}

.feed {
  padding: 8px 12px;
  background: #FFFFFF;
}

.title {
  padding: 18px 0 24px;
  text-align: center;
  font-size: 24px;
}

.item {
  padding: 18px;
  display: flex;
  border-top: 2px solid #FAFAFA;

  @media (min-width: 480px) {
    padding: 18px 8px;
  }
}

.details {
  flex: 1 1 auto;
}

.geo-location {
  display: flex;
  flex: 0 1 auto;
  align-items: center;
}

.link img {
  width: 28px;
  padding: 0 6px;
}

.label {
  width: 80px;
  color: #1EAEDB;
  font-weight: 200;
}

.bank {
  display: flex;
}

.country {
  display: flex;
}

.address {
  display: flex;
}

.distance {
  display: flex;
}


// -- RESET.CSS

html {
    overflow-x: hidden;
    height: 100%;
    margin: 0 auto;
    font-size: 100%;
    box-sizing: border-box;
    -ms-text-size-adjust: 100%;
    -webkit-text-size-adjust: 100%;
}

* {
    box-sizing: inherit;

    ::before,
    ::after {
        box-sizing: inherit;
    }
}


// -- body

body {
    overflow: show; // bricks reset legacy  (@todo investigate dropping this)
    position: relative; // bricks reset legacy  (@todo investigate dropping this)
    margin: 0;
    font: 16px/1 sans-serif;
    -moz-osx-font-smoothing: grayscale;
    -webkit-font-smoothing: antialiased;
}

// -- typography

h1,
h2,
h3,
h4,
h5,
h6,
p,
blockquote,
figure,
ol,
dl,
dt,
dd,
ul {
    margin: 0;
    padding: 0;
}

main {
    display: block;
}

ul,
ol {
    display: block;
    list-style-type: none;
}

h1,
h2,
h3,
h4,
h5,
h6 {
    font-size: inherit;
}

a {
    text-decoration: none;

    &:hover,
    &:active {
        outline: 0;
    }
}

input,
textarea,
keygen,
select,
button {
    overflow: hidden;
    font: inherit;
    font-family: inherit;
    letter-spacing: inherit;
    -webkit-font-smoothing: inherit;
}

button  {
    vertical-align: middle;
    border: 0;
    border-style: solid;
    -webkit-appearance: button;
}

select {
    overflow: auto;
    border-radius: 0;
}

input {

    &[type="button"],
    &[type="reset"],
    &[type="submit"] {
        vertical-align: middle;
        border: 0;
        border-style: solid;
        -webkit-appearance: button;
    }

    &[type="number"] {
        -moz-appearance: textfield;

        &::-webkit-inner-spin-button,
        &::-webkit-outer-spin-button {
            margin: 0;
            -webkit-appearance: none;
        }
    }
}

:focus {
    outline: 0;
}


img {
    overflow-x: hidden;
    vertical-align: middle;
    max-width: 100%;
    height: auto;
    border: 0;
}

table {
    border-collapse: collapse;
    border-spacing: 0;
}

</style>
