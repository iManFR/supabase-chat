<template>
    <div class="flex flex-col gap-4">
        <div class="text-white">Chat</div>
        <div class="border border-white rounded-md p-4 h-[400px] overflow-y-scroll" ref="chatContainer">
            <Message v-for="message in messages" :message="message" />
        </div>
        <div class="flex gap-4">
            <input class="w-2/3" type="text" v-model="message" @keydown.enter="sendMessage" />
            <button class="w-1/3 button primary" @click="sendMessage">Send the message</button>
        </div>
    </div>
</template>

<script setup>
import { onMounted, ref } from 'vue'
import { supabase } from '../supabase'
import Message from './Message.vue'

const message = ref('')
const messages = ref([])

const chatContainer = ref(null)

const resetChatPosition = () => {
    setTimeout(() => {
        chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
    }, 100)
}

const addMessage = (message) => {
    messages.value.push(message)
    resetChatPosition()
}

const sendMessage = async () => {
    if (!message.value) return
    try {
        const { body } = await supabase.from('messages').insert({ message: message.value }).single()
        message.value = ''
    } catch (error) {
        alert(error.message)
    }
}

const getMessages = async () => {
    try {
        const { data } = await supabase.from('messages').select('*').limit(30)
        messages.value = data
        console.log(messages.value)
    } catch (error) {
        alert(error.message)
    }
}


const subscribeNewMessages = async () => {
    const channel = supabase.channel('db-messages')

    channel.on(
    'postgres_changes',
    {
        event: 'INSERT',
        schema: 'public',
        table: 'messages',
    },
    (payload) => {
        console.log(payload)
        addMessage(payload.new)
    })
    
    channel.subscribe(async (status) => {
        if (status === 'SUBSCRIBED') {
            console.log('subscribed to messages')
        }
    })
}

onMounted(async () => {
    await getMessages()
    subscribeNewMessages()
    resetChatPosition()
})
</script>