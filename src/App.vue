<script setup>
import { ref } from "vue"
import Welcome from "./components/Welcome.vue"
import NewContract from "./components/NewContract.vue"

const log = ref('')
const currentPage = ref('Welcome')
const getUserIDTokenResponse = ref('')

const currentURL = window.location.pathname

async function getUserIDToken() {
  getUserIDTokenResponse.value = await fetch("https://gighabit-ticketing.auth.eu-west-2.amazoncognito.com/oauth2/authorize?response_type=code&client_id= kelm8fid312g68ul7h1ml9f9q&redirect_uri=https://18.130.254.119:3000/&state=STATE&scope=openid+profile+aws.cognito.signin.user.admin")
}
</script>

<template>
  <Welcome v-if="currentPage == 'Welcome'"  @response="(msg) => currentPage = msg" />
  
  <Welcome v-if="currentPage == 'Welcome'" />
  <NewContract v-if="currentPage == 'NewContract'" />
  
  <p>App current page: {{ currentPage }}</p>
  <p>App log: {{ log }}</p>
  <p>App current url: {{ currentURL }}</p>
  <p>App getUserIDTokenResponse: {{ getUserIDTokenResponse }}</p>
</template>
