<template>
    <div class="flex flex-col gap-4">
        <div class="text-white">Chat</div>
        <div class="border border-white rounded-md p-4 h-[400px] overflow-y-scroll" ref="chatContainer">
            <Message v-for="message in messages" :message="message" :fromMe="session.user.id === message.user_id" />
        </div>
        <div class="flex gap-4">
            <input class="w-2/3" type="text" v-model="message" @keydown.enter="sendMessage" />
            <button class="w-1/3 button primary" @click="sendMessage">Send the message</button>
        </div>
    </div>
</template>

<script setup>
import { onMounted, ref, toRefs } from 'vue'
import { supabase } from '../supabase'
import Message from './Message.vue'

const props = defineProps(['session'])
const { session } = toRefs(props)

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
        const test = await supabase.from('messages').insert({ message: message.value, user_id: session.value.user.id })
        message.value = ''
        console.log('Message sent', test)
    } catch (error) {
        alert(error.message)
    }
}

const getMessages = async () => {
    try {
        const { data } = await supabase.from('messages').select('*, profiles(username)').limit(30)
        messages.value = data
        console.log('Retrieve messages', messages.value)
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
    async (payload) => {
        try {
            let { data, error } = await supabase
                .from('messages')
                .select(`
                    *,
                    profiles (username)
                `)
                .eq("id", payload.new.id)
                .maybeSingle();
            if (error) throw error
            console.log(payload)
            addMessage(data)
        } catch (error) {
            console.log(error)
            addMessage(payload.new)
        }
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
    console.log(session.value)
})
</script>