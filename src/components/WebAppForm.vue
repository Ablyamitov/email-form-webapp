<script setup>
import { ref, onMounted } from 'vue';

const email = ref('');
const useSMTP = ref(false);
const smtpPassword = ref('');
const emailValid = ref(true);
const isEmailVerified = ref(false);
const verificationCode = ref('');
const showCodeInput = ref(false);
const url = ref('https://8fad-188-191-29-81.ngrok-free.app');
const chatID = ref(null);

onMounted(() => {
  if (window.Telegram?.WebApp?.initDataUnsafe?.user) {
    chatID.value = window.Telegram.WebApp.initDataUnsafe.user.id;
  } 
});

const validateEmail = () => {
  emailValid.value = /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email.value);
};

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
    } else {
      alert("Ошибка отправки кода: " + (result.error || "Попробуйте ещё раз"));
    }
  } catch (error) {
    console.error("Ошибка:", error);
    alert("Ошибка подключения к серверу");
  }
};

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
    } else {
      alert("Ошибка проверки кода: " + (result.error || "Неверный код!"));
    }
  } catch (error) {
    console.error("Ошибка:", error);
    alert("Ошибка подключения к серверу");
  }
};

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
    <div class="card">
      <h2>Настройка почты</h2>

      <div class="input-group">
        <input type="email" v-model="email" @blur="validateEmail" placeholder="Введите email" />
        <span v-if="!emailValid" class="error">Некорректный email</span>
      </div>

      <button v-if="!showCodeInput" class="btn primary" @click="sendVerificationCode">Отправить код</button>

      <div v-if="showCodeInput" class="input-group">
        <input type="text" v-model="verificationCode" placeholder="Введите код" />
        <button class="btn success" @click="verifyCode">Подтвердить</button>
      </div>

      <div v-if="isEmailVerified">
        <label class="checkbox">
          <input type="checkbox" v-model="useSMTP" />
          Отправлять почту от моего имени
        </label>

        <input v-if="useSMTP" type="password" v-model="smtpPassword" placeholder="Пароль от SMTP" />

        <button class="btn save" @click="submitForm">Сохранить</button>
      </div>
    </div>
  </div>
</template>

<style>
/* Общие стили */
.container {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  height: 100vh;
  width: 350px;
  background: #f5f5f5;
}

/* Карточка */
.card {
  background: #fff;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0px 4px 10px rgba(0, 0, 0, 0.1);
  max-width: 350px;
  width: 100%;
  text-align: center;
}

/* Заголовок */
h2 {
  margin-bottom: 15px;
  font-size: 20px;
  color: #333;
}

/* Поля ввода */
.input-group {
  margin-bottom: 8px;
}

input {
  width: 100%;
  padding: 10px;
  border: 2px solid #ccc;
  border-radius: 8px;
  font-size: 16px;
  transition: 0.3s;
}

input:focus {
  border-color: #007bff;
  outline: none;
}

/* Ошибки */
.error {
  color: red;
  font-size: 12px;
  margin-top: 5px;
}


/* Чекбокс */
.checkbox {
  display: flex;
  align-items: center;
  justify-content: flex-start;
  gap: 8px;
  font-size: 14px;
  margin-bottom: 10px;
  width: 100%;
  color: #333;
}

.checkbox input {
  width: 16px;
  height: 16px;
  margin: 10px;
}



/* Кнопки */
.btn {
  width: 100%;
  margin-top: 15px;
  padding: 10px;
  border-radius: 8px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
  border: none;
  color: white;
}

.btn.primary {
  background: #007bff;
}

.btn.success {
  background: #28a745;
}

.btn.save {
	margin-top: 5px;
  background: #ff9800;
}

.btn:hover {
  opacity: 0.8;
}
</style>
