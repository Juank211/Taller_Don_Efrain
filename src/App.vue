<script setup>
import { ref, computed } from 'vue'
import { useLocalStorage } from '@vueuse/core'
import { useQuasar } from 'quasar'

const $q = useQuasar()

// Persistencia en LocalStorage
const servicios = useLocalStorage('taller_servicios_v3', [
  {
    id: 1725883200000,
    ticket: 'SST-0001',
    fechaIngreso: '2026-09-08 10:30',
    cliente: 'Carlos Rodríguez',
    telefono: '310 456 7890',
    marca: 'Samsung',
    modelo: 'Galaxy S22 Ultra',
    estadoRecepcion: 'Pantalla estrellada en esquina superior, enciende y táctil responde.',
    tipoReparacion: 'Cambio pantalla',
    tecnicoAsignado: 'Efrain',
    precio: 280000,
    metodoPago: 'Transferencia',
    estadoPago: 'Abono',
    abono: 150000,
    estadoServicio: 'En reparacion',
    calificacion: 5,
    observaciones: 'Cliente solicita entrega antes del fin de semana.'
  },
  {
    id: 1725886800000,
    ticket: 'SST-0002',
    fechaIngreso: '2026-09-07 14:15',
    cliente: 'Mariana Gómez',
    telefono: '315 987 6543',
    marca: 'Apple (iPhone)',
    modelo: 'iPhone 13 Pro',
    estadoRecepcion: 'Buen estado estético, no recibe carga con ningún cable.',
    tipoReparacion: 'Cambio pin de carga',
    tecnicoAsignado: 'Juan',
    precio: 140000,
    metodoPago: 'Efectivo',
    estadoPago: 'Pagado',
    abono: 140000,
    estadoServicio: 'Entregado',
    calificacion: 5,
    observaciones: 'Garantía de 3 meses en repuesto.'
  },
  {
    id: 1725889000000,
    ticket: 'SST-0003',
    fechaIngreso: '2026-09-09 08:00',
    cliente: 'Andrés Morales',
    telefono: '320 654 1122',
    marca: 'Xiaomi',
    modelo: 'Redmi Note 12',
    estadoRecepcion: 'Carcasa con rayones leves, batería inflada.',
    tipoReparacion: 'Cambio bateria',
    tecnicoAsignado: 'Fernando',
    precio: 95000,
    metodoPago: 'Efectivo',
    estadoPago: 'Pendiente',
    abono: 0,
    estadoServicio: 'Recibido',
    calificacion: 5,
    observaciones: 'Revisar también si calienta al cargar.'
  }
])

// Estados del modal y filtros
const dialogNuevoServicio = ref(false)
const modoEdicion = ref(false)
const idEdicion = ref(null)
const filtroBusqueda = ref('')
const filtroEstado = ref('Todos')

// Opciones
const opcionesTecnicos = ['Efrain', 'Juan', 'Fernando', 'Alexander']
const opcionesMarcas = [
  'Samsung',
  'Apple (iPhone)',
  'Xiaomi',
  'Motorola',
  'Huawei',
  'Honor',
  'Realme',
  'Oppo',
  'Lenovo',
  'HP',
  'Dell',
  'Asus',
  'Acer',
  'Sony',
  'Otra'
]
const tipoReparacion = [
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

// Formulario reactivo
const formulario = ref({
  ticket: '',
  fechaIngreso: '',
  cliente: '',
  telefono: '',
  marca: 'Samsung',
  modelo: '',
  estadoRecepcion: '',
  tipoReparacion: 'Cambio pantalla',
  tecnicoAsignado: 'Efrain',
  precio: null,
  metodoPago: 'Efectivo',
  estadoPago: 'Pendiente',
  abono: null,
  estadoServicio: 'Recibido',
  calificacion: 5,
  observaciones: ''
})

// Fecha no editable
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
    fechaIngreso: obtenerFechaActual(), // Automática y no editable
    cliente: '',
    telefono: '',
    marca: 'Samsung',
    modelo: '',
    estadoRecepcion: '',
    tipoReparacion: 'Cambio pantalla',
    tecnicoAsignado: opcionesTecnicos[0],
    precio: null,
    metodoPago: 'Efectivo',
    estadoPago: 'Pendiente',
    abono: null,
    estadoServicio: 'Recibido',
    calificacion: 5,
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
  dialogNuevoServicio.value = true
}

function guardarServicio() {
  if (formulario.value.estadoPago === 'Abono') {
    if (!formulario.value.abono || formulario.value.abono <= 0) {
      $q.notify({
        type: 'negative',
        message: 'Abono obligatorio',
        caption: 'Ingresa el monto abonado.',
        position: 'top'
      })
      return
    }
  } else if (formulario.value.estadoPago === 'Pagado') {
    formulario.value.abono = formulario.value.precio
  } else {
    formulario.value.abono = 0
  }

  if (modoEdicion.value) {
    const idx = servicios.value.findIndex(s => s.id === idEdicion.value)
    if (idx !== -1) {
      if (servicios.value[idx].estadoServicio === 'Entregado') {
        $q.notify({
          type: 'negative',
          message: 'No permitido',
          caption: 'No se puede modificar un servicio entregado.',
          position: 'top'
        })
        return
      }
      servicios.value[idx] = { ...formulario.value, id: idEdicion.value }
      $q.notify({
        type: 'positive',
        message: 'Servicio actualizado correctamente',
        position: 'top'
      })
    }
  } else {
    const nuevo = {
      ...formulario.value,
      id: Date.now()
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
  limpiarFormulario()
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
    $q.notify({
      type: 'info',
      message: 'Registro eliminado',
      position: 'top'
    })
  })
}

// Saldo pendiente computado
const saldoPendiente = computed(() => {
  const precio = Number(formulario.value.precio) || 0
  const abono = Number(formulario.value.abono) || 0
  return Math.max(0, precio - abono)
})

// Filtrado de servicios
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
    case 'Recibido':
      return 'border-estado-recibido'
    case 'En reparacion':
      return 'border-estado-reparacion'
    case 'Listo para entrega':
      return 'border-estado-listo'
    case 'Entregado':
      return 'border-estado-entregado'
    default:
      return 'border-estado-recibido'
  }
}
</script>

<template>
  <q-layout view="lHh Lpr lFf" class="bg-dark-app text-white">
    <!-- Header -->
    <q-header class="bg-header text-white q-py-xs" style="border-bottom: 2px solid #ed2100;">
      <q-toolbar>
        <q-avatar size="36px" color="negative" text-color="white" icon="cell_tower" class="q-mr-sm" />
        <q-toolbar-title class="text-weight-bolder text-uppercase">
          Taller Don Efraín <span class="text-caption text-grey-5">| Tech Service</span>
        </q-toolbar-title>

        <q-btn
          unelevated
          rounded
          class="bg-scarlet text-white text-bold q-px-md"
          icon="add"
          label="Nuevo Servicio"
          @click="abrirModalNuevo"
        />
      </q-toolbar>
    </q-header>

    <!-- Contenido Principal -->
    <q-page-container class="q-pa-md">
      <div class="row q-col-gutter-sm q-mb-md items-center">
        <div class="col-12 col-sm-6 col-md-5">
          <q-input
            dark
            dense
            outlined
            v-model="filtroBusqueda"
            placeholder="Buscar por cliente, ticket, marca, modelo..."
            clearable
            class="bg-input-search"
          >
            <template v-slot:prepend>
              <q-icon name="search" color="grey-5" />
            </template>
          </q-input>
        </div>

        <div class="col-12 col-sm-6 col-md-7">
          <div class="row q-gutter-xs justify-sm-end">
            <q-btn
              v-for="estado in ['Todos', 'Recibido', 'En reparacion', 'Listo para entrega', 'Entregado']"
              :key="estado"
              dense
              no-caps
              unelevated
              :class="filtroEstado === estado ? 'bg-scarlet text-white text-bold' : 'bg-grey-9 text-grey-4'"
              :label="estado"
              class="q-px-sm"
              style="border-radius: 6px; font-size: 13px;"
              @click="filtroEstado = estado"
            />
          </div>
        </div>
      </div>

      <!-- Estado vacío -->
      <div
        v-if="serviciosFiltrados.length === 0"
        class="flex flex-center column text-center q-pa-xl text-grey-6"
        style="min-height: 50vh;"
      >
        <q-icon name="smartphone" size="80px" color="grey-8" />
        <p class="text-h6 q-mt-md text-weight-light">No hay servicios registrados</p>
        <p class="text-caption text-grey-5">
          Registra un servicio usando el botón <span class="text-scarlet font-bold">"+ Nuevo Servicio"</span>
        </p>
      </div>

      <div v-else class="row q-col-gutter-md">
        <div
          v-for="servicio in serviciosFiltrados"
          :key="servicio.id"
          class="col-12 col-sm-6 col-md-4"
        >
          <q-card
            flat
            class="tarjeta-equipo-estatica"
            :class="getBorderClass(servicio.estadoServicio)"
          >
            <!-- Cabecera de la tarjeta -->
            <q-card-section class="q-pb-none">
              <div class="row items-center justify-between no-wrap q-mb-xs">
                <span class="text-caption text-grey-5 font-mono">{{ servicio.ticket }}</span>
                <q-chip
                  dense
                  square
                  class="text-bold"
                  :color="
                    servicio.estadoServicio === 'Entregado'
                      ? 'positive'
                      : servicio.estadoServicio === 'Listo para entrega'
                      ? 'warning'
                      : servicio.estadoServicio === 'En reparacion'
                      ? 'orange-9'
                      : 'purple-8'
                  "
                  text-color="white"
                >
                  {{ servicio.estadoServicio }}
                </q-chip>
              </div>

              <div class="text-h6 text-bold text-white ellipsis">{{ servicio.cliente }}</div>
              <div class="text-subtitle2 text-scarlet text-bold">{{ servicio.marca }} - {{ servicio.modelo }}</div>
              <div v-if="servicio.telefono" class="text-caption text-grey-5">{{ servicio.telefono }}</div>
            </q-card-section>

            <!-- Datos del servicio -->
            <q-card-section class="q-py-sm text-grey-4">
              <div class="row q-mb-xs">
                <div class="col-5 text-grey-6">Reparación:</div>
                <div class="col-7 text-bold text-white">{{ servicio.tipoReparacion }}</div>
              </div>

              <div class="row q-mb-xs" v-if="servicio.estadoRecepcion">
                <div class="col-5 text-grey-6">Recepción:</div>
                <div class="col-7 text-grey-3 text-caption">{{ servicio.estadoRecepcion }}</div>
              </div>

              <div class="row q-mb-xs">
                <div class="col-5 text-grey-6">Técnico:</div>
                <div class="col-7">{{ servicio.tecnicoAsignado }}</div>
              </div>

              <div class="row q-mb-xs">
                <div class="col-5 text-grey-6">Fecha:</div>
                <div class="col-7 text-caption">{{ servicio.fechaIngreso }}</div>
              </div>

              <div class="row q-mb-xs items-center">
                <div class="col-5 text-grey-6">Costo:</div>
                <div class="col-7 text-bold text-scarlet text-subtitle1">
                  ${{ Number(servicio.precio || 0).toLocaleString() }}
                  <span class="text-caption text-grey-5 font-normal">({{ servicio.metodoPago }})</span>
                </div>
              </div>

              <div class="row items-center q-mt-xs">
                <div class="col-5 text-grey-6">Pago:</div>
                <div class="col-7">
                  <q-badge
                    outline
                    :color="
                      servicio.estadoPago === 'Pendiente'
                        ? 'negative'
                        : servicio.estadoPago === 'Abono'
                        ? 'warning'
                        : 'positive'
                    "
                    class="text-bold q-px-sm"
                  >
                    {{ servicio.estadoPago }}
                  </q-badge>
                  <span
                    v-if="servicio.estadoPago === 'Abono'"
                    class="text-caption text-warning q-ml-xs text-bold"
                  >
                    (Abonó: ${{ Number(servicio.abono || 0).toLocaleString() }})
                  </span>
                </div>
              </div>

              <div
                v-if="servicio.estadoPago === 'Abono'"
                class="row items-center q-mt-xs text-caption text-negative text-bold"
              >
                <div class="col-5">Saldo resta:</div>
                <div class="col-7">
                  ${{ Math.max(0, (servicio.precio || 0) - (servicio.abono || 0)).toLocaleString() }}
                </div>
              </div>

              <div
                v-if="servicio.observaciones"
                class="q-mt-sm q-pa-xs rounded-borders bg-black text-grey-5 text-caption"
              >
                <em>Obs: {{ servicio.observaciones }}</em>
              </div>

              <!-- Calificación si está Entregado -->
              <div v-if="servicio.estadoServicio === 'Entregado'" class="q-mt-sm q-pa-xs rounded-borders bg-black">
                <div class="row items-center justify-between">
                  <span class="text-caption text-positive text-bold">Calificación Cliente:</span>
                  <q-rating v-model="servicio.calificacion" readonly size="1.1em" color="amber-7" />
                </div>
              </div>

              <div
                v-if="servicio.estadoServicio === 'Entregado'"
                class="q-mt-xs text-caption text-grey-6 text-center"
              >
                <q-icon name="lock" size="12px" class="q-mr-xs text-positive" />
                Registro finalizado (No editable)
              </div>
            </q-card-section>

            <q-separator dark />

            <q-card-actions align="right" class="bg-black q-px-sm q-py-xs">
              <q-btn
                flat
                dense
                icon="edit"
                :color="servicio.estadoServicio === 'Entregado' ? 'grey-7' : 'grey-4'"
                :disable="servicio.estadoServicio === 'Entregado'"
                @click="abrirModalEdicion(servicio)"
              >
                <q-tooltip>
                  {{
                    servicio.estadoServicio === 'Entregado'
                      ? 'Los servicios entregados no pueden ser editados'
                      : 'Editar servicio'
                  }}
                </q-tooltip>
              </q-btn>

              <q-btn
                flat
                dense
                icon="delete"
                :color="servicio.estadoServicio === 'Entregado' ? 'grey-7' : 'negative'"
                :disable="servicio.estadoServicio === 'Entregado'"
                @click="eliminarServicio(servicio)"
              >
                <q-tooltip>
                  {{
                    servicio.estadoServicio === 'Entregado'
                      ? 'Los servicios entregados no pueden ser eliminados'
                      : 'Eliminar servicio'
                  }}
                </q-tooltip>
              </q-btn>
            </q-card-actions>
          </q-card>
        </div>
      </div>
    </q-page-container>

    <!-- MODAL DE FORMULARIO -->
    <q-dialog v-model="dialogNuevoServicio" persistent>
      <q-card
        class="bg-grey-10 text-white"
        style="min-width: 340px; max-width: 650px; width: 100%; border: 1px solid #d20a2e; border-radius: 8px;"
      >
        <!-- Título Modal -->
        <q-card-section class="row items-center bg-black q-py-sm">
          <div class="text-h6 text-bold text-scarlet">
            {{ modoEdicion ? 'Editar Registro de Servicio' : 'Ingresar Equipo al Taller' }}
          </div>
          <q-space />
          <q-btn icon="close" flat round dense v-close-popup text-color="white" />
        </q-card-section>

        <q-separator dark />

        <!-- Formulario -->
        <q-card-section class="q-pt-md" style="max-height: 80vh; overflow-y: auto;">
          <q-form @submit.prevent="guardarServicio" class="q-gutter-y-sm">
            
            <!-- Fila 1: Fecha Automática (NO editable) y Ticket -->
            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-6">
                <q-input
                  dark
                  color="negative"
                  v-model="formulario.fechaIngreso"
                  label="Fecha de ingreso (Automática) *"
                  outlined
                  dense
                  readonly
                  disable
                  hint="Generada automáticamente"
                >
                  <template v-slot:prepend>
                    <q-icon name="lock" size="18px" color="grey-5" />
                  </template>
                </q-input>
              </div>

              <div class="col-12 col-sm-6">
                <q-input
                  dark
                  color="negative"
                  v-model="formulario.ticket"
                  label="N° Ticket *"
                  outlined
                  dense
                  readonly
                  disable
                />
              </div>
            </div>

            <!-- Fila 2: Cliente y Teléfono -->
            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-7">
                <q-input
                  dark
                  color="negative"
                  v-model="formulario.cliente"
                  label="Nombre del cliente *"
                  outlined
                  dense
                  :rules="[val => !!val || 'Obligatorio']"
                />
              </div>
              <div class="col-12 col-sm-5">
                <q-input
                  dark
                  color="negative"
                  v-model="formulario.telefono"
                  label="Teléfono / Celular *"
                  outlined
                  dense
                  :rules="[val => !!val || 'Obligatorio']"
                />
              </div>
            </div>

            <!-- Fila 3: Marca y Modelo  -->
            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-6">
                <q-select
                  dark
                  color="negative"
                  v-model="formulario.marca"
                  :options="opcionesMarcas"
                  label="Marca del equipo *"
                  outlined
                  dense
                  :rules="[val => !!val || 'Seleccione la marca']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-input
                  dark
                  color="negative"
                  v-model="formulario.modelo"
                  label="Modelo del equipo *"
                  placeholder="Ej: Galaxy S22, Redmi Note 12..."
                  outlined
                  dense
                  :rules="[val => !!val || 'Campo independiente obligatorio']"
                />
              </div>
            </div>

            <!-- Fila 4: Estado del dispositivo -->
            <div>
              <q-input
                dark
                color="negative"
                v-model="formulario.estadoRecepcion"
                type="textarea"
                rows="2"
                label="Estado en el que se recibe el dispositivo *"
                placeholder="Ej: Pantalla estrellada pero da imagen, sin bandeja SIM, no enciende..."
                outlined
                dense
                :rules="[val => !!val || 'Indique el estado de recepción del equipo']"
              />
            </div>

            <!-- Fila 5: Tipo de Reparación y Técnico Asignado -->
            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-6">
                <q-select
                  dark
                  color="negative"
                  v-model="formulario.tipoReparacion"
                  :options="tipoReparacion"
                  label="Tipo de reparación *"
                  outlined
                  dense
                  :rules="[val => !!val || 'Obligatorio']"
                />
              </div>
              <div class="col-12 col-sm-6">
                <q-select
                  dark
                  color="negative"
                  v-model="formulario.tecnicoAsignado"
                  :options="opcionesTecnicos"
                  label="Técnico asignado *"
                  outlined
                  dense
                  :rules="[val => !!val || 'Obligatorio']"
                />
              </div>
            </div>

            <!-- Fila 6: Precio, Método y Estado de Pago -->
            <div class="row q-col-gutter-sm">
              <div class="col-12 col-sm-4">
                <q-input
                  dark
                  color="negative"
                  v-model.number="formulario.precio"
                  type="number"
                  label="Precio cobrado ($) *"
                  outlined
                  dense
                  :rules="[val => val !== null && val !== '' || 'Obligatorio', val => val >= 0 || 'Inválido']"
                />
              </div>
              <div class="col-12 col-sm-4">
                <q-select
                  dark
                  color="negative"
                  v-model="formulario.metodoPago"
                  :options="metodosPago"
                  label="Método de pago"
                  outlined
                  dense
                />
              </div>
              <div class="col-12 col-sm-4">
                <q-select
                  dark
                  color="negative"
                  v-model="formulario.estadoPago"
                  :options="estadosPago"
                  label="Estado del pago *"
                  outlined
                  dense
                />
              </div>
            </div>

            <!-- Fila Dinámica: Abono si estadoPago es 'Abono' -->
            <div v-if="formulario.estadoPago === 'Abono'" class="q-pa-sm bg-black rounded-borders">
              <div class="row q-col-gutter-sm items-center">
                <div class="col-12 col-sm-6">
                  <q-input
                    dark
                    color="warning"
                    v-model.number="formulario.abono"
                    type="number"
                    label="Valor del Abono ($) *"
                    outlined
                    dense
                    :rules="[val => val !== null && val !== '' && val > 0 || 'Ingrese el valor del abono']"
                  />
                </div>
                <div class="col-12 col-sm-6 text-center">
                  <div class="text-caption text-grey-4">Saldo pendiente a cancelar:</div>
                  <div class="text-subtitle1 text-warning text-bold">
                    ${{ saldoPendiente.toLocaleString() }}
                  </div>
                </div>
              </div>
            </div>

            <!-- Fila 7: Estado del Servicio -->
            <div>
              <q-select
                dark
                color="negative"
                v-model="formulario.estadoServicio"
                :options="estadosServicio"
                label="Estado del servicio / equipo *"
                outlined
                dense
              />
            </div>

            <!-- Fila Dinámica: Calificación  -->
            <div
              v-if="formulario.estadoServicio === 'Entregado'"
              class="q-pa-sm bg-black rounded-borders q-mt-xs"
            >
              <div class="row items-center justify-between">
                <label class="text-caption text-grey-4 text-bold">Calificación del cliente (1 a 5):</label>
                <q-rating v-model="formulario.calificacion" size="1.8em" color="amber-7" />
              </div>
            </div>

            <!-- Observaciones -->
            <div>
              <q-input
                dark
                color="negative"
                v-model="formulario.observaciones"
                type="textarea"
                rows="2"
                label="Observaciones (opcional)"
                outlined
                dense
              />
            </div>

            <!-- Botones -->
            <div class="row justify-end q-mt-md">
              <q-btn label="Cancelar" flat color="grey-5" v-close-popup class="q-mr-sm" />
              <q-btn
                class="bg-scarlet text-white text-bold"
                :label="modoEdicion ? 'Actualizar' : 'Guardar'"
                type="submit"
              />
            </div>

          </q-form>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<style scoped>
.bg-dark-app {
  background-color: #121212;
  min-height: 100vh;
}

.bg-header {
  background-color: #0a0a0a;
}

.bg-input-search {
  background-color: #1a1a1a;
  border-radius: 6px;
}

.bg-scarlet {
  background-color: #ed2100 !important;
}
.text-scarlet {
  color: #ed2100 !important;
}

/* ========================================================
   TARJETAS ESTÁTICAS (Sin animaciones ni transiciones)
   ======================================================== */
.tarjeta-equipo-estatica {
  background-color: #1c1c1c !important;
  color: #ffffff !important;
  border-radius: 8px !important;
  border: 1px solid #2e2e2e;
}

/* Bordes izquierdos según el ESTADO DEL SERVICIO */
.border-estado-recibido {
  border-left: 6px solid #9333ea !important; /* Morado: Recibido */
}

.border-estado-reparacion {
  border-left: 6px solid #f97316 !important; /* Naranja: En reparación */
}

.border-estado-listo {
  border-left: 6px solid #eab308 !important; /* Amarillo: Listo para entrega */
}

.border-estado-entregado {
  border-left: 6px solid #16a34a !important; /* Verde: Entregado */
}

.font-mono {
  font-family: monospace;
}
.font-normal {
  font-weight: normal;
}
.font-bold {
  font-weight: bold;
}
</style>
