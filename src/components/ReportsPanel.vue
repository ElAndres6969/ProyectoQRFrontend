<script setup>
import { ref, onMounted } from 'vue';
const props = defineProps({ token: String });
const report = ref([]);
const error = ref('');

async function loadReport() {
  error.value = '';
  try {
    const res = await fetch('http://localhost:4000/api/reports/hours', {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    report.value = data.report;
  } catch (err) {
    error.value = err.message || 'No se pudo cargar el reporte';
  }
}

onMounted(loadReport);
</script>

<template>
  <section class="panel">
    <h2>Reportes de horas</h2>
    <p class="panel-description">Monitorea entradas, salidas y tiempos de jornada por usuario.</p>

    <button class="small" @click.prevent="loadReport">Actualizar</button>
    <p class="error" v-if="error">{{ error }}</p>

    <div class="report-table" v-if="report.length">
      <table>
        <thead>
          <tr>
            <th>Usuario</th>
            <th>Rol</th>
            <th>Empresa</th>
            <th>Entradas</th>
            <th>Salidas</th>
            <th>Primera entrada</th>
            <th>Última salida</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in report" :key="item.idusuario">
            <td>{{ item.usuario }}</td>
            <td>{{ item.rol }}</td>
            <td>{{ item.nombre_empresa || item.idempresa }}</td>
            <td>{{ item.entradas }}</td>
            <td>{{ item.salidas }}</td>
            <td>{{ item.primera_entrada || '---' }}</td>
            <td>{{ item.ultima_salida || '---' }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</template>
