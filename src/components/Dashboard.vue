<script setup>
import { computed, ref, watchEffect } from 'vue';
import UsersPanel from './UsersPanel.vue';
import CompaniesPanel from './CompaniesPanel.vue';
import AttendancePanel from './AttendancePanel.vue';
import CreateCompanyWithAdminPanel from './CreateCompanyWithAdminPanel.vue';
import CreateAdminUserPanel from './CreateAdminUserPanel.vue';
import LogsPanel from './LogsPanel.vue';
const props = defineProps({ user: Object, token: String });
const emit = defineEmits(['logout']);
const section = ref('attendance');
const showPasswordForm = ref(false);
const currentPassword = ref('');
const newPassword = ref('');
const passwordMessage = ref('');
const passwordMessageType = ref('');

const sections = computed(() => {
  const role = Number(props.user?.idrol) || 0;
  if (role === 1) {
    return [
      { key: 'companies-list', label: 'EMPRESAS LISTA' },
      { key: 'company-create', label: 'CREACION DE EMPRESA' },
      { key: 'admin-create', label: 'CREACION DE USUARIO ADMIN' },
      { key: 'users', label: 'LISTA DE USUARIOS' },
    ];
  }
  if (role === 2) {
    return [
      { key: 'users', label: 'Lista de usuarios' },
      { key: 'logs', label: 'Lista de logs' },
      { key: 'attendance', label: 'Escaner QR' },
    ];
  }
  return [{ key: 'attendance', label: 'Escaner QR' }];
});

watchEffect(() => {
  if (!sections.value.find(item => item.key === section.value)) {
    section.value = sections.value[0]?.key || 'attendance';
  }
});

async function submitPasswordChange() {
  passwordMessage.value = '';
  passwordMessageType.value = '';

  if (!currentPassword.value || !newPassword.value) {
    passwordMessageType.value = 'error';
    passwordMessage.value = 'Debe completar la contraseña actual y la nueva contraseña.';
    return;
  }

  try {
    const response = await fetch('https://proyectoqrbackend.onrender.com/api/users/me/password', {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        Authorization: `Bearer ${props.token}`,
      },
      body: JSON.stringify({
        currentPassword: currentPassword.value,
        newPassword: newPassword.value,
      }),
    });

    const result = await response.json();
    if (!response.ok) {
      passwordMessageType.value = 'error';
      passwordMessage.value = result.message || 'Error al cambiar la contraseña.';
      return;
    }

    passwordMessageType.value = 'success';
    passwordMessage.value = result.message || 'Contraseña actualizada correctamente.';
    currentPassword.value = '';
    newPassword.value = '';
  } catch (error) {
    passwordMessageType.value = 'error';
    passwordMessage.value = 'No se pudo conectar con el servidor.';
  }
}
</script>

<template>
  <div class="dashboard-layout">
    <aside class="dashboard-sidebar">
      <div class="sidebar-brand">
        <h1>Jornadas QR</h1>
        <p v-if="props.user">Empresa: {{ props.user.nombre_empresa || props.user.idempresa }}</p>
      </div>
      <nav class="dashboard-nav">
        <button
          type="button"
          v-for="item in sections"
          :key="item.key"
          :class="{ active: section === item.key }"
          @click="section = item.key"
        >
          {{ item.label }}
        </button>
      </nav>
      <button type="button" class="logout" @click="emit('logout')">Cerrar sesión</button>
      <button
        type="button"
        class="secondary"
        @click="showPasswordForm = !showPasswordForm"
      >
        {{ showPasswordForm ? 'Cancelar cambio de contraseña' : 'Cambiar contraseña' }}
      </button>
      <div v-if="showPasswordForm" class="password-change-form">
        <label>
          Contraseña actual
          <input
            type="password"
            v-model="currentPassword"
            autocomplete="current-password"
          />
        </label>
        <label>
          Nueva contraseña
          <input
            type="password"
            v-model="newPassword"
            autocomplete="new-password"
          />
        </label>
        <button type="button" @click="submitPasswordChange">Guardar nueva contraseña</button>
        <p v-if="passwordMessage" :class="passwordMessageType">{{ passwordMessage }}</p>
      </div>
    </aside>

    <div class="dashboard-main">
      <header class="dashboard-header">
        <div>
          <h2>Bienvenido, {{ props.user?.usuario }}</h2>
          <p>Rol: {{ props.user?.rol_nombre }}</p>
        </div>
      </header>

      <div class="dashboard-content">
        <CompaniesPanel v-if="section === 'companies-list'" :token="token" />
        <CreateCompanyWithAdminPanel v-if="section === 'company-create'" :token="token" />
        <CreateAdminUserPanel v-if="section === 'admin-create'" :token="token" />
        <UsersPanel v-if="section === 'users'" :token="token" :user="props.user" />
        <LogsPanel v-if="section === 'logs'" :token="token" />
        <AttendancePanel v-if="section === 'attendance'" :token="token" :user="props.user" />
      </div>
    </div>
  </div>
</template>
