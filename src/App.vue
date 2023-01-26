<template>
  <div class="h-screen flex items-center p-8">
    <div class="flex gap-8 w-screen" v-if="session">
      <Chat class="w-1/2" :session="session" />
      <Account class="w-1/2" :session="session" />
    </div>
    <Auth v-else />
  </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import Account from './components/Account.vue'
import Auth from './components/Auth.vue'
import Chat from './components/Chat.vue'
import { supabase } from './supabase'

const session = ref()

onMounted(() => {
  supabase.auth.getSession().then(({ data }) => {
    session.value = data.session
  })

  supabase.auth.onAuthStateChange((_, _session) => {
    session.value = _session
  })
  console.log(session.value)
})
</script>