<script setup>
import { computed, ref } from 'vue'

const props = defineProps({
  modelValue: {
    type: String,
    default: ''
  },
  min: {
    type: String,
    default: ''
  },
  max: {
    type: String,
    default: ''
  },
  id: {
    type: String,
    default: ''
  }
})

const emit = defineEmits(['update:modelValue'])

const meses = [
  'Janeiro',
  'Fevereiro',
  'Março',
  'Abril',
  'Maio',
  'Junho',
  'Julho',
  'Agosto',
  'Setembro',
  'Outubro',
  'Novembro',
  'Dezembro'
]

const aberto = ref(false)
const anoExibido = ref(new Date().getFullYear())

const anoMinimo = computed(() => Number((props.min || '').slice(0, 4)) || new Date().getFullYear())
const anoMaximo = computed(() => Number((props.max || '').slice(0, 4)) || anoMinimo.value + 20)

const valorExibido = computed(() => {
  if (!/^\d{4}-\d{2}$/.test(props.modelValue)) return 'Selecione mês/ano'

  const [ano, mes] = props.modelValue.split('-')
  return `${mes}/${ano}`
})

const abrir = () => {
  const anoSelecionado = Number((props.modelValue || props.min || '').slice(0, 4))
  anoExibido.value = anoSelecionado || anoMinimo.value
  aberto.value = !aberto.value
}

const mesPermitido = (mes) => {
  const valor = `${anoExibido.value}-${String(mes + 1).padStart(2, '0')}`
  return (!props.min || valor >= props.min) && (!props.max || valor <= props.max)
}

const selecionarMes = (mes) => {
  if (!mesPermitido(mes)) return

  emit('update:modelValue', `${anoExibido.value}-${String(mes + 1).padStart(2, '0')}`)
  aberto.value = false
}
</script>

<template>
  <div class="relative">
    <button
      :id="id"
      type="button"
      class="w-full appearance-auto bg-gray-700 border border-gray-600 rounded-lg px-3 py-2 text-left text-white focus:outline-none focus:border-emerald-500 transition-colors flex items-center justify-between gap-2"
      :aria-expanded="aberto"
      :aria-label="`Selecionar início: ${valorExibido}`"
      @click="abrir"
    >
      <span :class="modelValue ? 'text-white' : 'text-gray-500'">{{ valorExibido }}</span>
      <span aria-hidden="true" class="text-gray-300">▣</span>
    </button>

    <div
      v-if="aberto"
      class="absolute z-20 mt-2 w-full min-w-[250px] rounded-lg border border-gray-600 bg-gray-800 p-3 shadow-xl"
      role="dialog"
      aria-label="Calendário de mês e ano"
    >
      <div class="flex items-center justify-between mb-3">
        <button
          type="button"
          class="rounded px-2 py-1 text-lg text-gray-300 hover:bg-gray-700 disabled:opacity-30"
          :disabled="anoExibido <= anoMinimo"
          aria-label="Ano anterior"
          @click="anoExibido--"
        >
          ‹
        </button>
        <strong class="text-sm text-white">{{ anoExibido }}</strong>
        <button
          type="button"
          class="rounded px-2 py-1 text-lg text-gray-300 hover:bg-gray-700 disabled:opacity-30"
          :disabled="anoExibido >= anoMaximo"
          aria-label="Próximo ano"
          @click="anoExibido++"
        >
          ›
        </button>
      </div>

      <div class="grid grid-cols-3 gap-2">
        <button
          v-for="(mes, indice) in meses"
          :key="mes"
          type="button"
          class="rounded px-2 py-2 text-xs transition-colors"
          :class="mesPermitido(indice)
            ? 'text-gray-200 hover:bg-emerald-500 hover:text-gray-900'
            : 'cursor-not-allowed text-gray-600'"
          :disabled="!mesPermitido(indice)"
          @click="selecionarMes(indice)"
        >
          {{ mes }}
        </button>
      </div>
    </div>
  </div>
</template>
