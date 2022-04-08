<script setup>
import { ref } from "vue"

const log = ref('')
const emit = defineEmits(['response'])

async function redirectToSignIn() {
  emit('response', 'NewContract')
  
  try {
    const signInURL = "https://gighabit-ticketing.auth.eu-west-2.amazoncognito.com/login?client_id=kelm8fid312g68ul7h1ml9f9q&response_type=code&scope=email+openid+phone&redirect_uri=https%3A%2F%2F18.130.254.119%3A3000%2F"
    window.location.href = signInURL 
  } catch (err) {
    log.value = err
  }
}
</script>

<template>
  <h1>Gighabit Ticketing</h1>
  <h2>Welcome</h2>
  <button @click="redirectToSignIn">Sign In</button>
  <p>Welcome log: {{ log }}</p>
</template>
