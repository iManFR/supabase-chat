<template>
  <div class="my-4 flex flex-col" :class="fromMe ? 'items-end' : 'items-start'">
    <div class="text-[12px] mb-1">{{ dayjs(message.created_at).format('HH : mm') }}</div>
    <div 
      class="p-2 bg-white   rounded-md w-fit flex flex-col min-w-[100px]"
      :class="fromMe ? 'bg-green-400' : 'bg-white'"
    >
      <span class="text-black text-sm">{{ fromMe ? 'Me' : message.profiles?.username ? message.profiles?.username : 'Anonyme'  }}</span>
      <span class="text-black text-base">{{ message.message }}</span>
    </div>
  </div>
</template>

<script setup>
import dayjs from 'dayjs';

defineProps(['message', 'fromMe'])

</script>