<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet></ion-router-outlet>
      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="tab1" href="/tabs/tab1">
          <ion-icon :icon="golf" />
          <ion-label>Range Finder</ion-label>
        </ion-tab-button>
          
        <ion-tab-button style="display: none;" tab="tab2" href="/tabs/tab2">
          <ion-icon :icon="pencil" />
          <ion-label>Score Card</ion-label>
        </ion-tab-button>
        
        <ion-tab-button tab="tab3" href="/tabs/tab3">
          <ion-icon :icon="person" />
          <ion-label>My Settings</ion-label>
        </ion-tab-button>
      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script>
import { defineComponent } from 'vue';
import { IonTabBar, IonTabButton, IonTabs, IonLabel, IonIcon, IonPage, IonRouterOutlet } from '@ionic/vue';
import { pencil, person, golf } from 'ionicons/icons';

export default defineComponent({
  name: 'TabsPage',
  components: { IonLabel, IonTabs, IonTabBar, IonTabButton, IonIcon, IonPage, IonRouterOutlet },
  setup() {
    return {
      pencil, 
      person, 
      golf,
    }
  }
});
</script>

<style scoped>
ion-tab-bar{
  bottom: 10px;
  position: relative;
  border-radius: 16px;
  width: 92%;
  margin: 0 auto;
  background-color: #a57a5a;
}

ion-tab-button{
  background-color: #a57a5a;
}

ion-tabs{
  background-color:#f5f5dd;
}

ion-icon{
  color: #f5f5dd;
}

ion-label{
  color: #f5f5dd;
}
</style>
