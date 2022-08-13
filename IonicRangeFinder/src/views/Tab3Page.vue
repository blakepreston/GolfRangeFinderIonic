<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="tool-bar-color">
        <ion-title>
          <div class="header-title">
            <h2>My Settings</h2>
            <img style="width: 55px" src="./icons/Rangefinder.png" alt="">
          </div>
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">My Settings</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="amplify-container">
        <div class="text-container" v-if="!signedIn">
          <h2 style="color: #076652;">Welcome to Range Finder!</h2>
          <p>Create an account below or sign in to set your average club distances. This allows us to give you club suggestions based on the yardage
            and current weather conditions. 
          </p>
        </div>
        <amplify-authenticator>
          <!-- eslint-disable-next-line vue/no-deprecated-slot-attribute -->
          <!-- <amplify-sign-in slot="sign-in"
                v-show="authState !== 'signup' && authState !== 'forgotpassword'  && authState !== 'confirmSignUp'"
                header-text="Sign in to update club settings."
          ></amplify-sign-in> -->


          <!-- eslint-disable-next-line vue/no-deprecated-slot-attribute -->
          <!-- <amplify-sign-up slot="sign-up"
                  headerText="Sign up to create a shipment."
                  :formFields="formFields"
          ></amplify-sign-up> -->

          <div class="sign-out-container">
            <!-- eslint-disable-next-line vue/no-deprecated-slot-attribute -->
            <div class="sign-out-inner">
              <amplify-sign-out slot="sign-out"
                id="signout-button">
              </amplify-sign-out>
            </div>
            
          </div>
              
        </amplify-authenticator>
      </div>

      <div class="main-container">
        <div class="inner-container" v-if="signedIn">
          <div class="text-container">
            <p style="color: black;">Hello! {{user.username}}.</p>
            <h2 style="color: black;">Update Club Distances</h2>
            <div class="label-container">
              <label style="color: black;" for="60wedge">60 Wedge</label>
              <input type="number" v-model="clubSettings.SandWedge"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="pitchingwedge">Pitching Wedge</label>
              <input type="number" v-model="clubSettings.PitchingWedge"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="nineiron">Nine Iron</label>
              <input type="number" v-model="clubSettings.NineIron"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="eightiron">Eight Iron</label>
              <input type="number" v-model="clubSettings.EightIron"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="seveniron">Seven Iron</label>
              <input type="number" v-model="clubSettings.SevenIron"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="sixiron">Six Iron</label>
              <input type="number" v-model="clubSettings.SixIron"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="fiveiron">Five Iron</label>
              <input type="number" v-model="clubSettings.FiveIron"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="fouriron">Four Iron</label>
              <input type="number" v-model="clubSettings.FourIron"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="threeiron">Three Iron</label>
              <input type="number" v-model="clubSettings.ThreeIron"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="fivewood">Five Wood</label>
              <input type="number" v-model="clubSettings.FiveWood"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="threewood">Three Wood</label>
              <input type="number" v-model="clubSettings.ThreeWood"/>
            </div>
            <div class="label-container">
              <label style="color: black;" for="driver">Driver</label>
              <input type="number" v-model="clubSettings.Driver"/>
            </div>
            <button class="update-clubs" @click="PostClubSettings()">Update <img style="width: 15px; height: 15px; background-color: white; border-radius: 500px; margin-left: 5%;" src="./icons/golf-ball-2-remove.png" alt=""></button>
          </div>
        </div>
      </div>
      <p style="color: black; display:none;">{{updatedClubSettings}}</p>

      

      <!-- <div class="autocomplete-container">
        <div class="autocomplete-inner">
          <div v-if="authState === 'signedin'">Hello! {{user.username}}.</div>
          <div class="autocomplete-input">
            <input type="text" v-model="searchText.test">
            {{searchText}}
          </div>

          <div class="autocomplete-result">
            <p @click="SelectAddress(index)" v-for="(autoCompleteResult, index) in autoCompleteData" :key="autoCompleteResult">{{autoCompleteResult.Place.Label}}</p>
          </div>

          <div v-if="selectedAddress">
            <h2>Selected Address</h2>
            <p>{{selectedAddress}}</p>
          </div>
          
        </div>
      </div> -->
      

    </ion-content>
  </ion-page>
</template>

<script>
import {AuthState, onAuthUIStateChange} from "@aws-amplify/ui-components";
import {Auth} from 'aws-amplify';
import { defineComponent } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import Location from "aws-sdk/clients/location";

export default defineComponent({
  data(){
    return{
      signedIn: false,
      user: {
        username: ""
      },
      searchText: {
        test: ""
      },
      autoCompleteData: {},
      selectedAddress: '',
      clubSettings: {
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
      }
    }
  },
  name: 'Tab3Page',
  components: { IonHeader, IonToolbar, IonTitle, IonContent, IonPage },
  watch:{
    'searchText.test': function(){
      console.log("Value has changed.")
      this.getClient();
    } 
  },
  methods:{
    async PostClubSettings(){
      this.clubSettings.username = this.user.username;
      const requestOptions = {
        method: "POST",
        headers: {"Content-Type": "application/json"},
        body: JSON.stringify(this.clubSettings)
      };
      const response = await  fetch('https://yka89jo4bd.execute-api.us-east-2.amazonaws.com/prod/clubsettings', requestOptions)
      const data = await response.json();
      console.log(data)
      alert(data.Message + ": Club settings updated.")
    },
    async GetClubSettings(){
      await Auth.currentAuthenticatedUser().then(user => {
            this.user = user;
            this.clubSettings.username = user.username;
        }).catch(error => {
          console.log(error)
          this.signedIn = false;
        });

      this.clubSettings.username = this.user.username;
      const requestOptions = {
        method: "GET",
        headers: {
          'Content-Type': 'application/json',
          'username': this.user.username
        }
      };
      const response = await  fetch('https://yka89jo4bd.execute-api.us-east-2.amazonaws.com/prod/getclubs', requestOptions)
      const data = await response.json();
      console.log(data.Items)
      if(data.items){
        this.updatedClubSettings = data.Items[0];
        this.clubSettings = this.updatedClubSettings;
      }else{
        this.PostClubSettings();
      }
      
    },
    SelectAddress(index){
      //this.selectedAddress = {};
      this.selectedAddress = this.autoCompleteData[index].Place.Label;
    },
    async getClient(){
      const credentials = await Auth.currentCredentials();

      const locationClient = new Location({
          credentials,
          region: 'us-east-2'
      });

      const params = {
          IndexName: "explore.place",
          Text: this.searchText.test,
          FilterCountries: ["USA"],
          BiasPosition: [-74.261398, 40.702503]
        };

      locationClient.searchPlaceIndexForText(params,(err,data)=>{
          if(err){
            console.log(err)
            console.log(credentials)
            }
          if(data){
            console.log(data);
            this.autoCompleteData = data.Results;
          }
        })
    }
  },
  mounted(){
    // Auth.currentAuthenticatedUser().then(user => {
    //         this.user = user;
    //         this.clubSettings.username = user.username;
    //         console.log(user.username)
    //         this.signedIn = true;
    //         console.log(user)
    //     }).catch(error => {
    //       console.log(error)
    //       this.signedIn = false;
    //     });

    this.GetClubSettings();

    // const getClient = async ()=>{
    //   const credentials = await Auth.currentCredentials();

    //   const locationClient = new Location({
    //       credentials,
    //       region: 'us-east-2'
    //   });

    //   const params = {
    //       IndexName: "explore.place",
    //       Text: this.searchText.test,
    //       FilterCountries: ["USA"],
    //       BiasPosition: [-74.261398, 40.702503]
    //     };

    //   locationClient.searchPlaceIndexForText(params,(err,data)=>{
    //       if(err){
    //         console.log(err)
    //         console.log(credentials)
    //         }
    //       if(data){
    //         console.log(data);
    //       }
    //     })
    // }

    //getClient();
    onAuthUIStateChange((nextAuthState, authData) => {
      Auth.currentAuthenticatedUser().then(user => {
            this.user = user;
            this.clubSettings.username = user.username;
            console.log(user.username)
            this.signedIn = true;
            console.log(user)
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
        })
  },
  created(){
    Auth.currentAuthenticatedUser().then(user => {
            this.user = user;
            this.clubSettings.username = user.username;
            console.log(this.user.username)
            this.signedIn = true;
            console.log(user)
        }).catch(error => {
          console.log(error)
          this.signedIn = false;
        });
        // //Cognito-Amplify Login setup
        // onAuthUIStateChange((nextAuthState, authData) => {
        //     if (nextAuthState === AuthState.SignedIn) {
        //         this.signedIn = true;
        //     }
        //     if (!authData) {
        //         this.signedIn = false;
        //     }
        // })
    }
  });
</script>

<style scoped>
p{
  color: #000;
}
ion-content{
  --ion-background-color:#f5f5dd;
}

ion-title{
  color:#f5f5dd;
}

amplify-authenticator{
  --container-height: 100%;
  --width: 450px;
  --height: 450px;
  --amplify-primary-color: #076652;
  --amplify-secondary-tint: #076652;
  --amplify-primary-shade: rgb(39, 159, 199);
  --amplify-primary-tint: #076652;
}

    amplify-sign-out{
      --amplify-primary-color: #076652;
      --amplify-secondary-tint: #076652;
      --amplify-primary-shade: rgb(39, 159, 199);
      --amplify-primary-tint: #076652;
    }

    .amplify-container{
      width: 100%;
      display: flex;
      justify-content: center;
      flex-direction: column;
    }

    .text-container{
      margin: 5%;
      display: flex;
      justify-content: center;
      flex-direction: column;
    }

    .text-container p{
      background-color: #fff;
      border-radius: 15px;
      padding: 5%;
      box-shadow: 0 0 10px rgba(0, 0, 0, 0.205);
    }

    .sign-out-container{
      width: 50%;
      display: block;
      margin: auto;
    }

    .sign-out-inner{
      margin: 10px;
    }

.tool-bar-color{
  --background: #a57a5a;
}

.header-title{
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
}

.main-container{
  width: 100%;
  display: flex;
  justify-content: center;
}

.inner-container{
  display: flex;
  justify-content: center;
  width: 80%;
  margin-bottom: 10%;
}

.label-container{
  width: 90%;
  margin-top: 10px;
}

.label-container input{
  width: 100%;
  border-radius: 15px;
  border: 1px solid #076652;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.26);
  outline: none;
  padding: 5px;
  background-color: #fff;
  color: #000;
  margin-top: 2.5px;
}

.update-clubs{
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

/* Autocomplete */

.autocomplete-container{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
}

.autocomplete-inner{
  width: 50%;
}

.autocomplete-result{
  background-color: #fff;
  border-radius: 15px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.062);
}

@keyframes animate-result {
  from{margin-top: -5px;}
  to{margin-top: 0;}
}

.autocomplete-result p{
  cursor: pointer;
  transition-duration: .5s;
  margin: 0;
  padding: 5px;
  animation: animate-result .5s ease;
}

.autocomplete-result p:hover{
  background-color: rgb(235, 235, 235);
  transition-duration: .5s;
}


.autocomplete-input input{
  border-radius: 15px;
  border: none;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.062);
  outline: none;
  padding: 5px;
  margin-bottom: 5px;
}
</style>