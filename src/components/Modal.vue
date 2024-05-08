<script setup>
import cerrarModal from '../assets/img/cerrar.svg'
import Alerta from './Alerta.vue'
import { ref } from 'vue'

const error = ref('')

const props = defineProps({
	id: {
		type: String,
		required: true,
	},
	modal: {
		type: Object,
		required: true,
	},
	nombre: {
		type: String,
		required: true,
	},
	cantidad: {
		type: [String, Number],
		required: true,
	},
	categoria: {
		type: String,
		required: true,
	},
	disponible: {
		type: Number,
		required: true,
	},
})

const old = props.cantidad

const emit = defineEmits([
	'ocultar-modal',
	'guardar-gasto',
	'eliminar-gasto',
	'update:nombre',
	'update:cantidad',
	'update:categoria',
])

const agregarGasto = () => {
	const { cantidad, categoria, nombre, disponible, id } = props

	if ([nombre, cantidad, categoria].includes('')) {
		error.value = 'Todos loscampos son obligatorios'
		setTimeout(() => {
			error.value = ''
		}, 3000)
		return
	}

	if (cantidad <= 0) {
		error.value = 'Cantidad no valida'
		setTimeout(() => {
			error.value = ''
		}, 3000)
		return
	}

	if (id) {
		if (cantidad > old + disponible) {
			error.value = 'Has excedido de presupuesto'
			setTimeout(() => {
				error.value = ''
			}, 3000)
			return
		}
	} else {
		if (cantidad > disponible) {
			error.value = 'Has excedido de presupuesto'
			setTimeout(() => {
				error.value = ''
			}, 3000)
			return
		}
	}

	emit('guardar-gasto')
}
</script>

<template>
	<div class="modal">
		<div class="cerrar-modal">
			<img @click="$emit('ocultar-modal')" :src="cerrarModal" />
		</div>

		<div
			class="contenedor contenedor-formulario"
			:class="[modal.animar ? 'animar' : 'cerrar']"
		>
			<form @submit.prevent="agregarGasto" class="nuevo-gasto">
				<legend>{{ !id ? 'Añadir gasto' : 'Modificar Gasto' }}</legend>
				<Alerta v-if="error">{{ error }}</Alerta>
				<div class="campo">
					<label for="nombre">Nombre Gasto</label>
					<input
						type="text"
						id="nombre"
						placeholder="Añade el nombre del gasto"
						:value="nombre"
						@input="$emit('update:nombre', $event.target.value)"
					/>
				</div>

				<div class="campo">
					<label for="Cantidad">Cantidad</label>
					<input
						type="number"
						id="Cantidad"
						placeholder="Añade la cantidad del gasto, ej. 300"
						:value="cantidad"
						@input="$emit('update:cantidad', +$event.target.value)"
					/>
				</div>

				<div class="campo">
					<label for="categoria">Categoria</label>
					<select
						id="categoria"
						:value="categoria"
						@input="$emit('update:categoria', $event.target.value)"
					>
						<option value="">-- Seleccione --</option>
						<option value="ahorro">Ahorro</option>
						<option value="comida">Comida</option>
						<option value="casa">Casa</option>
						<option value="gastos">Gastos Varios</option>
						<option value="ocio">Ocio</option>
						<option value="salud">Salud</option>
						<option value="suscripciones">Suscripciones</option>
					</select>
				</div>

				<input
					type="submit"
					:value="[!id ? 'Añadir Gasto' : 'Guardar Cambios']"
				/>
			</form>

			<button
				type="button"
				class="btn-eliminar"
				v-if="id"
				@click="$emit('eliminar-gasto')"
			>
				Eliminar gasto
			</button>
		</div>
	</div>
</template>

<style scoped>
.modal {
	position: absolute;
	height: 100vh;
	background-color: rgb(0 0 0 / 0.9);
	top: 0;
	left: 0;
	right: 0;
	left: 0;
}

.cerrar-modal {
	position: absolute;
	right: 3rem;
	top: 3rem;
}

.cerrar-modal img {
	width: 3rem;
	cursor: pointer;
}

.contenedor-formulario {
	transition-property: all;
	transition-duration: 300ms;
	transition-timing-function: ease-in;
	opacity: 0;
}

.contenedor-formulario.animar {
	opacity: 1;
}

.contenedor-formulario.cerrar {
	opacity: 0;
}

.campo {
	display: grid;
	gap: 2rem;
}

.nuevo-gasto {
	margin: 10rem auto 0 auto;
	display: grid;
	gap: 2rem;
}

.nuevo-gasto legend {
	text-align: center;
	color: var(--blanco);
	font-size: 3rem;
	font-weight: 700;
}

.nuevo-gasto input,
.nuevo-gasto select {
	background-color: var(--gris-claro);
	border-radius: 1rem;
	padding: 1rem;
	border: none;
	font-size: 2.2rem;
}

.nuevo-gasto label {
	color: var(--blanco);
	font-size: 3rem;
}

.nuevo-gasto input[type='submit'] {
	background-color: var(--azul);
	color: var(--blanco);
	font-weight: 700;
	cursor: pointer;
}

.btn-eliminar {
	padding: 1rem;
	width: 100%;
	background-color: #ef4444;
	font-weight: 700;
	font-size: 2rem;
	margin-top: 10rem;
	cursor: pointer;
	border: none;
	color: var(--blanco);
}
</style>
