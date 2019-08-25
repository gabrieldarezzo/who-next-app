<template>
  <q-page class="flex flex-center">

    <div id="mapView" class="google-map" ref="googleMap" style='width:500px;height:92vh;'></div>
    <template v-if="Boolean(this.google) && Boolean(this.map)">
      <slot
        :google="google"
        :map="map"
      />
    </template>
  </q-page>
</template>

<style scoped>
  .google-map {
    /* height: 92vh; */
    width: 100%;
    min-height: 100%;
   }
</style>

<script>
import GoogleMapsApiLoader from 'google-maps-api-loader'
import axios from 'axios'

export default {
  name: 'MapView',
  data: function () {
    return {
      // pinBlue: '~assets/pins/blue.png',
      pinBlue: require('./../assets/pins/blue.png'),
      google: null,
      map: null,
      users: [],
      actualMark: null,
      usersMarks: [],
      actualPosition: null,
      actualPositionLatLng: null
    }
  },
  props: {
    mapConfig: Object,
    apiKey: String
  },
  methods: {
    loadUsers () {
      const dataCusto = Object.assign(this.actualPositionLatLng, {
        'km_distance': 150
      })

      axios
        .post('http://192.168.0.8:8000/api/users_next', dataCusto)
        .then((response) => {
          this.users = response.data

          this.users.map((user) => {
            const mark = new this.google.maps.Marker({
              position: new this.google.maps.LatLng(user.lat, user.lng),
              map: this.map,
              title: user.name,
              draggable: false
              //  icon: this.pinBlue
            })
            this.usersMarks.push(mark)
          })
        })
    },
    initializeMap () {
      // console.log(position)

      // Pegar a posição Atual dinamicamente
      this.actualPosition = new this.google.maps.LatLng(-23.9772151, -46.3082780)
      this.actualPositionLatLng = {
        lat: -23.9772151,
        lng: -46.3082780
      }
      const mapContainer = document.getElementById('mapView')
      // const latlng = new this.google.maps.LatLng(this.position.lat, this.position.lng)
      console.log(this.pinBlue)
      this.map = new this.google.maps.Map(mapContainer, {
        center: this.actualPosition,
        zoom: 15,
        mapTypeId: this.google.maps.MapTypeId.ROADMAP
      })

      this.actualMark = new this.google.maps.Marker({
        position: this.actualPosition,
        map: this.map
      })
      this.loadUsers()

      /*
      // após resolver a treta do HTTPS
      navigator.geolocation.getCurrentPosition((position) => {
        // console.log(position)
        this.actualPosition = new this.google.maps.LatLng(position.coords.latitude, position.coords.longitude)
        this.actualPositionLatLng = {
          lat: position.coords.latitude,
          lng: position.coords.longitude
        }
        const mapContainer = document.getElementById('mapView')
        // const latlng = new this.google.maps.LatLng(this.position.lat, this.position.lng)
        console.log(this.pinBlue)
        this.map = new this.google.maps.Map(mapContainer, {
          center: this.actualPosition,
          zoom: 15,
          mapTypeId: this.google.maps.MapTypeId.ROADMAP
        })

        this.actualMark = new this.google.maps.Marker({
          position: this.actualPosition,
          map: this.map
        })
        this.loadUsers()
      })
      */
    }
  },
  async mounted () {
    const googleMapApi = await GoogleMapsApiLoader({
      apiKey: 'AIzaSyC-9XzfWIRagerL60bVUzOpPlpZgmL-4n0'
    })

    this.google = googleMapApi
    this.initializeMap()
  }

}
</script>
