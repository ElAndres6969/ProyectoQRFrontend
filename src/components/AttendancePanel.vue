<script setup>
import { computed, ref, onMounted, onUnmounted } from 'vue';
import QRCode from 'qrcode';
import { QrcodeStream } from 'vue-qrcode-reader';
const props = defineProps({ token: String, user: Object });
const tokenQr = ref(null);
const qrDataUrl = ref('');
const scanToken = ref('');
const scanType = ref('entrada');
const scanActive = ref(false);
const scannerError = ref('');
const message = ref('');
const error = ref('');
const canScan = computed(() => [1, 2].includes(Number(props.user?.idrol)));
const refreshSeconds = ref(60);
let refreshTimer = null;
let countdownTimer = null;
const newUserName = ref('');
const newUserEmail = ref('');
const newUserPassword = ref('');
const newUserMessage = ref('');
const newUserError = ref('');

async function createQrImage(text) {
  try {
    qrDataUrl.value = await QRCode.toDataURL(text, { width: 256, margin: 2 });
  } catch (err) {
    console.error('Error generando imagen QR:', err);
    qrDataUrl.value = '';
  }
}

async function loadToken() {
  error.value = '';
  try {
    const res = await fetch('https://proyectoqrbackend.onrender.com/api/attendance/qrcode/me', {
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    tokenQr.value = data.qr;
    if (data.qr?.tokenqr) {
      await createQrImage(data.qr.tokenqr);
    }
  } catch (err) {
    error.value = err.message || 'No se pudo cargar el token QR';
  }
}

function startRefreshCountdown() {
  refreshSeconds.value = 60;
  clearInterval(countdownTimer);
  countdownTimer = setInterval(() => {
    refreshSeconds.value -= 1;
    if (refreshSeconds.value <= 0) {
      refreshSeconds.value = 60;
    }
  }, 1000);
}

function startRefreshTimer() {
  clearInterval(refreshTimer);
  refreshTimer = setInterval(() => {
    generateToken();
  }, 60000);
  startRefreshCountdown();
}

async function generateToken() {
  error.value = '';
  message.value = '';
  try {
    const res = await fetch('https://proyectoqrbackend.onrender.com/api/attendance/qrcode/generate', {
      method: 'POST',
      headers: { Authorization: `Bearer ${props.token}` },
    });
    const data = await res.json();
    if (!res.ok) throw data;
    tokenQr.value = { tokenqr: data.token, creado_en: new Date().toISOString() };
    await createQrImage(data.token);
    message.value = 'Código QR generado correctamente.';
    refreshSeconds.value = 60;
    startRefreshCountdown();
  } catch (err) {
    error.value = err.message || 'No se pudo generar el QR';
  }
}

async function scan() {
  if (!canScan.value) {
    error.value = 'No tiene permiso para usar el escáner QR.';
    return;
  }
  error.value = '';
  message.value = '';
  if (!scanToken.value) {
    error.value = 'Debe haber un token QR válido para registrar.';
    return;
  }
  try {
    const res = await fetch('https://proyectoqrbackend.onrender.com/api/attendance/scan', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${props.token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({ tokenqr: scanToken.value, registra: scanType.value }),
    });
    const data = await res.json();
    if (!res.ok) throw data;
    message.value = data.message;
  } catch (err) {
    error.value = err.message || 'No se pudo registrar el fichaje';
  }
}

function handleDecode(decodedString) {
  scanToken.value = decodedString;
  scannerError.value = '';
  scan();
}

function onScannerInit(promise) {
  promise.catch(err => {
    scannerError.value = 'No se pudo iniciar la cámara. Comprueba permisos.';
    console.error(err);
  });
}

function onScannerError(err) {
  scannerError.value = err.message || 'Error en el escáner QR';
}

async function createUser() {
  newUserError.value = '';
  newUserMessage.value = '';

  if (!newUserName.value || !newUserEmail.value || !newUserPassword.value) {
    newUserError.value = 'Todos los campos son obligatorios';
    return;
  }

  if (!props.user?.idempresa) {
    newUserError.value = 'No se pudo detectar la empresa asociada';
    return;
  }

  try {
    const res = await fetch('https://proyectoqrbackend.onrender.com/api/users', {
      method: 'POST',
      headers: {
        Authorization: `Bearer ${props.token}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        usuario: newUserName.value,
        email: newUserEmail.value,
        password: newUserPassword.value,
        idempresa: props.user.idempresa,
        idrol: 3,
      }),
    });
    const data = await res.json();
    if (!res.ok) throw data;
    newUserMessage.value = 'Usuario creado correctamente';
    newUserName.value = '';
    newUserEmail.value = '';
    newUserPassword.value = '';
  } catch (err) {
    newUserError.value = err.message || 'Error al crear el usuario';
  }
}

onMounted(async () => {
  await generateToken();
  startRefreshTimer();
});

onUnmounted(() => {
  clearInterval(refreshTimer);
  clearInterval(countdownTimer);
});
</script>

<template>
  <section class="panel">
    <h2>Registro de jornada</h2>
    <p class="panel-description">Genera códigos QR dinámicos y registra entrada/salida.</p>

    <div class="panel-card">
      <h3>Token QR del usuario</h3>
      <button @click.prevent="generateToken">Generar código QR</button>
      <div v-if="tokenQr" class="qr-preview">
        <img v-if="qrDataUrl" :src="qrDataUrl" alt="Código QR generado" />
        <pre class="qr-token">{{ tokenQr.tokenqr }}</pre>
        <p>Creado: {{ tokenQr.creado_en }}</p>
        <p class="hint">El token se renovará cada {{ refreshSeconds }} segundos.</p>
      </div>
    </div>

    <div class="panel-card">
      <h3>Escanear código QR</h3>
      <p v-if="!canScan" class="panel-description error">
        No tiene permiso para usar el escáner QR. Este modo está reservado a supervisores y administradores.
      </p>
      <template v-if="canScan">
        <label>
          Tipo
          <select v-model="scanType">
            <option value="entrada">Entrada</option>
            <option value="salida">Salida</option>
          </select>
        </label>
        <button type="button" @click.prevent="scanActive = !scanActive">
          {{ scanActive ? 'Detener escáner' : 'Abrir escáner QR' }}
        </button>
        <div v-if="scanActive" class="qr-scanner">
          <QrcodeStream
            @decode="handleDecode"
            @init="onScannerInit"
            @error="onScannerError"
          />
          <p v-if="scannerError" class="error">{{ scannerError }}</p>
          <p class="hint">Permita el acceso a la cámara en el navegador cuando se lo solicite.</p>
        </div>
        <label>
          Token QR detectado / manual
          <input v-model="scanToken" placeholder="Token QR" />
        </label>
        <button type="button" @click.prevent="scan">Registrar entrada/salida</button>
      </template>
      <p class="success" v-if="message">{{ message }}</p>
      <p class="error" v-if="error">{{ error }}</p>
    </div>
  </section>
</template>
