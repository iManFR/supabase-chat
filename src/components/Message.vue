<template>
  <div class="my-4">
    <div class="text-sm mb-1">{{ dayjs(message.created_at).format('HH : mm') }}</div>
    <div class="p-2 bg-white   rounded-md w-fit">
      <span class="text-black">{{ message.message }}</span>
    </div>
  </div>
</template>

<script setup>
import dayjs from 'dayjs';

defineProps(['message'])

</script>