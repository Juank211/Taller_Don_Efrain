<script setup>
import { ref } from 'vue'
import { useLocalStorage } from '@vueuse/core';

const servicios = useLocalStorage('taller_servicios_v1', [])

const dialogNuevoServicio = ref(false)
const modoEdicion = ref(false)
const idEdicion = ref(null)

const opcionTecnicos = ['Efrain', 'Juan', 'Fernando']
const tipoReparacion = [
    'Cambio pantalla',
    'Cambio bateria',
    'Cambio pin de carga',
    'liberacion',
    'Mantenimiento de Software',
    'Cambio de flex',
    'Otros'
]

const metodosPago = ['Efectivo', 'Transferencia', 'Tarjeta']
const estadoPago = ['Pendiente', 'Pagado', 'Abono']
const estadoServicio = ['Recibido', 'En reparacion', 'Listo para entrega', 'Entregado']

const formulario = ref({
    cliente: '',
    equipo: '',
    tipoReparacion: '',
    tecnicoAsignado: '',
    fechaIngreso: '',
    precio: null,
    metodoPago: 'Efectivo',
    estadoPago: 'Pendiente',
    estadoServicio: 'Recibido',
    calificacion: 5,
    observaciones: ''
})

function limpiarFormulario() {
    const ahora = new Date()
    const fechaLocal = new Date(ahora.getTime() - (ahora.getTimezoneOffset() * 60000))
    const fechaFormateada = fechaLocal.toISOString().slice(0, 16)
    formulario.value = {
        cliente: '',
        equipo: '',
        tipoReparacion: '',
        tecnicoAsignado: '',
        fechaIngreso: fechaFormateada,
        precio: null,
        metodoPago: 'Efectivo',
        estadoPago: 'Pendiente',
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
    modoEdicion.value = true
    idEdicion.value = servicio.id
    formulario.value = { ...servicio }
    dialogNuevoServicio.value = true
}

function guardarServicio() {
    if (modoEdicion.value) {
        for (let i = 0; i < servicios.value.length; i++) {
            if (servicios.value[i].id === idEdicion.value) {
                servicios.value[i] = { ...formulario.value, id: idEdicion.value }
                break
            }
        }
    } else {
        const nuevoServicio = {
            ...formulario.value,
            id: Date.now()
        }
        servicios.value.push(nuevoServicio)
    }
    dialogNuevoServicio.value = false
    limpiarFormulario()
}

function eliminarServicio(id) {
    if (confirm('Seguro que deseas eliminar este servicio?')) {
        const nuevaLista = []
        for (let i = 0; i < servicios.value.length; i++) {
            if (servicios.value[i].id !== id) {
                nuevaLista.push(servicios.value[i])
            }
        }
        servicios.value = nuevaLista
    }
}
</script>

<template>
    <q-layout view="lHh Lpr lFf" class="bg-dark text-white">
        <q-header class="bg-black text-white q-py-xs" style="border-bottom: 2px solid #ed2100">
            <q-toolbar>
                <q-avatar size="36px" color="negative" text-color="white" icon="cell_tower" class="q-mr-sm" />
                <q-toolbar-title class="text-weight-bolder text-uppercase tracking-wide">
                    Taller Don Efraín <span class="text-caption text-grey-5">| Tech Service</span>
                </q-toolbar-title>

                <q-btn unelevated rounded class="bg-scarlett text-white text-bold q-px-md" icon="add"
                    label="Nuevo Servicio" @click="abrirModalNuevo" />
            </q-toolbar>
        </q-header>

        <q-page-container class="q-pa-md">
            <div v-if="servicios.length === 0" class="flex flex-center column text-center q-pa-xl text-grey-6"
                style="min-height: 60vh;">
                <q-icon name="smartphone" size="80px" color="grey-8" />
                <p class="text-h6 q-mt-md text-weight-light">No hay servicios registrados</p>
                <p class="text-caption">Registra tu primer servicio para comenzar usando el boton <span
                        class="text-scarlett">"+ Nuevo Servicio"</span> en la parte superior derecha de la pantalla
                </p>
            </div>

            <div v-else class="row q-col-gutter-md">
                <div v-for="servicio in servicios" :key="servicio.id" class="col-12 col-sm-6 col-md-4">
                    <q-card flat class="tarjeta-equipo" :class="{
                        'border-pendiente': servicio.estadoPago === 'Pendiente',
                        'border-abono': servicio.estadoPago === 'Abono',
                        'border-pagado': servicio.estadoPago === 'Pagado'
                    }">
                        <q-card-section class="row items-center q-pb-none">
                            <div class="col">
                                <div class="text-h6 text-bold text-white">{{ item.cliente }}</div>
                                <div class="text-subtitle2 text-grey-5">{{ item.equipo }}</div>
                            </div>
                            <q-chip dense square class="text-bold"
                                :color="item.estadoServicio === 'Entregado' ? 'positive' : item.estadoServicio === 'Listo para entrega' ? 'warning' : 'negative'"
                                text-color="white">
                                {{ item.estadoServicio }}
                            </q-chip>
                        </q-card-section>

                        <q-card-section class="q-py-md text-grey-4">
                            <div class="row q-mb-xs">
                                <div class="col-5 text-grey-6">Reparación:</div>
                                <div class="col-7 text-bold text-white">{{ item.tipoReparacion }}</div>
                            </div>
                            <div class="row q-mb-xs">
                                <div class="col-5 text-grey-6">Técnico:</div>
                                <div class="col-7">{{ item.tecnicoAsignado }}</div>
                            </div>
                            <div class="row q-mb-xs">
                                <div class="col-5 text-grey-6">Ingreso:</div>
                                <div class="col-7">{{ item.fechaIngreso }}</div>
                            </div>
                            <div class="row q-mb-xs items-center">
                                <div class="col-5 text-grey-6">Costo:</div>
                                <div class="col-7 text-bold text-scarlet text-subtitle1">
                                    ${{ item.precio }}
                                    <span class="text-caption text-grey-5">({{ item.metodoPago }})</span>
                                </div>
                            </div>

                            <div class="row items-center q-mt-sm">
                                <div class="col-5 text-grey-6">Pago:</div>
                                <div class="col-7">
                                    <q-badge outline
                                        :color="item.estadoPago === 'Pendiente' ? 'negative' : item.estadoPago === 'Abono' ? 'warning' : 'positive'"
                                        class="text-bold q-px-sm">
                                        {{ item.estadoPago }}
                                    </q-badge>
                                </div>
                            </div>

                            <div v-if="item.observaciones"
                                class="q-mt-sm q-pa-xs rounded-borders bg-black text-grey-5 text-caption">
                                <em>Obs: {{ item.observaciones }}</em>
                            </div>

                            <div v-if="item.estadoServicio === 'Entregado'" class="q-mt-sm">
                                <q-rating v-model="item.calificacion" readonly size="1em" color="scarlet" />
                            </div>
                        </q-card-section>
                        <q-separator dark />

                        <q-card-actions align="right" class="bg-black">
                            <q-btn flat dense icon="edit" color="grey-4" @click="abrirModalEdicion(item)" />
                            <q-btn flat dense icon="delete" color="negative" @click="eliminarServicio(item.id)" />
                        </q-card-actions>
                    </q-card>
                </div>
            </div>
        </q-page-container>

        <q-dialog v-model="dialogNuevoServicio" persistent>
            <q-card class="bg-grey-10 text-white"
                style="min-width: 350px; max-width: 600px; width: 100%; border: 1px solid #d20a2e;">
                <q-card-section class="row items-center bg-black">
                    <div class="text-h6 text-bold text-scarlet">{{ modoEdicion ? 'Editar Registro' : 'Ingresar Equipo'
                        }}</div>
                    <q-space />
                    <q-btn icon="close" flat round dense v-close-popup text-color="white" />
                </q-card-section>

                <q-card-section class="q-pt-md">
                    <q-form @submit.prevent="guardarServicio" class="q-gutter-md">

                        <q-input dark color="negative" v-model="formulario.cliente" label="Nombre del cliente *"
                            outlined dense :rules="[val => !!val || 'Obligatorio']" />
                        <q-input dark color="negative" v-model="formulario.equipo" label="Marca y Modelo *" outlined
                            dense :rules="[val => !!val || 'Obligatorio']" />

                        <q-select dark color="negative" v-model="formulario.tipoReparacion" :options="tipoReparacion"
                            label="Tipo de reparación *" outlined dense :rules="[val => !!val || 'Obligatorio']" />
                        <q-select dark color="negative" v-model="formulario.tecnicoAsignado" :options="opcionTecnicos"
                            label="Técnico asignado *" outlined dense :rules="[val => !!val || 'Obligatorio']" />

                        <q-input dark color="negative" v-model="formulario.fechaIngreso" type="datetime-local"
                            label="Fecha y hora de recepción *" outlined dense stack-label
                            :rules="[val => !!val || 'Obligatorio']" />
                        <q-input dark color="negative" v-model.number="formulario.precio" type="number"
                            label="Precio cobrado *" outlined dense
                            :rules="[val => val !== null && val !== '' || 'Obligatorio', val => val >= 0 || 'Inválido']" />

                        <q-select dark color="negative" v-model="formulario.metodoPago" :options="metodosPago"
                            label="Método de pago" outlined dense />
                        <q-select dark color="negative" v-model="formulario.estadoPago" :options="estadoPago"
                            label="Estado del pago" outlined dense />
                        <q-select dark color="negative" v-model="formulario.estadoServicio" :options="estadoServicio"
                            label="Estado del equipo" outlined dense />

                        <div v-if="formulario.estadoServicio === 'Entregado'">
                            <label class="text-caption text-grey-4">Calificación del cliente:</label>
                            <q-rating v-model="formulario.calificacion" size="2em" color="scarlet" />
                        </div>

                        <q-input dark color="negative" v-model="formulario.observaciones" type="textarea"
                            label="Observaciones (opcional)" outlined dense />

                        <div class="row justify-end q-mt-md">
                            <q-btn label="Cancelar" flat color="grey-5" v-close-popup class="q-mr-sm" />
                            <q-btn class="bg-scarlet text-white" :label="modoEdicion ? 'Actualizar' : 'Guardar'"
                                type="submit" />
                        </div>

                    </q-form>
                </q-card-section>
            </q-card>
        </q-dialog>
    </q-layout>
</template>

<style scoped>
.bg-scarlet {
  background-color: #ed2100 !important;
}
.text-scarlet {
  color: #ed2100 !important;
}

.border-pendiente {
  border-left: 6px solid #ed2100 !important;
}
.border-abono {
  border-left: 6px solid #d20a2e !important;
}
.border-pagado {
  border-left: 6px solid #2e7d32 !important;
}

.tarjeta-equipo {
  background-color: #1e1e1e !important;
  color: #ffffff !important;
  border-radius: 12px !important;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.tarjeta-equipo:hover {
  transform: translateY(-4px);
  box-shadow: 0 8px 20px rgba(237, 33, 0, 0.15) !important;
}
</style>
