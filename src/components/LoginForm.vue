<script setup>
import { ref } from 'vue';
const emit = defineEmits(['login']);
const email = ref('');
const password = ref('');
const error = ref('');
const loading = ref(false);
const mode = ref('login'); // 'login' o 'create-admin'
const adminUser = ref('');
const adminEmail = ref('');
const adminPassword = ref('');
const success = ref('');

async function submit() {
  error.value = '';
  loading.value = true;
  try {
    const response = await fetch('https://proyectoqrbackend.onrender.com/api/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email: email.value, password: password.value }),
    });
    const data = await response.json();
    if (!response.ok) {
      error.value = data.message || 'Error en el login';
      return;
    }
    emit('login', data);
  } catch (err) {
    error.value = 'No se pudo conectar con el servidor';
  } finally {
    loading.value = false;
  }
}

async function createAdmin() {
  error.value = '';
  success.value = '';
  loading.value = true;

  if (!adminUser.value || !adminEmail.value || !adminPassword.value) {
    error.value = 'Todos los campos son obligatorios';
    loading.value = false;
    return;
  }

  try {
    const response = await fetch('https://proyectoqrbackend.onrender.com/api/api/users/first-admin/create', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        usuario: adminUser.value,
        email: adminEmail.value,
        password: adminPassword.value,
      }),
    });
    const data = await response.json();
    if (!response.ok) {
      error.value = data.message || 'Error al crear administrador';
      loading.value = false;
      return;
    }
    success.value = 'Administrador creado correctamente. Ya puedes iniciar sesión.';
    adminUser.value = '';
    adminEmail.value = '';
    adminPassword.value = '';
    setTimeout(() => {
      mode.value = 'login';
      success.value = '';
    }, 2000);
  } catch (err) {
    error.value = 'No se pudo conectar con el servidor';
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <div class="card auth-card">
    <template v-if="mode === 'login'">
      <h2>Iniciar sesión</h2>
      <label>
        Correo
        <input type="email" v-model="email" autocomplete="username" />
      </label>
      <label>
        Contraseña
        <input type="password" v-model="password" autocomplete="current-password" />
      </label>
      <button @click.prevent="submit" :disabled="loading">{{ loading ? 'Cargando...' : 'Entrar' }}</button>
      <p class="error" v-if="error">{{ error }}</p>
      <p class="hint">Usa las credenciales de administrador o supervisor.</p>
      <hr />
      <p class="create-admin-text">¿Es la primera vez? Crea un usuario administrador:</p>
      <button class="secondary" @click.prevent="mode = 'create-admin'">Crear primer admin</button>
    </template>

    <template v-else-if="mode === 'create-admin'">
      <h2>Crear administrador</h2>
      <p class="panel-description">Crea el primer usuario administrador del sistema.</p>
      <label>
        Nombre de usuario
        <input v-model="adminUser" placeholder="Nombre del admin" />
      </label>
      <label>
        Email
        <input type="email" v-model="adminEmail" placeholder="admin@dominio.com" />
      </label>
      <label>
        Contraseña
        <input type="password" v-model="adminPassword" placeholder="Contraseña segura" />
      </label>
      <button @click.prevent="createAdmin" :disabled="loading">{{ loading ? 'Creando...' : 'Crear admin' }}</button>
      <p class="success" v-if="success">{{ success }}</p>
      <p class="error" v-if="error">{{ error }}</p>
      <button class="secondary" @click.prevent="mode = 'login'">Volver a login</button>
    </template>
  </div>
</template>

<style scoped>
.auth-card {
  max-width: 400px;
  margin: 0 auto;
}

.create-admin-text {
  margin-top: 20px;
  text-align: center;
  color: var(--text-secondary, #666);
  font-size: 0.9em;
}

.secondary {
  background-color: var(--button-secondary, #f0f0f0);
  color: var(--text-primary, #333);
  border: 1px solid var(--border-color, #ddd);
}

.secondary:hover {
  background-color: var(--button-secondary-hover, #e0e0e0);
}

hr {
  margin: 20px 0;
  border: none;
  border-top: 1px solid var(--border-color, #ddd);
}

.panel-description {
  color: var(--text-secondary, #666);
  margin-bottom: 20px;
  font-size: 0.9em;
}

.success {
  color: #28a745;
  margin-top: 10px;
}

.error {
  color: #dc3545;
  margin-top: 10px;
}
</style>
