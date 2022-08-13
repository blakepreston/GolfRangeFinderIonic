<template>
  <ion-page>
    <ion-header>
      <ion-toolbar class="tool-bar-color">
        <ion-title>
          <div class="header-title">
            <h2>Score Card</h2>
            <img style="width: 55px" src="./icons/Rangefinder.png" alt="">
          </div>
        </ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Score Card</ion-title>
        </ion-toolbar>
      </ion-header>

      <div class="container">
        <div class="inner-container">
          <div class="score-date-container">
            <h3 style="color: black;" v-if="PlayerScore < 0">Your Score: {{PlayerScore}}</h3>
            <h3 style="color: black;" v-if="PlayerScore > 0">Your Score: +{{PlayerScore}}</h3>
            <h3 style="color: black;" v-if="PlayerScore == 0">Your Score: E</h3>
            <input type="date" name="" id="date">
          </div>
          
          <table class="score-table">
            <thead>
              <tr>
                <th>Hole</th>
                <th>Score</th>
                <th>Par</th>
                <th>Handicap</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(scoreData) in scoreCardData" :key="scoreData.HoleNumber">
                <td style="color: black;">{{scoreData.HoleNumber}}</td>
                <td><input class="table-input" type="number" v-model="scoreData.Score"></td>
                <td><input class="table-input" type="number" v-model="scoreData.Par"></td>
                <td><input class="table-input" type="number" v-model="scoreData.Handicap"></td>
              </tr>
            </tbody>
          </table>
          <div class="button-container">
            <button @click="AddNextHole(), UpdateScore()" class="next-hole-button">Next Hole</button>
            <button v-if="scoreCardData.HoleNumber = 1" @click="SaveUserScore" class="next-hole-button">Save Score</button>
            <button class="next-hole-button" @click="TakePhoto">Photo</button>
          </div>

          <img id="image" src="" alt="">
          <p style="color: black;">{{SaveScoreData}}</p>
        </div>
      </div>
      
      <!-- <ExploreContainer name="Tab 2 page" /> -->
    </ion-content>
  </ion-page>
</template>

<script>
import { defineComponent } from 'vue';
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent } from '@ionic/vue';
import {AuthState, onAuthUIStateChange} from "@aws-amplify/ui-components";
import { Camera, CameraResultType } from '@capacitor/camera';

export default defineComponent({
  name: 'Tab2Page',
  components: { IonHeader, IonToolbar, IonTitle, IonContent, IonPage },
  data(){
    return{
      user: {},
      scoreCardData: [
          {
            HoleNumber: 1,
            Score: 0,
            Par: 0, 
            Handicap: 0
          }
        ],
      PlayerScore: 0,
      SaveScoreData: {
        User: '',
        Score: 0,
        Date: undefined,
        ScoreCardImage: ''
      }
    }
  },
  methods:{
    TakePhoto(){
      const photo = Camera.getPhoto({
        resultType: CameraResultType.Uri,
        //source: CameraSource.Camera,
        quality: 1,
      })
      
      photo.then((response)=>{
        console.log("Response")
        this.SaveScoreData.ScoreCardImage = response.webPath;
        console.log(this.SaveScoreData.ScoreCardImage)
        document.getElementById('image').src = this.SaveScoreData.ScoreCardImage
      }).catch((err)=> alert(err));
    },
    AddNextHole(){
      var newHole = {
        HoleNumber: 0,
        Score: 0,
        Par: 0, 
        Handicap: 0
      }
      
      var lastHoleNumber = this.scoreCardData[this.scoreCardData.length - 1].HoleNumber;

      newHole.HoleNumber = lastHoleNumber + 1;

      if(newHole.HoleNumber <= 18){
        this.scoreCardData.push(newHole)
      }
    },
    UpdateScore(){
      let scoreArray = [];
      let parArray = [];
      this.scoreCardData.forEach(score => {
        scoreArray.push(score.Score)
        parArray.push(score.Par)
      })
      let scoreSum = 0;
      let parSum = 0;
      scoreArray.forEach(x => {
        scoreSum += x;
      })

      parArray.forEach(y => {
        parSum += y;
      })

      this.PlayerScore = scoreSum - parSum;
    },
    SaveUserScore(){
      this.SaveScoreData.Score = this.PlayerScore;
      this.SaveScoreData.Date = document.getElementById('date').value;
      this.SaveScoreData.User = this.user.username;
    }
  },
  created(){
    onAuthUIStateChange((nextAuthState, authData) => {
            if (nextAuthState === AuthState.SignedIn) {
                this.authState = nextAuthState;
                this.signedIn = true;
                // JWT = authData.signInUserSession.accessToken.jwtToken)
                this.user = authData;
                this.token = authData.signInUserSession.accessToken.jwtToken;
                this.njlsUser = authData.attributes;
                console.log(this.token);
            }
           
            if (!authData) {
                this.signedIn = false;
            }

            if(nextAuthState === AuthState.SignUp){
                this.authState = nextAuthState;
                this.signUp = true;
            }else if(nextAuthState === AuthState.ForgotPassword){
                this.authState = nextAuthState;
                this.forgotPassword = true;
            }else if(nextAuthState === AuthState.SignIn){
                this.authState = nextAuthState;
                this.backSignIn = true;
            }
        });
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

    .container{
      display: flex;
      justify-content: center;
      width: 100%;
    }

    .button-container{
      display: flex;
      justify-content: space-between;
    }

    .inner-container{
      width: 95%;
      display: flex;
      justify-content: center;
      flex-direction: column;
      margin-bottom: 20px;
    }

    .score-date-container{
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      margin-top: 5px;
    }

    .score-table{
        width: 100%;
        margin-top: 10px;
        border-collapse: collapse;
        border-top-left-radius: 5px;
        border-top-right-radius: 5px;
        font-size: 0.9em;
        box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
        text-align: center;
    }

    .score-table th:first-child{
        border-top-left-radius: 5px;
    }

    .score-table th:last-child{
        border-top-right-radius: 5px;
    }

    .score-table tbody tr:nth-of-type(even) {
        background-color: #ae6e4e;
    }

    .score-table tbody tr:last-of-type {
        border-bottom: 2px solid #76b6ef;
    }

    .score-table th{
        background-color: #8b4417;
        color: #ffffff;
    }

    .score-table th,
    .score-table td{
        padding: 1.5%;
        width: 20%;
    }

    .score-table tbody tr{
        border-bottom: 1px solid #dddddd;
    }

    .table-input{
        width: 50%;
    }

    .next-hole-button{
      margin-top: 2.5%;
      width: 35%;
      color: #fff;
      padding: 15px;
      border-radius: 50px;
      background-color: #76b6ef;
      transition-duration: .5s;
    }

    .next-hole-button:hover{
      background-color: rgb(41, 174, 219);
      transition-duration: .5s;
    }
</style>
