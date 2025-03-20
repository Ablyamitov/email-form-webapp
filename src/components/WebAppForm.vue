<script setup>
import { ref } from 'vue';

const email = ref('');
const useSMTP = ref(false);
const smtpPassword = ref('');
const emailValid = ref(true);
const isEmailVerified = ref(false);
const verificationCode = ref('');
const showCodeInput = ref(false);
const url = ref('https://7dad-149-102-241-20.ngrok-free.app');

// Простая валидация email
const validateEmail = () => {
  emailValid.value = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.value);
};

// Отправка email на сервер для проверки
const sendVerificationCode = async () => {
  validateEmail();
  if (!emailValid.value) return alert("Введите корректный email!");

  try {
    const response = await fetch(url.value + "/api/send-code", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email: email.value }),
    });

    const result = await response.json();
    if (result.success) {
      showCodeInput.value = true;
      alert("Код отправлен на email!");
    } else if (result.error)  {
      alert("Ошибка отправки кода: " + result.error);
	} else {
	  alert("Ошибка отправки кода, попробуйте ещё раз");
	}
  } catch (error) {
    console.error("Ошибка:", error);
    alert("Ошибка подключения к серверу");
  }
};

// Проверка кода на сервере
const verifyCode = async () => {
  try {
    const response = await fetch(url.value + "/api/verify-code", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ email: email.value, code: verificationCode.value }),
    });

    const result = await response.json();
    if (result.success) {
      isEmailVerified.value = true;
      alert("Email подтвержден!");
    } else if (result.error != undefined){
		alert("Ошибка проверки кода: " + result.error);
	}else {
      alert("Неверный код!");
    }
  } catch (error) {
    console.error("Ошибка:", error);
    alert("Ошибка подключения к серверу");
  }
};

// Отправка формы после верификации
const submitForm = async () => {
  if (!isEmailVerified.value) return alert("Сначала подтвердите email!");

  try {
    const response = await fetch(url.value + "/api/process-email", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({
        email: email.value,
        smtp_enabled: useSMTP.value,
        smtp_password: useSMTP.value ? smtpPassword.value : null,
		user_chat_id: chatID.value
      }),
    });

    const result = await response.json();
    alert(result.message);
  } catch (error) {
    console.error("Ошибка:", error);
    alert("Ошибка подключения к серверу");
  }
};
</script>

<template>
  <div class="container">
    <h2>Настройки email</h2>

    <input type="email" v-model="email" @blur="validateEmail" placeholder="Введите email" />
    <span v-if="!emailValid" class="error">Некорректный email</span>

    <button v-if="!showCodeInput" @click="sendVerificationCode">Отправить код</button>

    <div v-if="showCodeInput">
      <input type="text" v-model="verificationCode" placeholder="Введите код" />
      <button @click="verifyCode">Подтвердить</button>
    </div>

    <div v-if="isEmailVerified">
      <label>
        <input type="checkbox" v-model="useSMTP" />
        Отправлять почту от моего имени
      </label>

      <input
        v-if="useSMTP"
        type="password"
        v-model="smtpPassword"
        placeholder="Пароль от SMTP"
      />

      <button @click="submitForm">Сохранить</button>
    </div>
  </div>
</template>

<style>
.container {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 300px;
  margin: auto;
}
.error {
  color: red;
  font-size: 12px;
}
</style>
