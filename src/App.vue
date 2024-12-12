<template>
  <div id="app">
    <div id="g_id_onload" :data-client_id="googleClientId" data-callback="handleCredentialResponse">
    </div>
    <div class="g_id_signin" data-type="standard"></div>
    <div v-if="accessToken">
      <p>
        Access token:<br> <span style="word-break: break-all;">{{ accessToken }}</span>
      </p>
      <ul>
        <p>{{ todos }}</p>
        <!-- iterate over list of todos: type: list[str] -->
        <li v-for="todo in todos" :key="todo">
          {{ todo }}
        </li>
      </ul>
    </div>
    <div v-else>
      No access token
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import { GOOGLE_CLIENT_ID } from './config';

export default {
  data() {
    return {
      accessToken: null,
      googleClientId: GOOGLE_CLIENT_ID,
      todos: [],
    };
  },
  created() {
    this.accessToken = localStorage.getItem('accessToken');
    this.todos = [];
    if (this.accessToken) {
      this.fetchTodos();
    }
  },
  mounted(){
    window.handleCredentialResponse = this.handleCredentialResponse;
    // const script = document.createElement('script');
    // script.src = 'https://accounts.google.com/gsi/client';
    // script.async = true;
    // document.body.appendChild(script);
  },
  methods: {
    handleCredentialResponse(response) {
      console.log("Encoded JWT ID token: " + response.credential);
      // Decode the JWT token to get user information
      const userObject = JSON.parse(atob(response.credential.split('.')[1]));
      console.log("User object:", userObject);

      const mockGoogleResponse = {
        email: userObject.email,
        name: userObject.name,
      };
      console.log("mockGoogleResponse:", mockGoogleResponse);

      axios.post('http://localhost:8000/login/google', mockGoogleResponse)
        .then(response => {
          console.log("Login response:", response.data);
          this.accessToken = response.data.access_token;
          this.fetchTodos();
          localStorage.setItem('accessToken', response.data.access_token);
        })
        .catch(err => {
          console.error("Error posting login data:", err);
        });
    },
    async fetchTodos() {
      const response = await axios.get('http://localhost:8000/todos', {
        headers: { Authorization: `Bearer ${this.accessToken}` },
      });
      this.todos = response.data
      console.log("Todos:", this.todos);
    },
  },
};
</script>
<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  text-align: center;
  align-items: center;
  width: 70vh;
  margin: 10vh auto;
}
</style>
