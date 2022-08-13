<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="tool-bar-color">
        <ion-title>
          <div class="header-title">
            <h2>Range Finder <small v-if="user.username">| {{user.username}}</small></h2>
            <img style="width: 55px" src="./icons/Rangefinder.png" alt="">
          </div>
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Range Finder <small v-if="user.username">| {{user.username}}</small></ion-title>
        </ion-toolbar>
      </ion-header>
      <div class="main-container">
        <div class="container">
          <div class="upper-container">
            <div class="distance-container">
              <p style="color: black;">{{distance}} yards</p>
            </div>
            <div>
              <!-- <p style="color: black;" class="club-suggestion">{{clubLogic}}</p> -->
              <p style="color: black;" v-if="10 < distance && distance <= updatedClubSettings.SandWedge" class="club-suggestion">60 Wedge</p>
              <p style="color: black;" v-if="updatedClubSettings.SandWedge < distance && distance <= updatedClubSettings.PitchingWedge" class="club-suggestion">P Wedge</p>
              <p style="color: black;" v-if="updatedClubSettings.PitchingWedge < distance && distance <= updatedClubSettings.NineIron" class="club-suggestion">9 Iron</p>
              <p style="color: black;" v-if="updatedClubSettings.NineIron < distance && distance <= updatedClubSettings.EightIron" class="club-suggestion">8 Iron</p>
              <p style="color: black;" v-if="updatedClubSettings.EightIron < distance && distance <= updatedClubSettings.SevenIron" class="club-suggestion">7 Iron</p>
              <p style="color: black;" v-if="updatedClubSettings.SevenIron < distance && distance <= updatedClubSettings.SixIron" class="club-suggestion">6 Iron</p>
              <p style="color: black;" v-if="updatedClubSettings.SixIron < distance && distance <= updatedClubSettings.FiveIron" class="club-suggestion">5 Iron</p>
              <p style="color: black;" v-if="updatedClubSettings.FiveIron < distance && distance <= updatedClubSettings.FourIron" class="club-suggestion">4 Iron</p>
              <p style="color: black;" v-if="updatedClubSettings.FourIron < distance && distance <= updatedClubSettings.ThreeIron" class="club-suggestion">3 Iron</p>
              <p style="color: black;" v-if="updatedClubSettings.ThreeIron < distance && distance <= updatedClubSettings.FiveWood" class="club-suggestion">5 Wood</p>
              <p style="color: black;" v-if="updatedClubSettings.FiveWood < distance && distance <= updatedClubSettings.ThreeWood" class="club-suggestion">3 Wood</p>
              <p style="color: black;" v-if="updatedClubSettings.ThreeWood < distance && distance <= updatedClubSettings.Driver && updatedClubSettings.Driver != 0" class="club-suggestion">Driver</p>
              <!-- <p style="color: black;" v-if="updatedClubSettings.Driver == 0" class="club-suggestion">Sign In</p> -->
            </div>
            <div>
              <div class="wind-data">
                <img style="width: 25px" src="./icons/wind-logo.png" alt="">
                <p style="color: black;">{{windSpeed}} mph</p>
                <div class="arrowWind" id="arrowWind"></div>
              </div>
              
            </div>
          </div>
        
          <!-- <div ref="mapDiv" id="mapDiv" style="width: 100%; height: 65vh;"/>
          <button id="get-location" class="get-location-button" @click="GetLocation(), GetWeatherData()">Update Location</button> -->
        </div>
      </div>
      <!-- <ion-refresher slot="fixed" @ionRefresh="doRefresh($event)">
        <ion-refresher-content></ion-refresher-content>
      </ion-refresher> -->
      <div ref="mapDiv" id="mapDiv" style="width: 100%; height: 70vh;"/>
      <div class="location-container-main">
        <div class="location-button-container" id="location-button-container">
          <button id="get-location" class="get-location-button" @click="GetLocation(), GetWeatherData()">Update Location <img style="width: 15px; height: 15px; background-color: white; border-radius: 500px; margin-left: 5%;" src="./icons/golf-ball-2-remove.png" alt=""></button>
          <div class="guide-user" id="guide-user">
            <div class="inner-guide-user" id="inner-guide-user"></div>
            <div class="inner-guide-user-2" id="inner-guide-user-2"></div>
          </div>
        </div>
      </div>


      <p style="color: black; display: none;">{{updatedClubSettings}}</p>
      
          
    </ion-content>
  </ion-page>
</template>

<script>
/* eslint-disable no-undef */
import { computed, defineComponent, onMounted, ref, watch } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
//import {useGeolocation} from '../useGeolocation.js';
import { Loader } from '@googlemaps/js-api-loader';
import { Geolocation } from '@capacitor/geolocation';
//import { HTTP } from '@awesome-cordova-plugins/http/ngx';
import * as APIReqest from '@ionic-native/http';
import {Auth} from 'aws-amplify';
import {AuthState, onAuthUIStateChange} from "@aws-amplify/ui-components";


export default  defineComponent({
  name: 'Tab1Page',
  components: {IonHeader, IonToolbar, IonTitle, IonContent, IonPage}, //GoogleMap, Polyline, Marker },
  data(){
    return{
        clubSelection:{
          SandWedge: 60,
          PitchingWedge: 80,
          NineIron: 95,
          EightIron: 110,
          SevenIron: 120,
          SixIron: 130,
          FiveIron: 140, 
          FourIron: 150,
          ThreeIron: 160,
          FiveWood: 170,
          ThreeWood: 180,
          Driver: 210
        },
        userLocation: {
          lat: 0,
          lng: 0
        },
        windSpeed: 0,
        windDirection: 0,
        ballLocationTest: {},
        setDistance: {},
        markerCount: 0,
        setPlayerLocation: false,
        setBallLocation: false,
        updatedClubSettings: {
          username: "",
          SandWedge: 0,
          PitchingWedge: 0,
          NineIron: 0,
          EightIron: 0,
          SevenIron: 0,
          SixIron: 0,
          FiveIron: 0, 
          FourIron: 0,
          ThreeIron: 0,
          FiveWood: 0,
          ThreeWood: 0,
          Driver: 0
        },
        user: {
          username: ""
        }
    }
  },
  methods:{
    async GetClubSettings(){
      await Auth.currentAuthenticatedUser().then(user => {
            this.user = user;
            this.updatedClubSettings.username = user.username;
        }).catch(error => {
          console.log(error)
          this.signedIn = false;
        });

      //this.updatedClubSettings.username = this.user.username;
      const requestOptions = {
        method: "GET",
        headers: {
          'Content-Type': 'application/json',
          'username': this.user.username
        }
      };
      const response = await  fetch('https://yka89jo4bd.execute-api.us-east-2.amazonaws.com/prod/getclubs', requestOptions)
      const data = await response.json();
      if(data.items){
        console.log(data.Items)
        this.updatedClubSettings = data.Items[0];
      }
      
      //this.updatedClubSettings = this.updatedClubSettings;
    },
    GetLocation(){
      var userLatitude;
      var userLongitude;
      Geolocation.getCurrentPosition().then((response)=>{
        // console.log(response.coords)
        // console.log("User Latitude: " + response.coords.latitude)
        // console.log("User Longitude: " + response.coords.longitude)
        userLatitude = response.coords.latitude;
        userLongitude = response.coords.longitude;
        this.userLocation.lat = userLatitude;
        this.userLocation.lng = userLongitude;
        }).catch((err)=> console.log(err))
      
      console.log(userLatitude, userLongitude)
      this.$router.go(0);
    },
    async GetWeatherData(){
      APIReqest.HTTP.get('https://api.openweathermap.org/data/2.5/weather?lat=' + this.userLocation.lat + '&' + 'lon=' + this.userLocation.lng + '&appid=5afb314d8ef4094a42eb11e6bee894ae',{}, {})
      .then(data => {
        console.log(data);
      }).catch(err=>{console.log(err)})

      await fetch('https://api.openweathermap.org/data/2.5/weather?lat=' + this.userLocation.lat + '&' + 'lon=' + this.userLocation.lng + '&appid=5afb314d8ef4094a42eb11e6bee894ae')
      .then(response => {
        let returnData = response.json();
        returnData.then(dataReturned=>{
          console.log(dataReturned.wind)
          this.windSpeed = dataReturned.wind.speed;
          this.windDirection = dataReturned.wind.deg;
          document.getElementById('arrowWind').style.transform = "rotate(-" + this.windDirection + "deg)";
          })
        //console.log(returnData)
      })
    }
  },
  mounted(){
    this.GetClubSettings();
    this.GetWeatherData();

    Auth.currentAuthenticatedUser().then(user => {
            this.user = user;
            this.updatedClubSettings.username = user.username;
            console.log(user.username)
            
            // this.token = user.signInUserSession.accessToken.jwtToken;
            // this.cognitoUserName = user.username;
            // this.cognitoJWT = user.signInUserSession.accessToken.jwtToken;
            // this.GetTrackShipmentsByCriteriaInTransit();
            console.log(user)
        }).catch(error => {
          console.log(error)
          //this.$router.push('Login');
          //Auth.signOut({global: true})
        });

      onAuthUIStateChange((nextAuthState, authData) => {
      Auth.currentAuthenticatedUser().then(user => {
            this.user = user;
            this.updatedClubSettings.username = user.username;
            console.log(user.username)
            this.signedIn = true;
            console.log(user)
            this.GetClubSettings();
        }).catch(error => {
          console.log(error)
          this.signedIn = false;
        });
            if (nextAuthState === AuthState.SignedIn) {
                this.signedIn = true;
                this.GetClubSettings();
            }
            if (!authData) {
                this.signedIn = false;
            }
        });
  },
  setup(){

    var otherPosition = ref(null)
    var ballPosition = ref(null)
    //Google Maps JS API
    const loader = new Loader({
      apiKey: "AIzaSyAdpl8aEv7NgfIFYQA_9NFtiNayACiojrE"
    });

    let map = ref(null)

    var userLocation = {
          lat: 36.5621,
          lng: -121.940
        }

    onMounted(async ()=> {
      var userLatitude;
      var userLongitude;

      const div = document.createElement('div');
      div.className = 'direct-user';
      div.innerHTML = '<p style="font-size: 10px; margin-top: 10px; color: black;">Select the current ball location.</p>';
      document.getElementById('inner-guide-user').appendChild(div);

        Geolocation.requestPermissions();
        Geolocation.getCurrentPosition().then((response)=>{
          console.log(response.coords)
          console.log("User Latitude: " + response.coords.latitude)
          console.log("User Longitude: " + response.coords.longitude)
          userLatitude = response.coords.latitude;
          userLongitude = response.coords.longitude;
          userLocation.lat = userLatitude;
          userLocation.lng = userLongitude;

         loader.load().then((google) => {
          console.log('Location Data!')
          console.log(userLatitude, userLongitude)
          map.value = new google.maps.Map(document.getElementById("mapDiv"), {
            center: userLocation,
            zoom: 19,
            mapTypeId: "satellite",
            streetViewControl: false,
            rotateControl: false,
            mapTypeControl: false,
            zoomControl: false
          });

          const svgMarker = {
            path: "M396.532,135.845c9.333,3.155,23.906-4.313,25.676-20.272c1.08-9.725-1.913-19.221-11.36-22.252l-37.639-3.51    c-13.109-3.892-27.138,3.519-31.298,16.562l-34.808,93.598l-17.729,4.141L282.93,33.957c0.144-13.693-10.863-25.121-24.528-25.522    L221.557,0c-9.917-0.077-15.73,8.013-17.748,17.585c-3.318,15.711,8.186,27.358,18.044,27.282L262.054,47l6.474,161.989    l-28.754,6.722L190.919,84.332c-4.16-13.043-18.178-20.455-31.298-16.562l-37.628,3.509c-9.438,3.041-12.431,12.527-11.36,22.252    c1.769,15.96,16.342,23.428,25.675,20.272l38.852-10.557l43.901,117.303l-6.062,1.415c-12.976,0-23.792,9.094-26.526,21.257    c-12.775,3.414-55.769,21.879-49.113,114.387c4.15,57.604,27.884,90.251,48.405,107.961v40.038    c0,15.042,12.192,27.234,27.234,27.234h116.51c15.042,0,27.233-12.183,27.233-27.234v-4.542h24.7    c10.031,0,18.149-8.128,18.149-18.149V380.311c0-10.03-8.128-18.149-18.149-18.149h-24.7v-65.293h7.373    c10.021,0,18.149-8.138,18.149-18.149v-45.384c0-10.031-8.138-18.15-18.149-18.15h-8.329    c-2.859-11.063-12.46-19.335-24.174-20.244l26.058-69.625L396.532,135.845z M185.784,424.528    c-10.222-15.559-19.01-37.696-21.257-68.869c-3.978-55.271,11.293-75.219,21.257-82.344V424.528z",
            fillColor: "#fff",
            fillOpacity: 1,
            strokeWeight: 0,
            rotation: 0,
            scale: .1,
            anchor: new google.maps.Point(15, 30),
          };

          new google.maps.Marker({
              position: userLocation,
              map: map.value,
              icon: svgMarker
            });


            for(let i = 0; i <= 2; i++){
              map.value.addListener('click',({latLng: {lat, lng}})=>{if(i==1){(otherPosition.value = {lat: lat(), lng: lng()}); i+=1; console.log("Player Location Selected");}})
              map.value.addListener('click',({latLng: {lat, lng}})=>{if(i==2){(ballPosition.value = {lat: lat(), lng: lng()}); console.log("Ball Location Selected");}})
            }
        })
        .catch(e => {d
          alert(e)
        });
        }).catch((err)=> console.log(err))

      let line = null
      let newMarker = null
      let newBallMarker = null
      watch([map, ballPosition, otherPosition], () => {
        if (line) line.setMap(null)
        if (map.value && ballPosition.value != null)
          line = new google.maps.Polyline({
            path: [otherPosition.value, ballPosition.value],
            map: map.value,
            strokeColor: "#fff"
          })
      });

      watch([ballPosition], () => {
        if (newBallMarker) newBallMarker.setMap(null)
          const svgMarker3 = {
            path: "M402.541,233.58h107.285C499.522,109.441,400.386,10.305,276.247,0.001v107.285 C341.506,116.694,393.133,168.321,402.541,233.58z M359.426,276.249h-19.179c-11.797,0-21.333-9.557-21.333-21.333c0-11.776,9.536-21.333,21.333-21.333h19.179 c-8.512-41.728-41.451-74.667-83.179-83.179v19.179c0,11.776-9.536,21.333-21.333,21.333c-11.797,0-21.333-9.557-21.333-21.333 v-19.179c-41.728,8.512-74.667,41.451-83.179,83.179h19.179c11.797,0,21.333,9.557,21.333,21.333 c0,11.776-9.536,21.333-21.333,21.333h-19.179c8.512,41.728,41.451,74.667,83.179,83.179v-19.179 c0-11.776,9.536-21.333,21.333-21.333c11.797,0,21.333,9.557,21.333,21.333v19.179 C317.975,350.915,350.914,317.977,359.426,276.249z M254.914,276.249c-11.776,0-21.333-9.557-21.333-21.333 c0-11.776,9.557-21.333,21.333-21.333c11.776,0,21.333,9.557,21.333,21.333C276.247,266.691,266.69,276.249,254.914,276.249z M233.579,107.289V0.003C109.44,10.307,10.304,109.443,0,233.582h107.285C116.693,168.323,168.32,116.697,233.579,107.289z M107.287,276.251H0.002c10.304,124.139,109.44,223.275,233.579,233.579V402.544 C168.322,393.136,116.695,341.509,107.287,276.251z M402.545,276.249c-9.408,65.259-61.035,116.885-126.293,126.293v107.285c124.139-10.304,223.275-109.44,233.579-233.579 H402.545z",
            fillColor: "#fff",
            fillOpacity: 1,
            strokeWeight: 0,
            rotation: 0,
            scale: .05,
            anchor: new google.maps.Point(267, 270),
          };

          newBallMarker = new google.maps.Marker({
            position: ballPosition.value,
            map: map.value,
            icon: svgMarker3
          });
      });

      watch([otherPosition], () => {
        if (newMarker) newMarker.setMap(null)
          const svgMarker2 = {
            path: "M253.851,124.664c0-22.109-18.178-40.039-40.631-40.039c-22.424,0-40.603,17.92-40.603,40.039    c0,22.232,18.178,40.22,40.603,40.22C235.634,164.883,253.851,146.896,253.851,124.664z, M419.321,163.87c-1.588-8.587-8.635-13.521-9.056-13.895L187.554,3.191c-8.816-5.805-20.827-3.471-26.833,5.221    l-11.274,16.352c-4.198,6.407-2.362,14.736,3.959,18.886c6.359,4.208,13.33,1.042,17.423-5.374l8.597-21.181l199.608,131.972    l-158.767,28.668c-11.284,1.817-19.393,13.101-16.878,24.078l34.54,133.655l-111.7,167.593    c-6.742,11.437-2.668,26.182,8.903,32.704c11.695,6.646,26.431,2.562,33.163-8.865c0,0,116.567-175.271,117.007-176.466    l6.072,24.489l-17.604,139.823c-2.171,13.015,6.761,25.302,20.005,27.502c13.205,2.218,25.675-6.627,27.922-19.642l18.055-143.505    c0.573-3.461-0.144-11.073-0.976-14.344l-39.79-158.422l103.667-18.666C413.766,185.71,421.233,175.039,419.321,163.87z",
            fillColor: "#fff",
            fillOpacity: 1,
            strokeWeight: 0,
            rotation: 0,
            scale: .1,
            anchor: new google.maps.Point(267, 270),
          };

          newMarker = new google.maps.Marker({
            position: otherPosition.value,
            map: map.value,
            icon: svgMarker2
          });

          document.getElementById('inner-guide-user').remove();
          const div = document.createElement('div');
          div.className = 'ball-location';
          div.innerHTML = '<p style="font-size: 10px; margin-top: 10px; color: black;">Select the desired location.</p>';
          document.getElementById('inner-guide-user-2').appendChild(div);
        });
      });

      const haversineDistance = (pos1, pos2) => {
      const R = 3958.8 // Radius of the Earth in miles
      const rlat1 = pos1.lat * (Math.PI / 180) // Convert degrees to radians
      const rlat2 = pos2.lat * (Math.PI / 180) // Convert degrees to radians
      const difflat = rlat2 - rlat1 // Radian difference (latitudes)
      const difflon = (pos2.lng - pos1.lng) * (Math.PI / 180) // Radian difference (longitudes)
      const d =
        2 *
        R *
        Math.asin(
          Math.sqrt(
            Math.sin(difflat / 2) * Math.sin(difflat / 2) +
              Math.cos(rlat1) *
                Math.cos(rlat2) *
                Math.sin(difflon / 2) *
                Math.sin(difflon / 2)
          )
        )
      return Math.round(d * 1760)
    }

    var distance = computed(() =>
      ballPosition.value === null
        ? 0
        : haversineDistance(otherPosition.value, ballPosition.value)
    )

    let clubSelection = {
          SandWedge: 60,
          PitchingWedge: 80,
          NineIron: 95,
          EightIron: 110,
          SevenIron: 120,
          SixIron: 130,
          FiveIron: 140, 
          FourIron: 150,
          ThreeIron: 160,
          FiveWood: 170,
          ThreeWood: 180,
          Driver: 210
        }

    let clubChoice;
    const clubLogic = computed(()=>{
        if(10 < distance.value && distance.value < clubSelection.SandWedge){
          clubChoice = '60 Wedge'
        }
        if(clubSelection.SandWedge < distance.value && distance.value < clubSelection.PitchingWedge){
          clubChoice = 'P Wedge'
        } 
        if(clubSelection.PitchingWedge < distance.value && distance.value < clubSelection.NineIron){
          clubChoice = '9 Iron'
        } 
        if(clubSelection.NineIron < distance.value && distance.value < clubSelection.EightIron){
          clubChoice = '8 Iron'
        } 
        if(clubSelection.EightIron < distance.value && distance.value < clubSelection.SevenIron){
          clubChoice = '7 Iron'
        } 
        if(clubSelection.SevenIron < distance.value && distance.value < clubSelection.SixIron){
          clubChoice = '6 Iron'
        } 
        if(clubSelection.SixIron < distance.value && distance.value < clubSelection.FiveIron){
          clubChoice = '5 Iron'
        } 
        if(clubSelection.FiveIron < distance.value && distance.value < clubSelection.FourIron){
          clubChoice = '4 Iron'
        } 
        if(clubSelection.FourIron < distance.value && distance.value < clubSelection.ThreeIron){
          clubChoice = '3 Iron'
        } 
        if(clubSelection.ThreeIron < distance.value && distance.value < clubSelection.FiveWood){
          clubChoice = '5 Wood'
        } 
        if(clubSelection.FiveWood < distance.value && distance.value < clubSelection.ThreeWood){
          clubChoice = '3 Wood'
        } 
        if(clubSelection.ThreeWood < distance.value && distance.value < clubSelection.Driver){
          clubChoice = 'Driver'
        }
      return clubChoice;
      
    })

    return{ clubChoice, distance, clubLogic }
  }
});
</script>

<style scoped>
ion-content{
  --ion-background-color:#f5f5dd;
}

ion-title{
  color:#f5f5dd;
}

.header-title{
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.tool-bar-color{
  --background: #a57a5a;
}

.main-container{
  width: 100%;
  display: flex;
  justify-content: center;
}

.upper-container{
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2.5px;
}

.wind-data{
  display: flex;
  flex-direction: row;
  align-items: center;
}

.wind-data img, p{
  margin-right: 5px;
}

.container{
  display: flex;
  justify-content: center;
  flex-direction: column;
  width: 80%;
}

.location-button-container{
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 90%;
}

.location-container-main{
  width: 100%;
  display: flex;
  justify-content: center;
}

.get-location-button{
  width: 50%;
  background-color: #076652;
  color: #fff;
  padding: 10px;
  margin-top: 10px;
  border-radius: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.arrowWind{
    width: 0; 
    height: 0; 
    border-left: 5px solid transparent;
    border-right: 5px solid transparent;
    
    border-bottom: 5px solid black;
}
</style>
