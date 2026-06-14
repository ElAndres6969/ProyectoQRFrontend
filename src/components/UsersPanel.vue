<script setup>
import { ref, watch, watchEffect, nextTick } from 'vue';
const props = defineProps({ token: String, user: Object });
const users = ref([]);
const error = ref('');
const success = ref('');
const selectedUser = ref(null);
const formContainer = ref(null);
const searchName = ref('');
const searchEmail = ref('');
const companyQuery = ref('');
const companyOptions = ref([]);
const companyLoading = ref(false);
const form = ref({ usuario: '', email: '', password: '', idempresa: props.user?.idempresa || null, idrol: 3 });
let userTimeoutId;
let companyTimeoutId;

async function loadUsers() {
  error.value = '';
  try {
    const params = new URLSearchParams();
    if (searchName.value.trim()) params.append('name', searchName.value.trim());
    if (searchEmail.value.trim()) params.append('email', searchEmail.value.trim());
    if (companyQuery.value.trim()) params.append('company', companyQuery.value.trim());

    const url = `https://proyectoqrbackend.onrender.com/api/users${params.toString() ? `?${params.toString()}` : ''}`;
    const res = await fetch(url, {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    users.value = data.users;
  } catch (err) {
    error.value = err.message || 'No se pudieron cargar los usuarios';
  }
}

function scheduleUserSearch() {
  clearTimeout(userTimeoutId);
  userTimeoutId = setTimeout(loadUsers, 250);
}

async function loadCompanyOptions() {
  const query = companyQuery.value.trim();
  if (!query) {
    companyOptions.value = [];
    return;
  }

  companyLoading.value = true;
  try {
    const res = await fetch(`https://proyectoqrbackend.onrender.com/api/companies?search=${encodeURIComponent(query)}`, {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    companyOptions.value = data.companies || [];
  } catch {
    companyOptions.value = [];
  } finally {
    companyLoading.value = false;
  }
}

function scheduleCompanySearch() {
  clearTimeout(companyTimeoutId);
  scheduleUserSearch();
  if (props.user?.idrol !== 1) {
    return;
  }
  companyTimeoutId = setTimeout(loadCompanyOptions, 250);
}

function selectCompany(company) {
  companyQuery.value = company.nombre_empresa;
  companyOptions.value = [];
  scheduleUserSearch();
}

function resetFilters() {
  searchName.value = '';
  searchEmail.value = '';
  companyQuery.value = '';
  companyOptions.value = [];
  success.value = '';
  error.value = '';
  loadUsers();
}

function resetForm() {
  selectedUser.value = null;
  form.value = { usuario: '', email: '', password: '', idempresa: props.user?.idempresa || null, idrol: 3 };
  error.value = '';
  success.value = '';
}

function startEdit(userItem) {
  selectedUser.value = userItem;
  form.value = {
    usuario: userItem.usuario,
    email: userItem.email,
    password: '',
    idempresa: userItem.idempresa,
    idrol: userItem.idrol,
  };
  error.value = '';
  success.value = '';
}

async function openEditAndFocus(userItem) {
  startEdit(userItem);
  await nextTick();
  try {
    if (formContainer.value && formContainer.value.scrollIntoView) {
      formContainer.value.scrollIntoView({ behavior: 'smooth', block: 'center' });
    }
    const firstInput = formContainer.value && formContainer.value.querySelector('input');
    if (firstInput) firstInput.focus();
  } catch (e) {
    // ignore
  }
}

async function saveUser() {
  error.value = '';
  success.value = '';
  try {
    const url = selectedUser.value
      ? `https://proyectoqrbackend.onrender.com/api/users/${selectedUser.value.idusuario}`
      : 'https://proyectoqrbackend.onrender.com/api/users';

    const method = selectedUser.value ? 'PUT' : 'POST';
    const res = await fetch(url, {
      method,
      headers: {
        Authorization: `Bearer ${props.token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(form.value),
    });
    const data = await res.json();
    if (!res.ok) throw data;
    await loadUsers();
    resetForm();
    success.value = selectedUser.value ? 'Usuario actualizado' : 'Usuario creado';
  } catch (err) {
    error.value = err.message || 'No se pudo guardar el usuario';
  }
}

async function deleteUser(id) {
  if (!confirm('¿Eliminar este usuario?')) return;
  error.value = '';
  success.value = '';
  try {
    const res = await fetch(`https://proyectoqrbackend.onrender.com/api/users/${id}`, {
      method: 'DELETE',
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    await loadUsers();
    if (selectedUser.value?.idusuario === id) resetForm();
    success.value = 'Usuario eliminado';
  } catch (err) {
    error.value = err.message || 'No se pudo eliminar el usuario';
  }
}

watch([searchName, searchEmail, companyQuery], scheduleUserSearch);

watchEffect(() => {
  if (props.token) {
    loadUsers();
  }
});
</script>

<template>
  <section class="panel">
    <h2>Gestión de usuarios</h2>
    <p class="panel-description">Administradores y supervisores pueden crear y editar usuarios.</p>

    <div v-if="props.user?.idrol === 1 || props.user?.idrol === 2" class="panel-card" ref="formContainer">
      <h3>{{ selectedUser ? 'Editar usuario' : 'Crear usuario' }}</h3>
      <label>
        Nombre
        <input v-model="form.usuario" />
      </label>
      <label>
        Email
        <input type="email" v-model="form.email" />
      </label>
      <label>
        Contraseña
        <input type="password" v-model="form.password" />
      </label>
      <template v-if="props.user?.idrol === 1">
        <label>
          Empresa
          <input type="number" v-model.number="form.idempresa" />
        </label>
        <label>
          Rol
          <select v-model.number="form.idrol">
            <option :value="3">Usuario</option>
            <option :value="2">Supervisor</option>
            <option :value="1">Administrador</option>
          </select>
        </label>
      </template>
      <template v-else>
        <p class="panel-description">Este usuario se creará automáticamente en tu empresa con rol Usuario.</p>
      </template>
      <div class="form-actions">
        <button @click.prevent="saveUser">{{ selectedUser ? 'Actualizar usuario' : 'Crear usuario' }}</button>
        <button v-if="selectedUser" class="secondary" @click.prevent="resetForm">Cancelar</button>
      </div>
    </div>

    <p class="success" v-if="success">{{ success }}</p>
    <p class="error" v-if="error">{{ error }}</p>

    <div class="user-table">
      <h3>Usuarios</h3>
      <table>
        <thead>
          <tr><th>ID</th><th>Nombre</th><th>Email</th><th>Rol</th><th>Empresa</th><th>Última entrada</th><th>Última salida</th><th>Acciones</th></tr>
        </thead>
        <tbody>
          <tr v-for="userItem in users" :key="userItem.idusuario">
            <td>{{ userItem.idusuario }}</td>
            <td>{{ userItem.usuario }}</td>
            <td>{{ userItem.email }}</td>
            <td>{{ userItem.rol_nombre }}</td>
            <td>{{ userItem.nombre_empresa || userItem.idempresa }}</td>
            <td>{{ userItem.ultima_entrada ? new Date(userItem.ultima_entrada).toLocaleString('es-ES') : '-' }}</td>
            <td>{{ userItem.ultima_salida ? new Date(userItem.ultima_salida).toLocaleString('es-ES') : '-' }}</td>
            <td>
              <button class="small" type="button" @click.prevent="openEditAndFocus(userItem)">Editar</button>
              <button class="small secondary" @click.prevent="deleteUser(userItem.idusuario)">Borrar</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>
