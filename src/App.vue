<script setup>
import Presupuesto from './components/Presupuesto.vue'
import ControlPresupuesto from './components/ControlPresupuesto.vue'
import iconoNuevoGasto from './assets/img/nuevo-gasto.svg'
import Modal from './components/Modal.vue'
import Gasto from './components/Gasto.vue'
import Filtros from './components/Filtros.vue'
import { ref, reactive, watch, computed, onMounted } from 'vue'
import { generarId } from './helpers'

const modal = reactive({
	mostrar: false,
	animar: false,
})
const gasto = reactive({
	nombre: '',
	cantidad: '',
	categoria: '',
	id: null,
	fecha: Date.now(),
})
const presupuesto = ref(0)
const disponible = ref(0)
const gastado = ref(0)
const gastos = ref([])
const filtro = ref('')

watch(
	gastos,
	() => {
		localStorage.setItem('gastos', JSON.stringify(gastos.value))

		const totalGastado = gastos.value.reduce(
			(total, gasto) => gasto.cantidad + total,
			0
		)
		disponible.value = presupuesto.value - totalGastado
		gastado.value = totalGastado
	},
	{ deep: true }
)

watch(
	modal,
	() => {
		if (!modal.mostrar) {
			reiniciarGasto()
		}
	},
	{ deep: true }
)

watch(presupuesto, () => {
	localStorage.setItem('presupuesto', presupuesto.value)
})

onMounted(() => {
	const presupuestoStorage = localStorage.getItem('presupuesto')
	const gastosStorage = localStorage.getItem('gastos')

	if (gastosStorage) {
		gastos.value = JSON.parse(gastosStorage)
	}
	if (presupuestoStorage) {
		presupuesto.value = Number(presupuestoStorage)
		disponible.value = Number(presupuestoStorage)
	}
})

const definirPresupuesto = (cantidad) => {
	presupuesto.value = cantidad
	disponible.value = cantidad
}

const mostrarModal = () => {
	modal.mostrar = true
	setTimeout(() => {
		modal.animar = true
	}, 300)
}

const ocultarModal = () => {
	setTimeout(() => {
		modal.mostrar = false
	}, 300)
	modal.animar = false
}

const guardarGasto = () => {
	if (gasto.id) {
		const id = gasto.id
		const index = gastos.value.findIndex((gasto) => gasto.id === id)
		gastos.value[index] = { ...gasto }
	} else {
		gastos.value.push({ ...gasto, id: generarId() })
	}

	ocultarModal()
	reiniciarGasto()
}

const reiniciarGasto = () => {
	Object.assign(gasto, {
		nombre: '',
		cantidad: '',
		categoria: '',
		id: null,
		fecha: Date.now(),
	})
}

const seleccionarGasto = (id) => {
	const gastoEditar = gastos.value.filter((gasto) => gasto.id === id)[0]
	Object.assign(gasto, gastoEditar)
	mostrarModal()
}

const eliminarGasto = () => {
	if (confirm('¿Eliminar?')) {
		gastos.value = gastos.value.filter(
			(gastoState) => gastoState.id !== gasto.id
		)
		ocultarModal()
	}
}

const gastosFiltrados = computed(() => {
	if (filtro.value) {
		return gastos.value.filter((gasto) => gasto.categoria === filtro.value)
	}
	return gastos.value
})

const resetApp = () => {
	if (confirm('¿Deseas reinicar el presupuesto y los gastos?')) {
		presupuesto.value = 0
		gastos.value = []
	}
}
</script>

<template>
	<div :class="{ fijar: modal.mostrar }">
		<header>
			<h1>Planificador de Gastos</h1>
			<div class="contenedor-header contenedor sombra">
				<Presupuesto
					v-if="presupuesto <= 0"
					@definir-presupuesto="definirPresupuesto"
				/>
				<ControlPresupuesto
					v-else
					:presupuesto="presupuesto"
					:disponible="disponible"
					:gastado="gastado"
					@reset-app="resetApp"
				/>
			</div>
		</header>

		<main v-if="presupuesto > 0">
			<Filtros v-model:filtro="filtro" />
			<div class="contenedor listado-gastos">
				<h2>{{ gastos.length > 0 ? 'Gastos' : 'No hay gastos' }}</h2>

				<Gasto
					v-for="gasto in gastosFiltrados"
					:key="gasto.id"
					:gasto="gasto"
					@seleccionar-gasto="seleccionarGasto"
				/>
			</div>
			<div class="crear-gasto">
				<img
					@click="mostrarModal"
					:src="iconoNuevoGasto"
					alt="Icono Nuevo Gasto"
				/>
			</div>
			<Modal
				v-model:nombre="gasto.nombre"
				v-model:cantidad="gasto.cantidad"
				v-model:categoria="gasto.categoria"
				v-if="modal.mostrar"
				@ocultar-modal="ocultarModal"
				@guardar-gasto="guardarGasto"
				@eliminar-gasto="eliminarGasto"
				:modal="modal"
				:disponible="disponible"
				:id="gasto.id"
			/>
		</main>
	</div>
</template>

<style>
:root {
	--azul: #3b86f6;
	--blanco: #fff;
	--gris-claro: #f5f5f5;
	--gris: #94a3b8;
	--gris-oscuro: #64748b;
	--negro: #000;
}

html {
	font-size: 62.5%;
	box-sizing: border-box;
}

*,
*:before,
*:after {
	box-sizing: inherit;
}

body {
	font-size: 1.6rem;
	font-family: 'Lato', sans-serif;
	background-color: var(--gris-claro);
}

h1 {
	font-size: 4rem;
}

h2 {
	font-size: 3rem;
}

header {
	background-color: var(--azul);
}

header h1 {
	padding: 3rem 0;
	margin: 0;
	color: var(--blanco);
	text-align: center;
}

.contenedor {
	width: 90%;
	max-width: 80rem;
	margin: 0 auto;
}

.contenedor-header {
	margin-top: -5rem;
	transform: translateY(5rem);
	padding: 5rem;
}

.sombra {
	box-shadow: 0px 10px 15px -3px rgba(0, 0, 0, 0.1);
	background-color: var(--blanco);
	border-radius: 1.2rem;
	padding: 5rem;
}

.crear-gasto {
	position: fixed;
	bottom: 5rem;
	right: 5rem;
}

.crear-gasto img {
	width: 5rem;
	cursor: pointer;
}

.listado-gastos {
	margin-top: 10rem;
}

.listado-gastos h2 {
	font-weight: 900;
	color: var(--gris-oscuro);
}

.fijar {
	overflow: hidden;
	height: 100vh;
}
</style>
