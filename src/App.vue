<script setup>
import { ref, onMounted } from 'vue';
import WebAppForm from './components/WebAppForm.vue';

const chatID = ref(null);

onMounted(() => {
  if (window.Telegram && window.Telegram.WebApp) {
    const userData = window.Telegram.WebApp.initDataUnsafe;
    chatID.value = userData?.user?.id || null;
    console.log("Полученный chatID:", chatID.value);
  }
});
</script>

<template>
  <WebAppForm />
</template>

