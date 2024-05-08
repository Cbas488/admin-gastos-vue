<script setup>
import Alerta from './Alerta.vue'
import { ref } from 'vue'

const presupuesto = ref(0)
const error = ref('')

const emit = defineEmits(['definir-presupuesto'])

const definirPresupuesto = () => {
	if (presupuesto.value <= 0 || presupuesto.value === '') {
		error.value = 'Presupuesto no valido'

		setTimeout(() => {
			error.value = ''
		}, 3000)

		return
	}

	emit('definir-presupuesto', presupuesto.value)
}
</script>

<template>
	<form class="presupuesto" @submit.prevent="definirPresupuesto">
		<div class="campo">
			<Alerta v-if="error">
				{{ error }}
			</Alerta>
			<label for="nuevo-presupuesto">Definir presupuesto</label>
			<input
				id="nuevo-presupuesto"
				class="nuevo-presupuesto"
				placeholder="Añade tu presupuesto"
				type="number"
				v-model.number="presupuesto"
			/>
		</div>

		<input type="submit" value="Definir tu presupuesto" />
	</form>
</template>

<style scoped>
.presupuesto {
	width: 100%;
}

.presupuesto label {
	font-size: 2.8rem;
	text-align: center;
	color: var(--azul);
}

.presupuesto input[type='number'] {
	background-color: var(--gris-claro);
	border-radius: 1rem;
	padding: 1rem;
	border: none;
	font-size: 2.2rem;
	text-align: center;
}

.presupuesto input[type='submit'] {
	background-color: var(--azul);
	border: none;
	padding: 1rem;
	text-align: center;
	font-size: 2rem;
	margin-top: 2rem;
	color: var(--blanco);
	font-weight: 900;
	width: 100%;
}

.presupuesto input[type='submit']:hover {
	background-color: #1048a4;
	cursor: pointer;
	transition: background-color 300ms ease;
}

.campo {
	display: grid;
	gap: 2rem;
}

.nuevo-presupuesto {
}
</style>
