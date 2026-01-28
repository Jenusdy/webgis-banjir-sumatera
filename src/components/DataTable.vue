<template>
  <div class="bg-white rounded-lg shadow-lg p-2 sm:p-4 m-2 sm:m-4 overflow-x-auto">
    <!-- Loading State -->
    <div v-if="isLoading" class="flex items-center justify-center py-12">
      <div class="flex flex-col items-center gap-3">
        <svg class="animate-spin h-8 w-8 text-blue-600" fill="none" viewBox="0 0 24 24">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
          <path class="opacity-75" fill="currentColor"
            d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z">
          </path>
        </svg>
        <span class="text-sm text-slate-600">Memuat data...</span>
      </div>
    </div>

    <!-- Error State -->
    <div v-else-if="error" class="flex items-center justify-center py-12">
      <div class="flex flex-col items-center gap-3 text-center">
        <svg class="w-12 h-12 text-red-500" fill="none" stroke="currentColor" viewBox="0 0 24 24">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
            d="M12 8v4m0 4h.01M21 12a9 9 0 11-18 0 9 9 0 0118 0z"></path>
        </svg>
        <span class="text-sm text-red-600 font-medium">Gagal memuat data</span>
        <span class="text-xs text-slate-500 max-w-md">{{ error }}</span>
      </div>
    </div>

    <!-- Data Content -->
    <div v-else>
      <!-- Header with filters -->
      <div class="flex flex-col sm:flex-row sm:items-center sm:justify-between gap-3 mb-4">
        <div>
          <h3 class="font-semibold text-slate-800 text-base sm:text-lg flex items-center gap-2">
            <svg class="w-5 h-5 text-blue-600 flex-shrink-0" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                d="M9 17v-2m3 2v-4m3 4v-6m2 10H7a2 2 0 01-2-2V5a2 2 0 012-2h5.586a1 1 0 01.707.293l5.414 5.414a1 1 0 01.293.707V19a2 2 0 01-2 2z">
              </path>
            </svg>
            <span class="truncate">Data Statistik Terkait Dampak Banjir Sumatera</span>
          </h3>
        </div>
        <div class="flex flex-col sm:flex-row gap-2">
          <input v-model="searchQuery" type="text" placeholder="Cari kabupaten/kota..."
            class="w-full sm:w-64 px-3 py-1.5 text-xs sm:text-sm border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
          <select v-model="filterProvinsi"
            class="w-full sm:w-48 px-3 py-1.5 text-xs sm:text-sm border border-slate-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500">
            <option value="">Semua Provinsi</option>
            <option v-for="prov in uniqueProvinsi" :key="prov" :value="prov">{{ prov }}</option>
          </select>
        </div>
      </div>

      <!-- Keterangan Singkatan -->
      <div class="mb-3 pb-3 border-b border-slate-200">
        <div class="flex flex-wrap gap-x-4 gap-y-2 text-[10px] sm:text-xs text-slate-600">
          <div class="flex items-center gap-1.5">
            <span class="font-semibold text-red-600">T</span>
            <span>: Terdampak</span>
          </div>
          <div class="flex items-center gap-1.5">
            <span class="font-semibold text-green-600">TT</span>
            <span>: Tidak Terdampak</span>
          </div>
          <div class="flex items-center gap-1.5">
            <span class="font-semibold text-blue-600">Tempat Tinggal</span>
            <span>: Bangunan tempat tinggal</span>
          </div>
          <div class="flex items-center gap-1.5">
            <span class="font-semibold text-slate-700">LBS</span>
            <span>: Lahan Baku Sawah (hektar)</span>
          </div>
        </div>
      </div>

      <!-- Table wrapper with responsive design -->
      <div class="overflow-x-auto rounded-lg border border-slate-200">
        <table class="w-full text-xs sm:text-sm border-collapse min-w-[1200px]">
          <thead>
            <tr class="bg-slate-50">
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-left font-semibold text-slate-700 border-b-2 border-slate-200 sticky left-0 bg-slate-50 z-10 w-8">
                No</th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-left font-semibold text-slate-700 border-b-2 border-slate-200 sticky left-8 sm:left-10 bg-slate-50 z-10 min-w-[100px]">
                Provinsi</th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-left font-semibold text-slate-700 border-b-2 border-slate-200 sticky left-[108px] sm:left-[118px] bg-slate-50 z-10 min-w-[150px]">
                Kabupaten/Kota</th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[80px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M19 21V5a2 2 0 00-2-2H7a2 2 0 00-2 2v16m14 0h2m-2 0h-5m-9 0H3m2 0h5M9 7h1m-1 4h1m4-4h1m-1 4h1m-5 10v-5a1 1 0 011-1h2a1 1 0 011 1v5m-4 0h4">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Ekonomi</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[100px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M3.055 11H5a2 2 0 012 2v1a2 2 0 002 2 2 2 0 012 2v2.945M8 3.935V5.5A2.5 2.5 0 0010.5 8h.5a2 2 0 012 2 2 2 0 104 0 2 2 0 012-2h1.064M15 20.488V18a2 2 0 012-2h3.064M21 12a9 9 0 11-18 0 9 9 0 0118 0z">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Pertanian</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[90px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Kesehatan</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[110px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Pendidikan Negeri</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[110px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Pendidikan Swasta</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[90px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Perbankan</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 bg-blue-50 min-w-[80px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Bangunan Tidak Terdampak</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 bg-red-50 min-w-[80px]">
                <div class="flex flex-col items-center gap-0.5">
                  <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                      d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6">
                    </path>
                  </svg>
                  <span class="text-[9px] sm:text-[10px]">Bangunan Terdampak</span>
                </div>
              </th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[110px]">
                LBS Tidak Terdampak (ha)</th>
              <th
                class="px-2 py-2 sm:px-3 sm:py-2 text-center font-semibold text-slate-700 border-b-2 border-slate-200 min-w-[100px]">
                LBS Terdampak (ha)</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(item, index) in paginatedData" :key="index" class="hover:bg-slate-50 transition-colors">
              <td
                class="px-2 py-2 sm:px-3 sm:py-2 border-b border-slate-200 font-medium sticky left-0 bg-white z-0 w-8 text-xs sm:text-sm">
                {{ index + 1 + (currentPage - 1) * itemsPerPage }}</td>
              <td
                class="px-2 py-2 sm:px-3 sm:py-2 border-b border-slate-200 sticky left-8 sm:left-10 bg-white z-0 text-xs sm:text-sm min-w-[100px]">
                {{ item.properties.namaprov }}</td>
              <td
                class="px-2 py-2 sm:px-3 sm:py-2 border-b border-slate-200 font-medium sticky left-[108px] sm:left-[118px] bg-white z-0 text-xs sm:text-sm min-w-[150px]">
                {{ item.properties.namakab }}</td>
              <td class="px-2 py-1 sm:px-3 sm:py-1 border-b border-slate-200 text-center min-w-[80px]">
                <div class="flex flex-col gap-0.5 text-[9px] sm:text-[10px]">
                  <div class="text-red-600 font-semibold">T: {{ item.properties.infra_ekonomi_terdampak }}</div>
                  <div class="text-green-600">TT: {{ item.properties.infra_ekonomi_tidak }}</div>
                </div>
              </td>
              <td class="px-2 py-1 sm:px-3 sm:py-1 border-b border-slate-200 text-center min-w-[100px]">
                <div class="flex flex-col gap-0.5 text-[9px] sm:text-[10px]">
                  <div class="text-red-600 font-semibold">T: {{ item.properties.infra_pertanian_terdampak }}</div>
                  <div class="text-green-600">TT: {{ item.properties.infra_pertanian_tidak }}</div>
                </div>
              </td>
              <td class="px-2 py-1 sm:px-3 sm:py-1 border-b border-slate-200 text-center min-w-[90px]">
                <div class="flex flex-col gap-0.5 text-[9px] sm:text-[10px]">
                  <div class="text-red-600 font-semibold">T: {{ item.properties.infra_kesehatan_terdampak }}</div>
                  <div class="text-green-600">TT: {{ item.properties.infra_kesehatan_tidak }}</div>
                </div>
              </td>
              <td class="px-2 py-1 sm:px-3 sm:py-1 border-b border-slate-200 text-center min-w-[110px]">
                <div class="flex flex-col gap-0.5 text-[9px] sm:text-[10px]">
                  <div class="text-red-600 font-semibold">T: {{ item.properties.infra_pendidikan_negeri_terdampak }}
                  </div>
                  <div class="text-green-600">TT: {{ item.properties.infra_pendidikan_negeri_tidak }}</div>
                </div>
              </td>
              <td class="px-2 py-1 sm:px-3 sm:py-1 border-b border-slate-200 text-center min-w-[110px]">
                <div class="flex flex-col gap-0.5 text-[9px] sm:text-[10px]">
                  <div class="text-red-600 font-semibold">T: {{ item.properties.infra_pendidikan_swasta_terdampak }}
                  </div>
                  <div class="text-green-600">TT: {{ item.properties.infra_pendidikan_swasta_tidak }}</div>
                </div>
              </td>
              <td class="px-2 py-1 sm:px-3 sm:py-1 border-b border-slate-200 text-center min-w-[90px]">
                <div class="flex flex-col gap-0.5 text-[9px] sm:text-[10px]">
                  <div class="text-red-600 font-semibold">T: {{ item.properties.infra_perbankan_terdampak }}</div>
                  <div class="text-green-600">TT: {{ item.properties.infra_perbankan_tidak }}</div>
                </div>
              </td>
              <td class="px-2 py-2 sm:px-3 sm:py-2 border-b border-slate-200 text-center bg-blue-50 min-w-[80px]">
                <div class="text-[10px] sm:text-xs font-semibold text-blue-700">{{
                  item.properties.jumlah_bangunan_tidak_terdampak.toLocaleString() }}</div>
              </td>
              <td class="px-2 py-2 sm:px-3 sm:py-2 border-b border-slate-200 text-center bg-red-50 min-w-[80px]">
                <div class="text-[10px] sm:text-xs font-semibold text-red-700">{{
                  item.properties.jumlah_bangunan_terdampak.toLocaleString() }}</div>
              </td>
              <td class="px-2 py-2 sm:px-3 sm:py-2 border-b border-slate-200 text-center min-w-[110px]">
                <div class="text-[10px] sm:text-xs text-green-700">{{
                  item.properties.area_lbs_tidak_terdampak.toFixed(2)
                }}</div>
              </td>
              <td class="px-2 py-2 sm:px-3 sm:py-2 border-b border-slate-200 text-center min-w-[100px]">
                <div class="text-[10px] sm:text-xs text-red-700">{{ item.properties.area_lbs_terdampak.toFixed(2) }}
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Pagination -->
      <div class="flex flex-col sm:flex-row items-center justify-between mt-4 pt-4 border-t border-slate-200 gap-3">
        <div class="text-[10px] sm:text-xs text-slate-600 text-center sm:text-left">
          Menampilkan {{ (currentPage - 1) * itemsPerPage + 1 }} - {{ Math.min(currentPage * itemsPerPage,
            filteredData.length) }} dari {{ filteredData.length }} data
        </div>
        <div class="flex gap-1 sm:gap-2 justify-center sm:justify-end overflow-x-auto pb-2 sm:pb-0">
          <button @click="currentPage--" :disabled="currentPage === 1"
            class="px-2 py-1 text-xs border border-slate-300 rounded hover:bg-slate-50 disabled:opacity-50 disabled:cursor-not-allowed whitespace-nowrap">
            Prev
          </button>
          <button v-for="page in visiblePages" :key="page" @click="currentPage = page" :class="[
            'px-2 py-1 text-xs border rounded whitespace-nowrap',
            currentPage === page
              ? 'bg-blue-600 text-white border-blue-600'
              : 'border-slate-300 hover:bg-slate-50'
          ]">
            {{ page }}
          </button>
          <button @click="currentPage++" :disabled="currentPage === totalPages"
            class="px-2 py-1 text-xs border border-slate-300 rounded hover:bg-slate-50 disabled:opacity-50 disabled:cursor-not-allowed whitespace-nowrap">
            Next
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const dataStatistik = ref(null)
const isLoading = ref(true)
const error = ref(null)
const searchQuery = ref('')
const filterProvinsi = ref('')
const currentPage = ref(1)
const itemsPerPage = 15

const uniqueProvinsi = computed(() => {
  if (!dataStatistik.value) return []
  const provinsi = dataStatistik.value.features.map(f => f.properties.namaprov)
  return [...new Set(provinsi)].sort()
})

const filteredData = computed(() => {
  if (!dataStatistik.value) return []
  let filtered = dataStatistik.value.features

  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase()
    filtered = filtered.filter(item =>
      item.properties.namakab.toLowerCase().includes(query) ||
      item.properties.namaprov.toLowerCase().includes(query)
    )
  }

  if (filterProvinsi.value) {
    filtered = filtered.filter(item => item.properties.namaprov === filterProvinsi.value)
  }

  return filtered
})

const totalPages = computed(() => Math.ceil(filteredData.value.length / itemsPerPage))

const paginatedData = computed(() => {
  const start = (currentPage.value - 1) * itemsPerPage
  const end = start + itemsPerPage
  return filteredData.value.slice(start, end)
})

const visiblePages = computed(() => {
  const pages = []
  const maxVisible = 5
  let startPage = Math.max(1, currentPage.value - Math.floor(maxVisible / 2))
  let endPage = Math.min(totalPages.value, startPage + maxVisible - 1)

  if (endPage - startPage + 1 < maxVisible) {
    startPage = Math.max(1, endPage - maxVisible + 1)
  }

  for (let i = startPage; i <= endPage; i++) {
    pages.push(i)
  }

  return pages
})

// Watch for filter changes to reset to page 1
import { watch } from 'vue'
watch([searchQuery, filterProvinsi], () => {
  currentPage.value = 1
})

// Fetch data on mount
onMounted(async () => {
  try {
    const response = await fetch(`data-statistik.json`)
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }
    dataStatistik.value = await response.json()
  } catch (err) {
    error.value = `Gagal memuat data: ${err.message}. Pastikan folder data tersedia.`
    console.error('Error fetching data:', err)
  } finally {
    isLoading.value = false
  }
})
</script>
