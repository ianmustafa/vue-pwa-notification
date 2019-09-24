<template>
  <div class="hello">
    <h1>{{ msg }}</h1>
    <h3>Firebase Cloud Messaging Token</h3>
    <code>{{ fcmToken }}</code>
  </div>
</template>

<script>
import firebase from 'firebase/app';
import 'firebase/messaging';

export default {
  name: 'HelloWorld',
  props: {
    msg: String,
  },
  data() {
    return {
      fcmToken: '',
    };
  },
  async mounted() {
    const config = {
      apiKey: process.env.VUE_APP_FCM_API_KEY,
      authDomain: process.env.VUE_APP_FCM_AUTH_DOMAIN,
      databaseURL: process.env.VUE_APP_FCM_DATABASE_URL,
      projectId: process.env.VUE_APP_FCM_PROJECT_ID,
      storageBucket: process.env.VUE_APP_FCM_STORAGE_BUCKET,
      messagingSenderId: process.env.VUE_APP_FCM_SENDER_ID,
      appId: process.env.VUE_APP_FCM_APP_ID,
      measurementId: process.env.VUE_APP_FCM_MEASUREMENT_ID,
    };
    firebase.initializeApp(config);

    const messaging = firebase.messaging();

    messaging.usePublicVapidKey(process.env.VUE_APP_FCM_CERT_KEY_PAIR);

    try {
      await messaging.requestPermission();
    } catch (err) {
      // eslint-disable-next-line no-console
      console.error('Unable to get permission to notify.', err);
    }

    try {
      const currentToken = await messaging.getToken();
      this.fcmToken = currentToken;
    } catch (err) {
      // eslint-disable-next-line no-console
      console.error('Unable to retrieve token.', err);
    }

    const vm = this;
    messaging.onTokenRefresh(async () => {
      try {
        const refreshedToken = await messaging.getToken();
        // eslint-disable-next-line no-console
        console.log('Token refreshed.');
        vm.fcmToken = refreshedToken;
      } catch (err) {
        // eslint-disable-next-line no-console
        console.error('Unable to retrieve refreshed token.', err);
      }
    });
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
