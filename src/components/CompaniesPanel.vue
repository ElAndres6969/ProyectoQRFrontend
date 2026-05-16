<script setup>
import { ref, watch, onMounted } from 'vue';
const props = defineProps({ token: String });
const companies = ref([]);
const search = ref('');
const page = ref(1);
const pages = ref(1);
const total = ref(0);
const error = ref('');
const success = ref('');
const loading = ref(false);
let timeoutId;

async function loadCompanies() {
  loading.value = true;
  error.value = '';
  try {
    const response = await fetch(`https://proyectoqrbackend.onrender.com/api/api/companies?search=${encodeURIComponent(search.value)}&page=${page.value}`, {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await response.json();
    if (!response.ok) throw data;
    companies.value = data.companies;
    total.value = data.total;
    pages.value = data.pages;
  } catch (err) {
    error.value = err.message || 'No se pudieron cargar las empresas';
  } finally {
    loading.value = false;
  }
}

async function deleteCompany(idempresa) {
  if (!confirm('¿Eliminar esta empresa y todos sus usuarios? Esta acción no se puede deshacer.')) {
    return;
  }

  error.value = '';
  success.value = '';
  loading.value = true;

  try {
    const response = await fetch(`https://proyectoqrbackend.onrender.com/api/api/companies/${idempresa}`, {
      method: 'DELETE',
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await response.json();
    if (!response.ok) throw data;
    success.value = data.message || 'Empresa y usuarios eliminados correctamente';
    await loadCompanies();
  } catch (err) {
    error.value = err.message || 'No se pudo eliminar la empresa';
  } finally {
    loading.value = false;
  }
}

function scheduleSearch() {
  page.value = 1;
  clearTimeout(timeoutId);
  timeoutId = setTimeout(loadCompanies, 250);
}

watch(search, scheduleSearch);
watch(page, loadCompanies);

onMounted(loadCompanies);
</script>

<template>
  <section class="panel">
    <h2>Empresas lista</h2>
    <p class="panel-description">Busca empresas en tiempo real y navega por los resultados paginados.</p>

    <div class="panel-card">
      <label>
        Buscar empresa
        <input v-model="search" placeholder="Escribe el nombre…" />
      </label>
    </div>

    <div class="report-table">
      <h3>Empresas encontradas</h3>
      <p v-if="loading">Cargando...</p>
      <table v-if="companies.length && !loading">
        <thead>
          <tr>
            <th>ID</th>
            <th>Nombre</th>
            <th>Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="company in companies" :key="company.idempresa">
            <td>{{ company.idempresa }}</td>
            <td>{{ company.nombre_empresa }}</td>
            <td>
              <button class="small secondary" type="button" @click.prevent="deleteCompany(company.idempresa)">
                Borrar empresa
              </button>
            </td>
          </tr>
        </tbody>
      </table>
      <p v-if="!companies.length && !loading">No hay empresas que coincidan.</p>
      <p class="success" v-if="success">{{ success }}</p>
      <p class="error" v-if="error">{{ error }}</p>
    </div>

    <div class="pagination" v-if="pages > 1">
      <button class="small" :disabled="page === 1" @click.prevent="page--">Anterior</button>
      <span>Página {{ page }} de {{ pages }}</span>
      <button class="small" :disabled="page === pages" @click.prevent="page++">Siguiente</button>
    </div>
  </section>
</template>
