<script setup>
import { ref, onMounted } from 'vue';
const props = defineProps({ token: String });
const incidents = ref([]);
const form = ref({ tipo: '', descripcion: '' });
const error = ref('');
const message = ref('');

async function loadIncidents() {
  error.value = '';
  try {
    const res = await fetch('https://proyectoqrbackend.onrender.com/api/api/incidents', {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    incidents.value = data.incidents;
  } catch (err) {
    error.value = err.message || 'No se pudieron cargar las incidencias';
  }
}

async function createIncident() {
  error.value = '';
  message.value = '';
  try {
    const res = await fetch('https://proyectoqrbackend.onrender.com/api/api/incidents', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${props.token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(form.value),
    });
    const data = await res.json();
    if (!res.ok) throw data;
    message.value = 'Incidencia registrada';
    form.value.tipo = '';
    form.value.descripcion = '';
    await loadIncidents();
  } catch (err) {
    error.value = err.message || 'No se pudo crear la incidencia';
  }
}

async function resolveIncident(id) {
  error.value = '';
  try {
    const res = await fetch(`https://proyectoqrbackend.onrender.com/api/api/incidents/${id}/resolve`, {
      method: 'PUT',
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    message.value = data.message;
    await loadIncidents();
  } catch (err) {
    error.value = err.message || 'No se pudo resolver la incidencia';
  }
}

onMounted(loadIncidents);
</script>

<template>
  <section class="panel">
    <h2>Incidencias y validaciones</h2>
    <p class="panel-description">Registra eventos y gestiona su resolución para trazar la historia laboral.</p>

    <div class="panel-card">
      <h3>Reportar incidencia</h3>
      <label>
        Tipo
        <input v-model="form.tipo" placeholder="Ej. retraso, ausencia" />
      </label>
      <label>
        Descripción
        <textarea v-model="form.descripcion" rows="3"></textarea>
      </label>
      <button @click.prevent="createIncident">Enviar incidencia</button>
      <p class="success" v-if="message">{{ message }}</p>
      <p class="error" v-if="error">{{ error }}</p>
    </div>

    <div class="report-table">
      <h3>Incidencias recientes</h3>
      <table>
        <thead>
          <tr><th>ID</th><th>Tipo</th><th>Estado</th><th>Creado</th><th>Por</th><th>Acción</th></tr>
        </thead>
        <tbody>
          <tr v-for="item in incidents" :key="item.idincidencia">
            <td>{{ item.idincidencia }}</td>
            <td>{{ item.tipo }}</td>
            <td>{{ item.estado }}</td>
            <td>{{ item.creado_en }}</td>
            <td>{{ item.creador }}</td>
            <td>
              <button v-if="item.estado !== 'resuelta'" @click.prevent="resolveIncident(item.idincidencia)">Resolver</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>
