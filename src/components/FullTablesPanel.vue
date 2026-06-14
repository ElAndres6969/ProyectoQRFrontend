<script setup>
import { ref } from 'vue';
const props = defineProps({ token: String });

const selected = ref('');
const loading = ref(false);
const error = ref('');
const rows = ref([]);
const tableName = ref('');

const options = [
  { value: 'qr', label: 'QR' },
  { value: 'logs', label: 'logs' },
  { value: 'usuario', label: 'Usuario' },
  { value: 'empresa', label: 'Empresa' },
];

async function loadTable() {
  if (!selected.value) return;
  loading.value = true;
  error.value = '';
  rows.value = [];
  tableName.value = '';
  try {
    const res = await fetch(`http://localhost:4000/api/admin/full-table/${selected.value}`, {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw new Error(data.message || 'Error cargando tabla');
    tableName.value = data.table;
    rows.value = data.rows || [];
  } catch (err) {
    error.value = err.message || 'Error';
  } finally {
    loading.value = false;
  }
}
</script>

<template>
  <section class="panel">
    <h2>Ver tablas completas (admin)</h2>
    <p class="panel-description">Selecciona la tabla para ver todos sus registros.</p>

    <div class="panel-card">
      <label>
        Tabla
        <select v-model="selected">
          <option value="">-- Seleccionar --</option>
          <option v-for="opt in options" :key="opt.value" :value="opt.value">{{ opt.label }}</option>
        </select>
      </label>
      <div class="form-actions">
        <button type="button" @click="loadTable" :disabled="!selected || loading">Cargar tabla</button>
      </div>
    </div>

    <div class="panel-card">
      <h3 v-if="tableName">Tabla: {{ tableName }}</h3>
      <p v-if="error" class="error">{{ error }}</p>
      <p v-if="loading">Cargando...</p>

      <div v-if="!loading && rows.length">
        <table class="data-table">
          <thead>
            <tr>
              <th v-for="col in Object.keys(rows[0])" :key="col">{{ col }}</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(r, idx) in rows" :key="idx">
              <td v-for="col in Object.keys(rows[0])" :key="col">{{ r[col] }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <p v-else-if="!loading">No hay registros para mostrar.</p>
    </div>
  </section>
</template>

<style scoped>
.data-table { width: 100%; border-collapse: collapse; }
.data-table th, .data-table td { border: 1px solid #e6e6e6; padding: 6px; text-align: left; }
.panel-description { margin-bottom: 12px; }
</style>
