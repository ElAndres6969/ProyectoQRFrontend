<script setup>
import { ref, onMounted } from 'vue';
const props = defineProps({ token: String });
const logs = ref([]);
const error = ref('');
const loading = ref(false);
const searchUser = ref('');
let searchTimeoutId = null;

async function loadLogs() {
  loading.value = true;
  error.value = '';
  try {
    const params = new URLSearchParams();
    if (searchUser.value.trim()) {
      params.append('user', searchUser.value.trim());
    }
    const url = `https://proyectoqrbackend.onrender.com/api/api/audit/logs${params.toString() ? `?${params.toString()}` : ''}`;
    const res = await fetch(url, {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    logs.value = data.audit;
  } catch (err) {
    error.value = err.message || 'No se pudieron cargar los logs';
  } finally {
    loading.value = false;
  }
}

function scheduleLoadLogs() {
  clearTimeout(searchTimeoutId);
  searchTimeoutId = setTimeout(loadLogs, 250);
}

onMounted(loadLogs);
</script>

<template>
  <section class="panel">
    <h2>Lista de logs</h2>
    <p class="panel-description">Consulta los registros de auditoría de la empresa en tiempo real.</p>

    <div class="panel-card">
      <label>
        Buscar por nombre de usuario
        <input
          v-model="searchUser"
          placeholder="Escribe nombre de usuario"
          @input="scheduleLoadLogs"
        />
      </label>
    </div>

    <div class="report-table">
      <p v-if="loading">Cargando...</p>
      <p v-if="error" class="error">{{ error }}</p>
      <table v-if="logs.length && !loading">
        <thead>
          <tr>
            <th>ID</th>
            <th>Usuario</th>
            <th>Acción</th>
            <th>Fecha</th>
            <th>Empresa</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="log in logs" :key="log.idlog">
            <td>{{ log.idlog }}</td>
            <td>{{ log.usuario }}</td>
            <td>{{ log.registra }}</td>
            <td>{{ log.fecha }}</td>
            <td>{{ log.nombre_empresa || log.idempresa }}</td>
          </tr>
        </tbody>
      </table>
      <p v-if="!logs.length && !loading">No hay registros disponibles.</p>
    </div>
  </section>
</template>
