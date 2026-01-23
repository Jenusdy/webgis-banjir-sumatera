<template>
  <div class="bg-white rounded-xl border border-slate-200 overflow-hidden transition-all duration-200 hover:shadow-md">
    <!-- Group Header -->
    <button
      @click="isExpanded = !isExpanded"
      class="w-full px-4 py-3 flex items-center justify-between bg-gradient-to-r from-slate-50 to-white hover:from-slate-100 transition-colors"
    >
      <div class="flex items-center gap-3">
        <div class="p-2 rounded-lg bg-gradient-to-br from-blue-500 to-blue-600 text-white">
          <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" :d="group.icon"></path>
          </svg>
        </div>
        <span class="font-medium text-slate-800">{{ group.name }}</span>
      </div>
      <svg
        class="w-5 h-5 text-slate-500 transition-transform duration-200"
        :class="{ 'rotate-180': isExpanded }"
        fill="none"
        stroke="currentColor"
        viewBox="0 0 24 24"
      >
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path>
      </svg>
    </button>

    <!-- Layer List -->
    <div
      v-show="isExpanded"
      class="border-t border-slate-100 divide-y divide-slate-100"
    >
      <div
        v-for="layer in group.layers"
        :key="layer.id"
        class="px-4 py-3 hover:bg-slate-50 transition-colors"
      >
        <label class="flex items-center gap-3 cursor-pointer">
          <div class="relative">
            <input
              type="checkbox"
              :checked="activeLayers.some(l => l.id === layer.id)"
              @change="$emit('toggle-layer', layer)"
              class="peer w-5 h-5 rounded border-2 border-slate-300 text-blue-600 focus:ring-blue-500 focus:ring-offset-0 transition-colors cursor-pointer"
            >
            <svg
              class="w-3.5 h-3.5 text-white absolute top-0.5 left-0.5 pointer-events-none peer-checked:block hidden"
              fill="currentColor"
              viewBox="0 0 20 20"
            >
              <path fill-rule="evenodd" d="M16.707 5.293a1 1 0 010 1.414l-8 8a1 1 0 01-1.414 0l-4-4a1 1 0 011.414-1.414L8 12.586l7.293-7.293a1 1 0 011.414 0z" clip-rule="evenodd"></path>
            </svg>
          </div>
          <span class="text-sm text-slate-700 select-none">{{ layer.name }}</span>
        </label>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const props = defineProps({
  group: {
    type: Object,
    required: true
  },
  activeLayers: {
    type: Array,
    default: () => []
  }
})

defineEmits(['toggle-layer'])

const isExpanded = ref(true)
</script>
