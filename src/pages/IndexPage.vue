<template>
  <q-page class="bg-grey-2" >
    <div class="row">
      <div class="col-md-3 q-pa-lg ">
         <div class="bg-white q-pa-md rounded-borders">
           <div class="text-bold">
             Pusher Config:
           </div>
           <q-form>
             <q-input v-model="config.key" label="Key" outlined class="q-ma-sm"  dense/>
             <q-input v-model="config.wsHost" label="wsHost" outlined class="q-ma-sm" dense/>
             <q-input v-model="config.wsPort" label="wsPort" outlined type="number"  class="q-ma-sm" dense/>
             <q-input v-model="config.wssPort" label="wssPort" outlined type="number"  class="q-ma-sm" dense/>
             <q-checkbox
               v-model="config.forceTLS"
               label="forceTLS"
               class="q-ma-sm"
               dense
             />
             <div>
               <div class="flex justify-between q-px-sm" >
                 <div class="">
                   Enabled Transports
                 </div>
                 <q-btn
                   @click="config.enabledTransports = ['ws', 'wss']"
                   label="Reset"
                   color="primary"
                   dense
                   no-caps
                 />
               </div>
               <div class="q-pa-md bg-grey-2">
                 <div v-for="(transport, index) in config.enabledTransports" :key="index" class="flex justify-between">
                   <q-input v-model="config.enabledTransports[index]" outlined class="q-ma-sm flex-grow" dense />

                   <q-btn
                     @click="config.enabledTransports.splice(index, 1)"
                     color="negative"
                     flat
                     icon="delete"
                     dense
                     no-caps
                   />
                 </div>
               </div>
             </div>
             <!--          // auth url-->
             <q-input v-model="config.authEndpoint" label="authEndpoint" outlined class="q-ma-sm" />

             <div>
               <div class="flex justify-between  items-center q-px-sm" >
                 Auth Headers

                 <q-btn
                   @click="config.auth.headers = {
                  headers: {
                    Authorization: 'Bearer ' + 'token'
                  }
                }"
                   label="Reset"
                   color="primary"
                   dense
                   no-caps
                 />
                 <q-btn
                   @click="headerDialog = true"
                   label="Add New"
                   color="dark"
                   dense
                   no-caps
                 />

               </div>
               <template v-if="config?.auth?.headers">
                 <div  v-for="key in Object.keys(config.auth.headers)" :key="key" class="flex justify-between items-center q-px-sm">
                   <div>{{ key }}</div>
                   <q-input
                     v-model="config.auth.headers[key]"
                     outlined
                     class="q-ma-sm w-full"
                     dense
                   />
                   <q-btn
                     @click="delete config.auth.headers[key]"
                     color="negative"
                     flat
                     icon="delete"
                     dense
                   />
                 </div>
               </template>
             </div>
             <div>
               <q-btn
                 v-if="!connectionStatus"
                 :loading="connecting"
                 @click="initPusher"
                 label="Connect"
                 color="primary"
               />
               <q-btn
                 v-else
                 @click="disconnect()"
                 label="Disconnect"
                 color="negative"
               />
             </div>
           </q-form>
         </div>
      </div>
      <div class="col-md-6 q-pa-lg">
        <q-card  flat>
          <q-card-section>
            <div >
              <span class="text-bold q-mr-sm">Connection Status :</span>
              <span :class=" connectionStatus ? 'text-green' : 'text-red'">{{ connectionStatus ? 'Connected' : 'Disconnected' }}</span>
            </div>
            <div class="flex items-center" v-if="connectionStatus">
              <span class="text-bold q-mr-sm">Channels :</span>
              <q-input
                v-model="newChannelForm.name" label="Channel Name" outlined class="q-ma-sm"
                dense
              />
              <q-input
                v-model="newChannelForm.eventName" label="Event Name" outlined class="q-ma-sm"
                dense
              />
              <q-select
                v-model="newChannelForm.type" label="Channel Type" outlined class="q-ma-sm"
                :options="[
                  {label: 'Public', value: 'public'},
                  {label: 'Private', value: 'private'},
                ]"
                map-options
                emit-value
                dense
              />

              <q-btn
                @click="()=>{
                  if(newChannelForm.type === 'public'){
                    joinChannel(newChannelForm.name,newChannelForm.eventName)
                  }else{
                    privateChannel(newChannelForm.name,newChannelForm.eventName)
                  }
                  newChannelForm.name = ''

                }"
                label="Add New"
                color="dark"
                dense
                no-caps
              />
            </div>
            <div class="flex items-center" v-if="activeChannels.length">
              <span class="text-bold q-mr-sm"> Active Channels :</span>
              <div v-for="(channel, index) in activeChannels" :key="index" class="flex items-center q-ma-sm">
                <span>{{ channel }}</span>
                <q-btn
                  @click="leaveChannel(channel)"
                  color="negative"
                  flat
                  icon="delete"
                  dense
                  no-caps
                />
              </div>
            </div>
            <div v-if="activeChannels.length">
              <span class="text-bold q-mr-sm"> Events:</span>
               <div class="bg-black rounded-borders q-pa-lg text-green " style="min-height:5rem;  max-height: 30rem;overflow-y: scroll;overflow-x: hidden" id="event_message_connector">
                  <div v-for="(message, index) in eventMessages" :key="index" class="q-mb-md">
                    <div class="text-bold">{{ message.channel }}</div>
                    <div class="text-grey">{{ message.event }}</div>
                    <div class="text-white">{{ message.data }}</div>
                  </div>
               </div>
            </div>
          </q-card-section>
        </q-card>
      </div>
      <div class="col-md-3 q-pa-lg">
        <q-card flat>
          <q-card-section>
            <div class="text-bold">
              Current Event
            </div>
            <div class="q-mt-md" v-if="currentEventMessage">
              <div class="text-bold">Channel:</div>
              <div>{{ currentEventMessage?.channel }}</div>
              <div class="text-bold">Event:</div>
              <div>{{ currentEventMessage?.event }}</div>
              <div class="text-bold">Data:</div>
              <div>{{ currentEventMessage?.data }}</div>
            </div>
          </q-card-section>
        </q-card>
      </div>
    </div>
  </q-page>
  <q-dialog v-model="headerDialog">
    <q-card>
      <q-card-section>
        <div>
          <div class="flex justify-between" >
            Auth Headers

            <q-btn
              @click="headerDialog = false"
              label="Close"
              color="negative"
            />
          </div>
          <div>
            <q-input v-model="authHeaderFrom.key" label="Key" outlined class="q-ma-sm" />
            <q-input v-model="authHeaderFrom.value" label="Value" outlined class="q-ma-sm" />
            <q-btn
              @click="()=>{
                config.auth.headers[authHeaderFrom.key] = authHeaderFrom.value
                headerDialog = false
              }"
              label="Add"
              color="primary"
            />
          </div>
        </div>
      </q-card-section>
    </q-card>
  </q-dialog>
</template>

<script setup>
import Echo from "laravel-echo";

import Pusher from "pusher-js";
import {computed, onMounted, ref, watch} from "vue";

window.Pusher = Pusher;

const config = ref(JSON.parse(localStorage.getItem('config')?? JSON.stringify({
  key: '',
  wsHost: '',
  wsPort: 80,
  wssPort: 443,
  forceTLS: false,
  enabledTransports: ['ws', 'wss'],
  authEndpoint: '/broadcasting/auth',
  auth: {
    headers: {
      Authorization: 'Bearer ' + 'token'
    }
  },
  cluster: 'mt1',

})))
const headerDialog = ref(false)
const authHeaderFrom = ref({
  key: '',
  value: ''
})
const newChannelForm = ref({
  name: '',
  eventName: '',
  type: 'public'
})
const connectionStatus = ref(false)
const connecting = ref(false)
const activeChannels = ref([
])
const eventMessages =ref([]);
const currentEventMessage = ref({})
const transportDialog = ref(false)
watch(config, (value) => {
  localStorage.setItem('config', JSON.stringify(value))
},{
  deep: true
})

const  initPusher = ()=>{
  connecting.value = true
  window.Echo = new Echo({
    broadcaster: "pusher",
    key: config.value.key,
    wsHost: config.value.wsHost,
    wsPort: config.value.wsPort,
    wssPort: config.value.wssPort,
    forceTLS: config.value.forceTLS,
    enabledTransports: config.value.enabledTransports,
    cluster: 'mt1',
    authEndpoint: config.value.authEndpoint,
    auth: config.value.auth,
  });
 // on pusher connection
  window.Echo?.connector.pusher.connection.bind('connected', () => {
    connectionStatus.value = true
    connecting.value = false
  });
  // on pusher disconnection
  window.Echo?.connector.pusher.connection.bind('disconnected', () => {
    connectionStatus.value = false
    connecting.value = false
  });
  // privateChannel('chat-notification.1633525','.App\\Events\\ChatNotificationEvent')
}

const disconnect = ()=>{
  window.Echo.connector.pusher.disconnect()
}
const joinChannel = (name,eventName='*') => {
  // listen to channel on any event
  window.Echo.channel(name)
    .listen(eventName, (e) => {
      console.log('event',e)
      const data = {
        channel: name,
        event: eventName,
        data: e
      }
      eventMessages.value.push(data)
      currentEventMessage.value = data
  });
  activeChannels.value.push(name)
}
const privateChannel = (name,eventName='*') => {
  window.Echo.private(name)
    .listen(eventName, (e) => {
      console.log('event',e)
      const data = {
        channel: name,
        event: eventName,
        data: e
      }
      eventMessages.value.push(data)
      currentEventMessage.value = data
  });
  activeChannels.value.push(name)
}
const leaveChannel = (name) => {
  window.Echo.leave(name);
  activeChannels.value = activeChannels.value.filter(channel => channel !== name)
}


onMounted(()=>{
  console.log('mounted',config.value)
})
defineOptions({
  name: 'IndexPage'
});

</script>
