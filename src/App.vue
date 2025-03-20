<script setup>
import { ref, onMounted } from 'vue';
import WebAppForm from './components/WebAppForm.vue';

const chatID = ref(null);

onMounted(() => {
  if (window.Telegram?.WebApp?.initDataUnsafe?.user) {
    chatID.value = window.Telegram.WebApp.initDataUnsafe.user.id;
    alert("Chat ID:", chatID.value);
  } else {
    console.warn("Не удалось получить chat_id");
  }
});
</script>

<template>
  <WebAppForm />
</template>

