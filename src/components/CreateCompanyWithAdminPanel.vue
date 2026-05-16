<script setup>
import { ref } from 'vue';
const props = defineProps({ token: String });
const companyName = ref('');
const adminName = ref('');
const adminEmail = ref('');
const adminPassword = ref('');
const success = ref('');
const error = ref('');

async function submit() {
  error.value = '';
  success.value = '';
  if (!companyName.value || !adminName.value || !adminEmail.value || !adminPassword.value) {
    error.value = 'Todos los campos son obligatorios';
    return;
  }

  try {
    const res = await fetch('https://proyectoqrbackend.onrender.com/api/api/companies/with-admin', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${props.token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nombre_empresa: companyName.value,
        usuario: adminName.value,
        email: adminEmail.value,
        password: adminPassword.value,
      }),
    });
    const data = await res.json();
    if (!res.ok) throw data;
    success.value = 'Empresa y administrador de empresa creados correctamente';
    companyName.value = '';
    adminName.value = '';
    adminEmail.value = '';
    adminPassword.value = '';
  } catch (err) {
    error.value = err.message || 'Error al crear empresa y administrador';
  }
}
</script>

<template>
  <section class="panel">
    <h2>Creación de empresa</h2>
    <p class="panel-description">Crea la empresa y un usuario administrador de empresa (rol 2) en el mismo paso.</p>

    <div class="panel-card">
      <label>
        Nombre de empresa
        <input v-model="companyName" placeholder="Ej. Compañía S.A." />
      </label>
      <label>
        Nombre administrador de empresa
        <input v-model="adminName" placeholder="Nombre del supervisor" />
      </label>
      <label>
        Email administrador
        <input type="email" v-model="adminEmail" placeholder="admin@empresa.com" />
      </label>
      <label>
        Contraseña
        <input type="password" v-model="adminPassword" placeholder="Contraseña segura" />
      </label>
      <button @click.prevent="submit">Crear empresa y usuario empresa</button>
      <p class="success" v-if="success">{{ success }}</p>
      <p class="error" v-if="error">{{ error }}</p>
    </div>
  </section>
</template>
