<template>
  <div id="app">
      <div class="left-side">
          <div class="notification-wrapper" :class="{'notification-error': notification.type === 'error', 'notification-success': notification.type === 'success'}" v-if="notification.show">
              {{ notification.message }}
              <button @click="hideNotification" class="notification-hide-button">x</button>
          </div>
          <div class="search-content">
              <label>Search string: <input @keyup="searchIndex" v-model="searchString"></label>
              <button class="" @click="searchIndex">Search</button>
          </div>
          <div class="content">
              <ul>
                  <li v-for="airport in content">
                      <h2>{{ airport.Name }}</h2>
                      <div>
                          <ul>
                              <li>{{ airport.Latitude }}</li>
                              <li>{{ airport.Longitude }}</li>
                              <!-- <li>{{airport}}</li> -->
                          </ul>
                      </div>
                  </li>
              </ul>
          </div>
      </div>
      <div class="map-container">
          <div id="map"></div>
      </div>
  </div>
</template>

<script>
const config = require('../config')

const algoliasearch = require('algoliasearch')

let client = algoliasearch(config.algolia.applicationId, config.algolia.searchOnlyApiKey, {protocol: 'https:'})
// let client = algoliasearch(config.algolia.applicationId, config.algolia.adminApiKey, {protocol: 'https:'})
let index = client.initIndex('airports') // the index name used refers to the indices to use in your algolia dashboard

const GoogleMapsLoader = require('google-maps')


export default {
    name: 'app',
    data() {
        return {
            content: '',
            map: null,
            markers:[],
            error: null,
            newObject: '',
            searchString: '',
            notification: {
                message: '',
                show: false,
                type: 'error'
            }
        }
    },
    methods:{
        searchIndex(){
            let string = this.searchString
            index.search(string, (err, content) => {
                if(err){
                    console.error(err)
                    this.showNotification(err.message, 'error')
                } else {
                    this.showNotification('Results found', 'success')
                    this.content = content.hits
                    this.setMarkers()
                }
            })
        },
        setMarkers(){
            this.clearMarkers()
            this.content.map(airport => {
                let re = /(\w{1})(\d{2})(\d.+)/
                let explodedLat = re.exec(airport.Latitude)
                airport.lat = Number(`${explodedLat[2]}.${explodedLat[3]}`)
                airport.lat = Number(`${explodedLat[2]}.${explodedLat[3]}`)

                let explodedLng = re.exec(airport.Longitude)
                airport.lng = Number(`${explodedLng[2]}.${explodedLng[3]}`)
                airport.lng = Number(`${explodedLng[2]}.${explodedLng[3]}`)
                return airport
            })
            .forEach(airport => {
                this.markers.push(this.makeMarker(airport.lat, airport.lng))
            })
            this.fitBounds()
        },
        clearMarkers(){
            debugger
            this.markers.forEach(marker => {
                marker.setMap(null)
            })
            this.markers = []
        },
        makeMarker(lat,lng){
            console.log('marker placed')
            return new google.maps.Marker({
                position: {lat,lng},
                map: this.map
            });
        },
        fitBounds(){
            var bounds = new google.maps.LatLngBounds();
            for (var i = 0; i < this.markers.length; i++) {
                bounds.extend(this.markers[i].getPosition());
            }

            this.map.fitBounds(bounds);
        },
        showNotification(message, type){
            this.notification.message = message
            this.notification.type = type
            this.notification.show = true
        },
        hideNotification(){
            this.notification.message = ''
            this.notification.show = false
        }
    },
    mounted(){
        GoogleMapsLoader.KEY = "AIzaSyApFDlPdNA2_EMMUao5y0oC-27zwS2sEhY"
        GoogleMapsLoader.load(google => {
            this.map = new google.maps.Map(document.getElementById('map'), {
                zoom:4,
                center: {lat: 46.227638, lng: 2.213749} })
        })
    }
}
</script>
<style>
    body,html{
        height: 100%;
        width: 100%;
        padding:0;
        margin:0;
        box-sizing: border-box;
    }
    body{
        display:flex;
        justify-content: center;
    }

    .content{
    }

    #app{
        display: flex;
        flex:1;
    }
    .left-side{
        flex: 1;
        overflow-x: scroll;
    }

    .map-container{
        flex: 2;
        display: flex;
        /*height:100px;
        width: 100px;*/
    }

    #map{
        flex: 1;
    }
</style>
