<script setup>
import { ref, onMounted } from 'vue';
import LoginForm from './components/LoginForm.vue';
import Dashboard from './components/Dashboard.vue';

const token = ref(localStorage.getItem('token') || '');
const user = ref(JSON.parse(localStorage.getItem('user') || 'null'));
const error = ref('');

async function loadCurrentUser() {
  if (!token.value) return;

  try {
    const response = await fetch('https://proyectoqrbackend.onrender.com/api/api/auth/me', {
      headers: {
        Authorization: `Bearer ${token.value}`,
      },
    });
    const data = await response.json();
    if (response.ok && data.user) {
      user.value = data.user;
      localStorage.setItem('user', JSON.stringify(data.user));
    }
  } catch (err) {
    console.error('Error cargando usuario actual:', err);
  }
}

function handleLogin(data) {
  token.value = data.token;
  user.value = data.user;
  localStorage.setItem('token', data.token);
  localStorage.setItem('user', JSON.stringify(data.user));
}

function logout() {
  token.value = '';
  user.value = null;
  localStorage.removeItem('token');
  localStorage.removeItem('user');
}

onMounted(loadCurrentUser);
</script>

<template>
  <div id="app">
    <div class="app-shell">
      <template v-if="!token || !user">
        <LoginForm @login="handleLogin" />
      </template>
      <template v-else>
        <Dashboard :token="token" :user="user" @logout="logout" />
      </template>
    </div>
  </div>
</template>
