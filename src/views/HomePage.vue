<template>
  <ion-page>
    <ion-header :translucent="true">
      <ion-toolbar>
        <ion-title>Inbox</ion-title>
      </ion-toolbar>
    </ion-header>
    
    <ion-content :fullscreen="true">
      <ion-refresher slot="fixed" @ionRefresh="refresh($event)">
        <ion-refresher-content></ion-refresher-content>
      </ion-refresher>
      
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Inbox</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-button @click="scan" @disabled="scanning" expand="block">Scan</ion-button>

      <ion-button @click="connect" expand="block">Connect</ion-button>
      
      <ion-list>
        <ion-label v-for="device in results" :key="device.uuids">{{ device.localName }}</ion-label>
        <!-- <MessageListItem v-for="message in messages" :key="message.id" :message="message" /> -->
      </ion-list>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { IonContent, IonHeader, IonList, IonPage, IonRefresher, IonRefresherContent, IonTitle, IonToolbar } from '@ionic/vue';
import { BleClient, ScanResult } from '@capacitor-community/bluetooth-le';
import { OilerBluetoothLeService } from 'oiler-capacitor-ble-sdk';
import MessageListItem from '@/components/MessageListItem.vue';
import { defineComponent, ref, onMounted } from 'vue';
import { getMessages } from '@/data/messages';

export default defineComponent({
  name: 'HomePage',
  data() {
    return {
      messages: getMessages()
    }
  },
  setup() {
    const results = ref<ScanResult[]>([])
    const scanning = ref(false)
    const error = ref('')

    const init = async () => {
      try {
        await BleClient.initialize()
      } catch (err) {
        error.value = (err as Error).message
      }
    }

    const scan = async () => {
      results.value = []
      scanning.value = true

      try {
        await BleClient.requestLEScan({}, result => {
          results.value.push(result)
        })
        setTimeout(() => {
          void BleClient.stopLEScan().then(() => {
            scanning.value = false
          })
        }, 10000)
      } catch (err) {
        error.value = (err as Error).message
      }
    }

    const connect = async () => {
      try {
        await OilerBluetoothLeService.connect();
      } catch (err) {
        error.value = (err as Error).message
      }
    }

    onMounted(init)

    return { results, scanning, error, scan, connect };
  },
  methods: {
    refresh: (ev: CustomEvent) => {
      setTimeout(() => {
        ev.detail.complete();
      }, 3000);
    }
  },
  components: {
    IonContent,
    IonHeader,
    IonList,
    IonPage,
    IonRefresher,
    IonRefresherContent,
    IonTitle,
    IonToolbar,
    // MessageListItem
  },
});
</script>
