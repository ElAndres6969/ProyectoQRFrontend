<script setup>
import { ref } from 'vue';
const props = defineProps({ token: String });
const usuario = ref('');
const email = ref('');
const password = ref('');
const idempresa = ref('');
const success = ref('');
const error = ref('');

async function submit() {
  error.value = '';
  success.value = '';
  if (!usuario.value || !email.value || !password.value || !idempresa.value) {
    error.value = 'Todos los campos son obligatorios';
    return;
  }

  try {
    const res = await fetch('http://localhost:4000/api/users', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${props.token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        usuario: usuario.value,
        email: email.value,
        password: password.value,
        idempresa: 1,
        idrol: 1,
      }),
    });
    const data = await res.json();
    if (!res.ok) throw data;
    success.value = 'Usuario administrador creado correctamente';
    usuario.value = '';
    email.value = '';
    password.value = '';
    idempresa.value = '';
  } catch (err) {
    error.value = err.message || 'Error al crear el usuario admin';
  }
}
</script>

<template>
  <section class="panel">
    <h2>Creación de usuario admin</h2>
    <p class="panel-description">Crea un usuario administrador con rol admin para el sistema.</p>

    <div class="panel-card">
      <label>
        Nombre de usuario
        <input v-model="usuario" placeholder="Nombre del admin" />
      </label>
      <label>
        Email
        <input type="email" v-model="email" placeholder="admin@dominio.com" />
      </label>
      <label>
        Contraseña
        <input type="password" v-model="password" placeholder="Contraseña segura" />
      </label>
      <button @click.prevent="submit">Crear admin</button>
      <p class="success" v-if="success">{{ success }}</p>
      <p class="error" v-if="error">{{ error }}</p>
    </div>
  </section>
</template>
