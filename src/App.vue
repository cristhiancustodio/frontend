<template>
	<div class="min-h-screen bg-zinc-950 text-green-400 flex flex-col items-center justify-center p-6" style="font-family: 'JetBrains Mono', monospace;">
		<!-- Google Font -->
		<link rel="preconnect" href="https://fonts.googleapis.com" />
		<link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet" />

		<!-- Header -->
		<div class="w-full max-w-3xl mb-8">
			<p class="text-xs text-zinc-500 tracking-widest uppercase mb-1">Matrix Input System</p>
			<h1 class="text-3xl font-bold text-green-300 tracking-tight">
				MTX<span class="text-zinc-600">_</span>EDITOR
			</h1>
			<div class="mt-2 h-px bg-gradient-to-r from-green-500 via-green-800 to-transparent" />
		</div>

		<!-- Dimension Controls -->
		<div class="w-full max-w-3xl mb-6">
			<div class="flex items-center gap-6 flex-wrap">
				<div class="flex flex-col gap-1">
					<label class="text-xs text-zinc-500 tracking-widest uppercase">Filas (N)</label>
					<div class="flex items-center gap-2">
						<button @click="adjustDim('rows', -1)"
							class="w-8 h-8 flex items-center justify-center bg-zinc-800 border border-zinc-700 text-green-400 rounded-sm hover:border-green-600 hover:bg-zinc-700 transition-all text-lg select-none">−</button>
						<span class="w-10 text-center text-green-300 text-lg font-bold">{{ rows }}</span>
						<button @click="adjustDim('rows', 1)"
							class="w-8 h-8 flex items-center justify-center bg-zinc-800 border border-zinc-700 text-green-400 rounded-sm hover:border-green-600 hover:bg-zinc-700 transition-all text-lg select-none">+</button>
					</div>
				</div>

				<div class="text-zinc-600 text-2xl self-end pb-1">×</div>

				<div class="flex flex-col gap-1">
					<label class="text-xs text-zinc-500 tracking-widest uppercase">Columnas (M)</label>
					<div class="flex items-center gap-2">
						<button @click="adjustDim('cols', -1)"
							class="w-8 h-8 flex items-center justify-center bg-zinc-800 border border-zinc-700 text-green-400 rounded-sm hover:border-green-600 hover:bg-zinc-700 transition-all text-lg select-none">−</button>
						<span class="w-10 text-center text-green-300 text-lg font-bold">{{ cols }}</span>
						<button @click="adjustDim('cols', 1)"
							class="w-8 h-8 flex items-center justify-center bg-zinc-800 border border-zinc-700 text-green-400 rounded-sm hover:border-green-600 hover:bg-zinc-700 transition-all text-lg select-none">+</button>
					</div>
				</div>

				<div class="flex items-end gap-3 ml-auto flex-wrap">
					<button @click="fillRandom" class="px-3 py-1.5 text-xs bg-zinc-800 border border-zinc-700 text-zinc-400 rounded-sm hover:border-zinc-500 hover:text-zinc-200 transition-all">
						⟳ Random
					</button>
					<button @click="clearMatrix" class="px-3 py-1.5 text-xs bg-zinc-800 border border-zinc-700 text-zinc-400 rounded-sm hover:border-zinc-500 hover:text-zinc-200 transition-all">
						✕ Clear
					</button>
					<div class="flex flex-col gap-1 items-center">
						<label class="text-xs text-zinc-500 tracking-widest uppercase">NxN</label>
						<button @click="squareMode = !squareMode" :class="squareMode
							? 'px-3 py-1 text-xs border rounded-sm transition-all border-green-600 text-green-400 bg-green-950'
							: 'px-3 py-1 text-xs border rounded-sm transition-all border-zinc-700 text-zinc-500 bg-zinc-800'">
							{{ squareMode ? 'ON' : 'OFF' }}
						</button>
					</div>
				</div>
			</div>
		</div>

		<!-- Matrix Grid -->
		<div class="w-full max-w-3xl mb-6 overflow-x-auto">
			<div class="inline-block border border-zinc-700 rounded-sm bg-zinc-900 p-4 min-w-full">
				<!-- Column indices -->
				<div class="flex gap-2 mb-2 pl-8">
					<span v-for="j in cols" :key="'ch-' + j" class="text-center text-xs text-zinc-600 select-none" style="width:3.5rem;">
						{{ j - 1 }}
					</span>
				</div>

				<div v-for="(row, i) in matrix" :key="i" class="flex items-center gap-2 mb-2">
					<!-- Row index -->
					<span class="w-6 text-right text-xs text-zinc-600 select-none">{{ i }}</span>

					<input v-for="(_, j) in row" :key="j" v-model.number="matrix[i][j]" type="number" :class="focusedCell?.r === i && focusedCell?.c === j
						? 'text-center text-sm bg-zinc-700 border border-green-500 text-green-300 rounded-sm outline-none transition-all'
						: 'text-center text-sm bg-zinc-800 border border-zinc-700 text-green-300 rounded-sm outline-none transition-all hover:border-zinc-500'"
						style="width:3.5rem; height:2.5rem; -moz-appearance:textfield;" @focus="focusedCell = { r: i, c: j }" @blur="focusedCell = null" @keydown="handleKeyNav($event, i, j)"
						:ref="el => setCellRef(el, i, j)" />
				</div>
			</div>
		</div>

		<!-- JSON Preview -->
		<div class="w-full max-w-3xl mb-6">
			<div class="flex items-center justify-between mb-2">
				<span class="text-xs text-zinc-500 tracking-widest uppercase">Payload JSON</span>
				<button @click="copyJSON" class="text-xs text-zinc-400 hover:text-green-400 transition-colors">
					{{ copied ? '✓ Copiado' : '⎘ Copiar' }}
				</button>
			</div>
			<pre class="bg-zinc-900 border border-zinc-700 rounded-sm p-3 text-xs text-zinc-400 overflow-x-auto leading-relaxed max-h-36 overflow-y-auto">{{ jsonPreview }}</pre>
		</div>

		<!-- Status + Send -->
		<div class="w-full max-w-3xl flex items-center gap-4 flex-wrap">
			<div class="flex-1 text-xs text-zinc-500">
				<span class="text-green-500">{{ rows }}×{{ cols }}</span>
				&nbsp;·&nbsp;{{ rows * cols }} celdas
				&nbsp;·&nbsp;
				<span :class="isValid ? 'text-green-400' : 'text-red-400'">
					{{ isValid ? '✓ Válida' : '✗ Hay celdas vacías' }}
				</span>
			</div>

			<button @click="sendMatrix" :disabled="loading || !isValid" :class="loading || !isValid
				? 'px-6 py-2.5 bg-green-500 text-zinc-950 font-bold text-sm rounded-sm tracking-wider opacity-50 cursor-not-allowed'
				: 'px-6 py-2.5 bg-green-500 text-zinc-950 font-bold text-sm rounded-sm tracking-wider hover:bg-green-400 transition-all'">
				<span v-if="loading" class="animate-pulse">Enviando...</span>
				<span v-else>Enviar Matriz →</span>
			</button>
		</div>

		<!-- Response -->
		<transition name="fade">
			<div v-if="response" class="w-full max-w-3xl mt-6">
				<div class="h-px bg-gradient-to-r from-green-800 via-green-500 to-transparent mb-4" />
				<p class="text-xs text-zinc-500 tracking-widest uppercase mb-2">Respuesta del Servidor</p>
				<pre class="bg-zinc-900 border border-green-900 rounded-sm p-3 text-xs text-green-300 overflow-x-auto">{{ response }}</pre>
			</div>
		</transition>

		<!-- Error -->
		<transition name="fade">
			<div v-if="error" class="w-full max-w-3xl mt-4">
				<pre class="bg-zinc-900 border border-red-900 rounded-sm p-3 text-xs text-red-400">ERROR: {{ error }}</pre>
			</div>
		</transition>
	</div>
</template>

<script setup>
import axios from 'axios'
import { ref, computed, watch, nextTick } from 'vue'

const rows = ref(3)
const cols = ref(3)
const squareMode = ref(false)
const matrix = ref([])
const focusedCell = ref(null)
const loading = ref(false)
const response = ref(null)
const error = ref(null)
const copied = ref(false)
const cellRefs = ref({})
const token = ref(null);
// ← Cambia esto por tu endpoint
const API_URL = import.meta.env.VITE_API_BACKEND_NODE;

function buildMatrix(r, c) {
	const prev = matrix.value
	matrix.value = Array.from({ length: r }, (_, i) =>
		Array.from({ length: c }, (_, j) => prev[i]?.[j] ?? 0)
	)
}
getToken();
buildMatrix(rows.value, cols.value)

watch([rows, cols], ([r, c]) => buildMatrix(r, c))
watch(squareMode, (on) => { if (on) cols.value = rows.value })
watch(rows, (r) => { if (squareMode.value) cols.value = r })
watch(cols, (c) => { if (squareMode.value) rows.value = c })

function adjustDim(dim, delta) {
	if (dim === 'rows') rows.value = Math.max(1, Math.min(10, rows.value + delta))
	else cols.value = Math.max(1, Math.min(10, cols.value + delta))
}

function fillRandom() {
	matrix.value = matrix.value.map(row => row.map(() => Math.floor(Math.random() * 20) - 5))
}

function clearMatrix() {
	matrix.value = matrix.value.map(row => row.map(() => 0))
}

const jsonPreview = computed(() =>
	JSON.stringify({ rows: rows.value, cols: cols.value, matrix: matrix.value }, null, 2)
)

const isValid = computed(() =>
	matrix.value.every(row => row.every(v => v !== '' && v !== null && !isNaN(v)))
)



async function getToken() {
	try {
		const { data } = await axios.get(API_URL + 'generate-token')
		token.value = data.access_token;
	} catch (error) {
		console.log(error);
	}
}
async function copyJSON() {
	await navigator.clipboard.writeText(jsonPreview.value)
	copied.value = true
	setTimeout(() => (copied.value = false), 1500)
}

async function sendMatrix() {
	if (!isValid.value) return
	loading.value = true
	response.value = null
	error.value = null
	try {

		const { data } = await axios.post(API_URL, { rows: rows.value, cols: cols.value, matrix: matrix.value },
			{
				headers: {
					Authorization: `Bearer ${token.value}`
				}
			}
		)
		response.value = data;
	} catch (e) {
		error.value = e.message
	} finally {
		loading.value = false
	}
}

function setCellRef(el, i, j) {
	if (el) cellRefs.value[`${i}-${j}`] = el
}

function handleKeyNav(e, i, j) {
	const map = { ArrowRight: [i, j + 1], ArrowLeft: [i, j - 1], ArrowDown: [i + 1, j], ArrowUp: [i - 1, j] }
	const target = map[e.key]
	if (!target) return
	const [ni, nj] = target
	if (ni >= 0 && ni < rows.value && nj >= 0 && nj < cols.value) {
		e.preventDefault()
		nextTick(() => cellRefs.value[`${ni}-${nj}`]?.focus())
	}
}
</script>

<style scoped>
input[type=number]::-webkit-outer-spin-button,
input[type=number]::-webkit-inner-spin-button {
	-webkit-appearance: none;
	margin: 0;
}

.fade-enter-active,
.fade-leave-active {
	transition: opacity 0.3s;
}

.fade-enter-from,
.fade-leave-to {
	opacity: 0;
}
</style>