<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Formato moneda COP
function formatoMoneda(valor) {
  const num = Number(valor) || 0
  return new Intl.NumberFormat('es-CO', {
    style: 'currency',
    currency: 'COP',
    maximumFractionDigits: 0
  }).format(num)
}

// Persistencia en LocalStorage
const servicios = useLocalStorage('taller_servicios_v5', [])

const dialogNuevoServicio = ref(false)
const modoEdicion = ref(false)
const idEdicion = ref(null)
const filtroBusqueda = ref('')
const filtroEstado = ref('Todos')

const opcionesTecnicos = ['Efrain', 'Juan', 'Fernando', 'Alexander']
const opcionesMarcas = [
  'Samsung', 'Apple (iPhone)', 'Xiaomi', 'Motorola', 'Huawei',
  'Honor', 'Realme', 'Oppo', 'Lenovo', 'HP', 'Dell', 'Asus', 'Acer', 'Sony', 'Otra'
]
const opcionesServicios = [
  'Cambio pantalla',
  'Cambio bateria',
  'Cambio pin de carga',
  'Reparación de placa / microsoldadura',
  'Mantenimiento de Software / Flasheo',
  'Cambio de flex / periféricos',
  'Limpieza por humedad',
  'Diagnóstico general',
  'Otros'
]
const metodosPago = ['Efectivo', 'Transferencia', 'Tarjeta']
const estadosPago = ['Pendiente', 'Abono', 'Pagado']
const estadosServicio = ['Recibido', 'En reparacion', 'Listo para entrega', 'Entregado']

// Formulario reactivo (sin campo calificacion; se guarda pero NO se pide en el modal)
const formulario = ref({
  ticket: '',
  fechaIngreso: '',
  cliente: '',
  telefono: '',
  marca: 'Samsung',
  modelo: '',
  tipoReparacion: ['Cambio pantalla'],
  tecnicoAsignado: 'Efrain',
  precio: null,
  metodoPago: 'Efectivo',
  estadoPago: 'Pendiente',
  abono: null,
  estadoServicio: 'Recibido',
  calificacion: null,     // Null hasta que se guarda con estado Entregado
  observaciones: ''
})

function obtenerFechaActual() {
  const ahora = new Date()
  const year = ahora.getFullYear()
  const month = String(ahora.getMonth() + 1).padStart(2, '0')
  const day = String(ahora.getDate()).padStart(2, '0')
  const hours = String(ahora.getHours()).padStart(2, '0')
  const minutes = String(ahora.getMinutes()).padStart(2, '0')
  return `${year}-${month}-${day} ${hours}:${minutes}`
}

function generarTicket() {
  const num = servicios.value.length + 1
  return `SST-${String(num).padStart(4, '0')}`
}

function limpiarFormulario() {
  formulario.value = {
    ticket: generarTicket(),
    fechaIngreso: obtenerFechaActual(),
    cliente: '',
    telefono: '',
    marca: 'Samsung',
    modelo: '',
    tipoReparacion: ['Cambio pantalla'],
    tecnicoAsignado: opcionesTecnicos[0],
    precio: null,
    metodoPago: 'Efectivo',
    estadoPago: 'Pendiente',
    abono: null,
    estadoServicio: 'Recibido',
    calificacion: null,
    observaciones: ''
  }
}

function abrirModalNuevo() {
  limpiarFormulario()
  modoEdicion.value = false
  idEdicion.value = null
  dialogNuevoServicio.value = true
}

function abrirModalEdicion(servicio) {
  if (servicio.estadoServicio === 'Entregado') {
    $q.notify({
      type: 'warning',
      message: 'Bloqueado',
      caption: 'Un registro en estado "Entregado" no puede ser editado.',
      position: 'top'
    })
    return
  }
  modoEdicion.value = true
  idEdicion.value = servicio.id
  formulario.value = JSON.parse(JSON.stringify(servicio))
  if (!Array.isArray(formulario.value.tipoReparacion)) {
    formulario.value.tipoReparacion = [formulario.value.tipoReparacion]
  }
  dialogNuevoServicio.value = true
}

// Ref para la calificación al momento de guardar un servicio entregado
const calificacionEntrega = ref(5)
const dialogCalificacion = ref(false)
const servicioPendienteGuardar = ref(null)

function guardarServicio() {
  // Validación estricta sin campos vacíos
  if (
    !formulario.value.cliente?.trim() ||
    !formulario.value.telefono?.trim() ||
    !formulario.value.marca ||
    !formulario.value.modelo?.trim() ||
    !formulario.value.tipoReparacion ||
    formulario.value.tipoReparacion.length === 0 ||
    !formulario.value.tecnicoAsignado ||
    formulario.value.precio === null ||
    formulario.value.precio === '' ||
    Number(formulario.value.precio) <= 0 ||
    !formulario.value.metodoPago ||
    !formulario.value.estadoPago ||
    !formulario.value.estadoServicio ||
    !formulario.value.observaciones?.trim()
  ) {
    $q.notify({
      type: 'negative',
      message: 'Campos requeridos',
      caption: 'Por favor completa todos los campos obligatorios antes de guardar.',
      position: 'top'
    })
    return
  }

  if (formulario.value.estadoPago === 'Abono') {
    if (!formulario.value.abono || Number(formulario.value.abono) <= 0) {
      $q.notify({
        type: 'negative',
        message: 'Abono obligatorio',
        caption: 'Ingresa un valor de abono mayor a 0.',
        position: 'top'
      })
      return
    }
    if (Number(formulario.value.abono) > Number(formulario.value.precio)) {
      $q.notify({
        type: 'negative',
        message: 'Valor inválido',
        caption: 'El abono no puede superar el precio total.',
        position: 'top'
      })
      return
    }
  } else if (formulario.value.estadoPago === 'Pagado') {
    formulario.value.abono = formulario.value.precio
  } else {
    formulario.value.abono = 0
  }

  const saldoResta = Number(formulario.value.precio) - Number(formulario.value.abono)
  if (formulario.value.estadoServicio === 'Entregado' && saldoResta !== 0 ) {
    $q.notify({
      type: 'negative',
      message: 'Saldo pendiente',
      caption: 'No se puede entregar un servicio con saldo pendiente.',
      position: 'top',
      timeout: 4000
    })
    return
  }

  // Si el estado es Entregado → mostrar diálogo de calificación DESPUÉS de guardar
  const esEntregado = formulario.value.estadoServicio === 'Entregado'

  if (modoEdicion.value) {
    const idx = servicios.value.findIndex(s => s.id === idEdicion.value)
    if (idx !== -1) {
      if (servicios.value[idx].estadoServicio === 'Entregado') {
        $q.notify({
          type: 'negative',
          message: 'No permitido',
          caption: 'No se puede modificar un servicio ya entregado.',
          position: 'top'
        })
        return
      }
      // Preservar la calificación previa si ya existía y no es Entregado ahora
      const calPrev = servicios.value[idx].calificacion
      servicios.value[idx] = {
        ...formulario.value,
        id: idEdicion.value,
        calificacion: esEntregado ? null : calPrev  // null = pendiente calificar
      }
      $q.notify({
        type: 'positive',
        message: 'Servicio actualizado correctamente',
        position: 'top'
      })
    }
  } else {
    const nuevo = {
      ...formulario.value,
      id: Date.now(),
      calificacion: esEntregado ? null : null
    }
    servicios.value.unshift(nuevo)
    $q.notify({
      type: 'positive',
      message: 'Servicio registrado con éxito',
      caption: `Ticket: ${nuevo.ticket}`,
      position: 'top'
    })
  }

  dialogNuevoServicio.value = false

  // Si fue marcado como Entregado, abrir calificación
  if (esEntregado) {
    // Encontrar el registro recién guardado para calificar
    calificacionEntrega.value = 5
    serviciosPorCalificar()
  }

  limpiarFormulario()
}

function serviciosPorCalificar() {
  // Buscar el primer servicio Entregado sin calificación aún
  const pendiente = servicios.value.find(
    s => s.estadoServicio === 'Entregado' && (s.calificacion === null || s.calificacion === undefined)
  )
  if (pendiente) {
    serviciosPendienteId.value = pendiente.id
    calificacionEntrega.value = 5
    dialogCalificacion.value = true
  }
}

const serviciosPendienteId = ref(null)

function guardarCalificacion() {
  const idx = servicios.value.findIndex(s => s.id === serviciosPendienteId.value)
  if (idx !== -1) {
    servicios.value[idx] = {
      ...servicios.value[idx],
      calificacion: calificacionEntrega.value
    }
  }
  dialogCalificacion.value = false
  $q.notify({
    type: 'positive',
    message: '¡Gracias por tu calificación!',
    icon: 'star',
    position: 'top'
  })
}

function omitirCalificacion() {
  dialogCalificacion.value = false
}

function eliminarServicio(servicio) {
  if (servicio.estadoServicio === 'Entregado') {
    $q.notify({
      type: 'warning',
      message: 'No permitido',
      caption: 'Un registro en estado "Entregado" no puede ser eliminado.',
      position: 'top'
    })
    return
  }

  $q.dialog({
    title: 'Confirmar eliminación',
    message: `¿Deseas eliminar el servicio de "${servicio.cliente}" (${servicio.ticket})?`,
    cancel: { label: 'Cancelar', flat: true, color: 'grey-4' },
    ok: { label: 'Eliminar', color: 'negative', unelevated: true },
    dark: true
  }).onOk(() => {
    servicios.value = servicios.value.filter(s => s.id !== servicio.id)
    $q.notify({ type: 'info', message: 'Registro eliminado', position: 'top' })
  })
}

const saldoPendiente = computed(() => {
  const precio = Number(formulario.value.precio) || 0
  const abono = Number(formulario.value.abono) || 0
  return Math.max(0, precio - abono)
})

const serviciosFiltrados = computed(() => {
  return servicios.value.filter(s => {
    const matchEstado = filtroEstado.value === 'Todos' || s.estadoServicio === filtroEstado.value
    const term = filtroBusqueda.value.toLowerCase().trim()
    const matchBusqueda =
      !term ||
      s.cliente?.toLowerCase().includes(term) ||
      s.ticket?.toLowerCase().includes(term) ||
      s.marca?.toLowerCase().includes(term) ||
      s.modelo?.toLowerCase().includes(term) ||
      s.telefono?.toLowerCase().includes(term)
    return matchEstado && matchBusqueda
  })
})

function getBorderClass(estado) {
  switch (estado) {
    case 'Recibido':       return 'border-estado-recibido'
    case 'En reparacion':  return 'border-estado-reparacion'
    case 'Listo para entrega': return 'border-estado-listo'
    case 'Entregado':      return 'border-estado-entregado'
    default:               return 'border-estado-recibido'
  }
}

// Chips de servicios con paleta de marca (rojo/oscuro)
function getChipStyle(idx) {
  const paleta = [
    { bg: '#ed2100', text: '#fff' },
    { bg: '#b81700', text: '#fff' },
    { bg: '#7a0e00', text: '#fff' },
    { bg: '#c23a00', text: '#fff' },
    { bg: '#ff4422', text: '#fff' },
    { bg: '#8b1400', text: '#fff' },
    { bg: '#d12b0e', text: '#fff' },
    { bg: '#a01500', text: '#fff' },
    { bg: '#ff6600', text: '#fff' },
  ]
  const c = paleta[idx % paleta.length]
  return `background-color: ${c.bg}; color: ${c.text}; border: none;`
}
</script>

<template>
  <q-layout view="lHh Lpr lFf" class="bg-dark-app text-white">

    <!-- ══════════════════════════════════════════════════════════
         HEADER — más amplio y con más padding
    ═══════════════════════════════════════════════════════════ -->
    <q-header class="bg-header" style="border-bottom: 3px solid #ed2100;">
      <q-toolbar class="q-py-md" style="min-height: 72px;">
        <q-avatar size="48px" color="negative" text-color="white" icon="cell_tower" class="q-mr-lg" />
        <q-toolbar-title>
          <div class="title-app text-weight-bolder text-uppercase text-white">
            Taller Don Efraín
          </div>
          <div class="text-grey-5" style="font-size: 0.85rem; letter-spacing: 1px; margin-top: 2px;">
            Sistema de Gestión | Tech Service
          </div>
        </q-toolbar-title>

        <q-btn
          unelevated
          rounded
          class="bg-scarlet text-white text-bold btn-nuevo-serv"
          icon="add_circle"
          label="Nuevo Servicio"
          @click="abrirModalNuevo"
          size="md"
        />
      </q-toolbar>
    </q-header>

    <!-- ══════════════════════════════════════════════════════════
         CONTENIDO PRINCIPAL — con espacio superior generoso
    ═══════════════════════════════════════════════════════════ -->
    <q-page-container style="padding-top: 24px;">
      <div class="page-wrapper">

        <!-- Buscador + Filtros con más espacio superior -->
        <div class="row q-col-gutter-lg items-center" style="margin-bottom: 36px; margin-top: 60px;">
          <div class="col-12 col-xl-4 col-lg-4">
            <q-input
              dark
              outlined
              v-model="filtroBusqueda"
              placeholder="Buscar por cliente, ticket, marca, modelo..."
              clearable
              class="bg-input-search search-field"
            >
              <template v-slot:prepend>
                <q-icon name="search" color="grey-4" size="26px" />
              </template>
            </q-input>
          </div>

          <div class="col-12 col-lg-8">
            <div class="filter-buttons-grid">
              <q-btn
                v-for="estado in ['Todos', 'Recibido', 'En reparacion', 'Listo para entrega', 'Entregado']"
                :key="estado"
                no-caps
                unelevated
                :class="filtroEstado === estado
                  ? 'bg-scarlet text-white text-bold shadow-2'
                  : 'bg-grey-9 text-grey-3'"
                :label="estado"
                class="btn-data-filtro"
                @click="filtroEstado = estado"
              />
            </div>
          </div>
        </div>

        <!-- Estado vacío -->
        <div
          v-if="serviciosFiltrados.length === 0"
          class="flex flex-center column text-center q-pa-xl text-grey-5"
          style="min-height: 50vh;"
        >
          <q-icon name="smartphone" size="96px" color="grey-8" />
          <p class="q-mt-md text-weight-medium text-grey-3" style="font-size: 1.4rem;">
            No hay servicios registrados
          </p>
          <p class="text-grey-5" style="font-size: 1rem; max-width: 420px;">
            Registra uno usando <span class="text-scarlet text-weight-bold">"+ Nuevo Servicio"</span>
          </p>
        </div>

        <!-- Grilla de tarjetas — igual altura con flex column -->
        <div v-else class="tarjetas-grid">
          <div
            v-for="servicio in serviciosFiltrados"
            :key="servicio.id"
            class="tarjeta-wrapper"
          >
            <q-card
              flat
              class="tarjeta-equipo column"
              :class="getBorderClass(servicio.estadoServicio)"
            >
              <!-- Cabecera -->
              <q-card-section class="q-pb-xs q-pt-md">
                <div class="row items-center justify-between no-wrap q-mb-sm">
                  <span class="ticket-label font-mono">{{ servicio.ticket }}</span>
                  <q-chip
                    dense
                    square
                    class="text-weight-bold chip-estado"
                    :color="
                      servicio.estadoServicio === 'Entregado'        ? 'positive'
                      : servicio.estadoServicio === 'Listo para entrega' ? 'warning'
                      : servicio.estadoServicio === 'En reparacion'  ? 'orange-9'
                      : 'purple-8'
                    "
                    text-color="white"
                  >
                    {{ servicio.estadoServicio }}
                  </q-chip>
                </div>

                <div class="cliente-nombre ellipsis">{{ servicio.cliente }}</div>
                <div class="equipo-nombre q-mt-xs">{{ servicio.marca }} — {{ servicio.modelo }}</div>
                <div v-if="servicio.telefono" class="telefono-text q-mt-xs row items-center">
                  <q-icon name="phone" size="16px" class="q-mr-xs text-grey-5" />
                  {{ servicio.telefono }}
                </div>
              </q-card-section>

              <!-- Cuerpo — crece para igualar alturas -->
              <q-card-section class="q-py-sm col-grow">
                <!-- Chips de servicios con paleta de marca -->
                <div class="q-mb-sm">
                  <div class="section-label q-mb-xs">Servicio(s) a realizar:</div>
                  <div class="row q-gutter-xs">
                    <span
                      v-for="(serv, idx) in (Array.isArray(servicio.tipoReparacion)
                        ? servicio.tipoReparacion
                        : [servicio.tipoReparacion])"
                      :key="idx"
                      class="servicio-chip"
                      :style="getChipStyle(idx)"
                    >
                      {{ serv }}
                    </span>
                  </div>
                </div>

                <!-- Datos rápidos -->
                <div class="info-line">
                  <span class="info-label">Técnico:</span>
                  <span class="info-val text-white text-weight-medium">{{ servicio.tecnicoAsignado }}</span>
                </div>
                <div class="info-line">
                  <span class="info-label">Fecha:</span>
                  <span class="info-val text-grey-4">{{ servicio.fechaIngreso }}</span>
                </div>

                <!-- Costo (moneda) -->
                <div class="info-line q-mt-xs">
                  <span class="info-label">Costo total:</span>
                  <span class="costo-val text-scarlet text-weight-bolder">
                    {{ formatoMoneda(servicio.precio) }}
                  </span>
                </div>
                <div class="info-line">
                  <span class="info-label">Método pago:</span>
                  <span class="info-val text-grey-3">{{ servicio.metodoPago }}</span>
                </div>

                <!-- Estado de pago -->
                <div class="info-line items-center q-mt-xs">
                  <span class="info-label">Estado pago:</span>
                  <q-badge
                    outline
                    :color="
                      servicio.estadoPago === 'Pendiente' ? 'negative'
                      : servicio.estadoPago === 'Abono' ? 'warning'
                      : 'positive'
                    "
                    class="text-weight-bold pago-badge"
                  >
                    {{ servicio.estadoPago }}
                  </q-badge>
                </div>

                <!-- Abono y saldo restante -->
                <div v-if="servicio.estadoPago === 'Abono'" class="abono-box q-pa-sm q-mt-xs rounded-borders">
                  <div class="row justify-between abono-text text-warning text-weight-bold">
                    <span>Abonado:</span>
                    <span>{{ formatoMoneda(servicio.abono) }}</span>
                  </div>
                  <div class="row justify-between abono-text text-negative text-weight-bold q-mt-xs">
                    <span>Saldo resta:</span>
                    <span>{{ formatoMoneda(Math.max(0, (servicio.precio || 0) - (servicio.abono || 0))) }}</span>
                  </div>
                </div>

                <!-- Observaciones / Recepción unificadas -->
                <div v-if="servicio.observaciones" class="obs-box q-mt-sm q-pa-sm rounded-borders">
                  <span class="obs-label">Recepción / Obs:</span>
                  {{ servicio.observaciones }}
                </div>

                <!-- Calificación (solo si está guardada, después de Entregado) -->
                <div
                  v-if="servicio.estadoServicio === 'Entregado' && servicio.calificacion !== null && servicio.calificacion !== undefined"
                  class="rating-box q-mt-sm q-pa-sm rounded-borders"
                >
                  <div class="row items-center justify-between">
                    <span class="section-label text-positive">Calificación del cliente:</span>
                    <q-rating v-model="servicio.calificacion" readonly size="1.3em" color="amber-7" />
                  </div>
                </div>

                <!-- Pendiente de calificar -->
                <div
                  v-if="servicio.estadoServicio === 'Entregado' && (servicio.calificacion === null || servicio.calificacion === undefined)"
                  class="q-mt-sm text-caption text-grey-5 text-italic"
                >
                  <q-icon name="hourglass_empty" size="14px" class="q-mr-xs" />
                  Calificación pendiente del cliente
                </div>
              </q-card-section>

              <q-separator dark />

              <!-- Acciones -->
              <q-card-actions class="bg-black q-px-md q-py-sm tarjeta-footer">
                <!-- Entregado: solo badge, sin botones de edición/eliminación -->
                <div
                  v-if="servicio.estadoServicio === 'Entregado'"
                  class="row items-center text-positive text-weight-bold footer-text"
                >
                  <q-icon name="check_circle" size="20px" class="q-mr-xs" />
                  Servicio Entregado
                </div>

                <!-- No entregado: botones de editar y eliminar -->
                <div v-else class="row items-center q-gutter-x-sm full-width justify-end">
                  <q-btn flat round dense icon="edit" color="grey-3" @click="abrirModalEdicion(servicio)">
                    <q-tooltip>Editar servicio</q-tooltip>
                  </q-btn>
                  <q-btn flat round dense icon="delete" color="negative" @click="eliminarServicio(servicio)">
                    <q-tooltip>Eliminar servicio</q-tooltip>
                  </q-btn>
                </div>
              </q-card-actions>
            </q-card>
          </div>
        </div>

      </div>
    </q-page-container>

    <!-- ══════════════════════════════════════════════════════════
         MODAL FORMULARIO
    ═══════════════════════════════════════════════════════════ -->
    <q-dialog v-model="dialogNuevoServicio" persistent>
      <q-card class="bg-grey-10 text-white modal-form-box">
        <!-- Encabezado Modal -->
        <q-card-section class="row items-center bg-black q-py-md">
          <div class="modal-title text-scarlet text-bold">
            {{ modoEdicion ? 'Editar Registro de Servicio' : 'Ingresar Equipo al Taller' }}
          </div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup text-color="white" size="md" />
        </q-card-section>

        <q-separator dark />

        <q-card-section class="q-pa-lg form-scroll">
          <q-form @submit.prevent="guardarServicio" class="q-gutter-y-lg">

            <!-- Fecha y Ticket (no editables) -->
            <div class="row q-col-gutter-md">
              <div class="col-12 col-sm-6">
                <q-input
                  dark color="negative"
                  v-model="formulario.fechaIngreso"
                  label="Fecha de ingreso (Automática)"
                  outlined readonly disable
                  hint="Generada automáticamente al crear el registro"
                >
                  <template v-slot:prepend>
                    <q-icon name="lock" size="22px" color="grey-5" />
                  </template>
                </q-input>
              </div>
              <div class="col-12 col-sm-6">
                <q-input
                  dark color="negative"
                  v-model="formulario.ticket"
                  label="N° Ticket"
                  outlined readonly disable class="font-mono"
                />
              </div>
            </div>

            <!-- Cliente y Teléfono -->
            <div class="row q-col-gutter-md">
              <div class="col-12 col-sm-7">
                <q-input
                  dark color="negative"
                  v-model="formulario.cliente"
                  label="Nombre del cliente *"
                  outlined
                  :rules="[v => !!v?.trim() || 'El nombre del cliente es obligatorio']"
                />
              </div>
              <div class="col-12 col-sm-5">
                <q-input
                  dark color="negative"
                  v-model="formulario.telefono"
                  label="Teléfono / Celular *"
                  outlined
                  :rules="[v => !!v?.trim() || 'El teléfono es obligatorio']"
                />
              </div>
            </div>

            <!-- Marca y Modelo -->
            <div class="row q-col-gutter-md">
              <div class="col-12 col-sm-6">
                <q-select
                  dark color="negative"
                  v-model="formulario.marca"
                  :options="opcionesMarcas"
                  label="Marca del equipo *"
                  outlined
                  :rules="[v => !!v || 'Seleccione la marca']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-input
                  dark color="negative"
                  v-model="formulario.modelo"
                  label="Modelo del equipo *"
                  placeholder="Ej: Galaxy S22, Redmi Note 12..."
                  outlined
                  :rules="[v => !!v?.trim() || 'El modelo es obligatorio']"
                />
              </div>
            </div>

            <!-- Servicios múltiples -->
            <div>
              <q-select
                dark color="negative"
                v-model="formulario.tipoReparacion"
                :options="opcionesServicios"
                label="Servicio(s) / Reparaciones a realizar *"
                multiple use-chips outlined
                :rules="[v => (v && v.length > 0) || 'Debes seleccionar al menos un servicio']"
              />
            </div>

            <!-- Técnico y Estado del Servicio -->
            <div class="row q-col-gutter-md">
              <div class="col-12 col-sm-6">
                <q-select
                  dark color="negative"
                  v-model="formulario.tecnicoAsignado"
                  :options="opcionesTecnicos"
                  label="Técnico asignado *"
                  outlined
                  :rules="[v => !!v || 'Seleccione el técnico asignado']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-select
                  dark color="negative"
                  v-model="formulario.estadoServicio"
                  :options="estadosServicio"
                  label="Estado del servicio / equipo *"
                  outlined
                  :rules="[v => !!v || 'Seleccione el estado del servicio']"
                />
              </div>
            </div>

            <!-- Precio, Método y Estado de Pago -->
            <div class="row q-col-gutter-md">
              <div class="col-12 col-sm-4">
                <q-input
                  dark color="negative"
                  v-model.number="formulario.precio"
                  type="number"
                  label="Precio total ($) *"
                  outlined
                  :rules="[
                    v => v !== null && v !== '' || 'Precio obligatorio',
                    v => Number(v) > 0 || 'Debe ser mayor a 0'
                  ]"
                />
              </div>
              <div class="col-12 col-sm-4">
                <q-select
                  dark color="negative"
                  v-model="formulario.metodoPago"
                  :options="metodosPago"
                  label="Método de pago *"
                  outlined
                  :rules="[v => !!v || 'Seleccione el método de pago']"
                />
              </div>
              <div class="col-12 col-sm-4">
                <q-select
                  dark color="negative"
                  v-model="formulario.estadoPago"
                  :options="estadosPago"
                  label="Estado del pago *"
                  outlined
                  :rules="[v => !!v || 'Seleccione el estado del pago']"
                />
              </div>
            </div>

            <!-- Abono dinámico -->
            <div
              v-if="formulario.estadoPago === 'Abono'"
              class="q-pa-md rounded-borders"
              style="background:#111; border: 1px solid #f59e0b;"
            >
              <div class="row q-col-gutter-md items-center">
                <div class="col-12 col-sm-6">
                  <q-input
                    dark color="warning"
                    v-model.number="formulario.abono"
                    type="number"
                    label="Valor del Abono ($) *"
                    outlined
                    :rules="[
                      v => v !== null && v !== '' || 'Ingrese el abono',
                      v => Number(v) > 0 || 'El abono debe ser mayor a 0',
                      v => Number(v) <= Number(formulario.precio) || 'El abono no puede superar el precio'
                    ]"
                  />
                </div>
                <div class="col-12 col-sm-6 text-center">
                  <div class="text-grey-4" style="font-size: 0.9rem;">Saldo pendiente a cancelar:</div>
                  <div class="text-warning text-weight-bolder" style="font-size: 1.5rem;">
                    {{ formatoMoneda(saldoPendiente) }}
                  </div>
                </div>
              </div>
            </div>

            <!-- Observaciones + Estado de recepción unificados -->
            <div>
              <q-input
                dark color="negative"
                v-model="formulario.observaciones"
                type="textarea" rows="3"
                label="Estado de recepción y observaciones del equipo *"
                placeholder="Detalla el estado físico/funcional al ingresar: pantalla rota, enciende, rayones, sin bandeja SIM, notas adicionales..."
                outlined
                :rules="[v => !!v?.trim() || 'Este campo es obligatorio']"
              />
            </div>

            <!-- Botones -->
            <div class="row justify-end q-gutter-sm q-mt-md">
              <q-btn
                label="Cancelar"
                flat color="grey-4"
                v-close-popup
                class="form-btn"
              />
              <q-btn
                class="bg-scarlet text-white text-bold form-btn q-px-xl"
                :label="modoEdicion ? 'Actualizar' : 'Guardar Servicio'"
                type="submit"
              />
            </div>

          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>

    <!-- ══════════════════════════════════════════════════════════
         MODAL CALIFICACIÓN — Se muestra DESPUÉS de guardar
    ═══════════════════════════════════════════════════════════ -->
    <q-dialog v-model="dialogCalificacion" persistent>
      <q-card class="bg-grey-10 text-white" style="min-width: 380px; max-width: 500px; border: 1px solid #16a34a; border-radius: 12px;">
        <q-card-section class="text-center q-pa-xl">
          <q-icon name="check_circle" color="positive" size="56px" />
          <div class="text-weight-bolder text-white q-mt-md" style="font-size: 1.4rem;">
            ¡Servicio Entregado!
          </div>
          <div class="text-grey-4 q-mt-sm" style="font-size: 1rem;">
            Por favor califica la satisfacción del cliente con el servicio prestado:
          </div>

          <div class="q-mt-xl q-mb-md">
            <div class="row items-center justify-between text-grey-5 q-mb-sm" style="font-size: 0.9rem;">
              <span>1 – Peor</span>
              <span>5 – Mejor</span>
            </div>
            <div class="flex flex-center">
              <q-rating
                v-model="calificacionEntrega"
                size="3.5em"
                color="amber-7"
                icon="star"
              />
            </div>
            <div class="text-amber-5 text-weight-bold q-mt-sm" style="font-size: 1.1rem;">
              {{
                calificacionEntrega === 5 ? '⭐ Excelente Servicio (5/5)'
                : calificacionEntrega === 4 ? '⭐ Muy Bueno (4/5)'
                : calificacionEntrega === 3 ? '⭐ Regular (3/5)'
                : calificacionEntrega === 2 ? '⭐ Malo (2/5)'
                : '⭐ Deficiente (1/5)'
              }}
            </div>
          </div>
        </q-card-section>

        <q-card-actions class="justify-center q-pa-md q-gutter-sm">
          <q-btn
            flat color="grey-5"
            label="Omitir"
            @click="omitirCalificacion"
            class="form-btn"
          />
          <q-btn
            class="bg-scarlet text-white text-bold form-btn q-px-xl"
            label="Guardar Calificación"
            @click="guardarCalificacion"
          />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-layout>
</template>

<style scoped>
/* ══════════════════════════════════════════════════════════
   LAYOUT BASE
═══════════════════════════════════════════════════════════ */
.bg-dark-app {
  background-color: #121212;
  min-height: 100vh;
}

.bg-header {
  background-color: #0a0a0a;
}

.page-wrapper {
  margin: 0 auto;
  padding: 0 28px 48px;
}

/* ══════════════════════════════════════════════════════════
   HEADER
═══════════════════════════════════════════════════════════ */
.title-app {
  font-size: 1.35rem;
  letter-spacing: 1px;
  line-height: 1.2;
}

.btn-nuevo-serv {
  font-size: 1rem;
  padding: 10px 24px;
  background-color: #ed2100;
}

/* ══════════════════════════════════════════════════════════
   BUSCADOR Y FILTROS
═══════════════════════════════════════════════════════════ */
.bg-input-search {
  background-color: #1c1c1c;
  border-radius: 8px;
}

.search-field {
  font-size: 1rem;
}

.filter-buttons-grid {
  display: flex;
  gap: 10px;
  width: 100%;
}

.btn-data-filtro {
  flex: 1 1 0;
  height: 44px;
  font-size: 0.92rem;
  font-weight: 600;
  border-radius: 8px;
  transition: none !important;
  letter-spacing: 0.3px;
}

/* ══════════════════════════════════════════════════════════
   TARJETAS — mismo tamaño con CSS Grid
═══════════════════════════════════════════════════════════ */
.tarjetas-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(450px, 1fr));
  gap: 28px;
  align-items: stretch;
}

.tarjeta-wrapper {
  display: flex;
}

.tarjeta-equipo {
  background-color: #1c1c1c !important;
  border-radius: 10px !important;
  border: 1px solid #2e2e2e;
  width: 100%;
}

/* Borde izquierdo según estado de servicio */
.border-estado-recibido   { border-left: 7px solid #9333ea !important; }
.border-estado-reparacion { border-left: 7px solid #f97316 !important; }
.border-estado-listo      { border-left: 7px solid #eab308 !important; }
.border-estado-entregado  { border-left: 7px solid #16a34a !important; }

/* Tipografía de la tarjeta */
.ticket-label {
  font-size: 0.95rem;
  color: #8b949e;
  font-weight: 700;
}

.chip-estado {
  font-size: 0.82rem;
  padding: 3px 10px;
}

.cliente-nombre {
  font-size: 1.25rem;
  font-weight: 800;
  color: #ffffff;
  line-height: 1.25;
}

.equipo-nombre {
  font-size: 1rem;
  font-weight: 700;
  color: #ed2100;
}

.telefono-text {
  font-size: 0.9rem;
  color: #9ca3af;
}

.section-label {
  font-size: 0.85rem;
  font-weight: 700;
  color: #6b7280;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

/* Chips de servicios con paleta de marca */
.servicio-chip {
  display: inline-block;
  font-size: 0.82rem;
  font-weight: 600;
  padding: 3px 9px;
  border-radius: 4px;
  margin-bottom: 4px;
  line-height: 1.4;
}

.info-line {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
  font-size: 0.95rem;
}

.info-label {
  color: #6b7280;
  flex: 0 0 45%;
}

.info-val {
  flex: 0 0 55%;
  text-align: right;
}

.costo-val {
  font-size: 1.15rem;
  text-align: right;
}

.pago-badge {
  font-size: 0.85rem;
  padding: 3px 10px;
}

.abono-box {
  background-color: rgba(245, 158, 11, 0.1);
  border: 1px solid rgba(245, 158, 11, 0.3);
  border-radius: 6px;
}

.abono-text {
  font-size: 0.9rem;
}

.obs-box {
  background-color: #0d0d0d;
  border-left: 3px solid #ed2100;
  color: #9ca3af;
  font-size: 0.88rem;
  line-height: 1.5;
  border-radius: 4px;
}

.obs-label {
  font-weight: 700;
  color: #6b7280;
  display: block;
  margin-bottom: 2px;
  font-size: 0.8rem;
  text-transform: uppercase;
}

.rating-box {
  background-color: rgba(22, 163, 74, 0.1);
  border: 1px solid rgba(22, 163, 74, 0.3);
  border-radius: 6px;
}

.tarjeta-footer {
  border-top: 1px solid #2a2a2a;
  border-radius: 0 0 10px 10px;
  min-height: 50px;
}

.footer-text {
  font-size: 0.95rem;
}

/* ══════════════════════════════════════════════════════════
   MODAL FORMULARIO
═══════════════════════════════════════════════════════════ */
.modal-form-box {
  min-width: 360px;
  max-width: 720px;
  width: 100%;
  border: 1px solid #d20a2e;
  border-radius: 10px;
}

.modal-title {
  font-size: 1.15rem;
}

.form-scroll {
  max-height: 82vh;
  overflow-y: auto;
}

.form-btn {
  font-size: 0.95rem;
  min-height: 42px;
}

/* ══════════════════════════════════════════════════════════
   UTILIDADES
═══════════════════════════════════════════════════════════ */
.bg-scarlet { background-color: #ed2100 !important; }
.text-scarlet { color: #ed2100 !important; }
.font-mono { font-family: monospace; }
</style>
