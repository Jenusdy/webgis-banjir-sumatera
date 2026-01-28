<template>
  <div class="min-h-screen flex flex-col">
    <!-- Header -->
    <header class="bg-gradient-to-r from-blue-600 to-blue-800 text-white shadow-lg z-50">
      <div class="px-4 py-3 flex items-center justify-between">
        <div class="flex items-center gap-3">
          <!-- BPS Logo -->
          <img :src="bpsLogo" alt="Logo BPS" class="w-14 h-14 rounded-lg bg-white p-1.5 shadow-md">
          <div>
            <h1 class="text-xl font-bold tracking-tight">WebGIS Bencana Sumatera</h1>
            <p class="text-blue-100 text-sm">Peta Sebaran & Dampak Banjir 2025</p>
            <p class="text-blue-200 text-xs">Dikembangkan oleh Direktorat Metodologi Statistik dan Sains Data</p>
          </div>
        </div>
        <button @click="toggleSidebar()"
          class="lg:hidden p-2 hover:bg-blue-700 rounded-lg transition-colors relative z-50">
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
          </svg>
        </button>
      </div>
    </header>

    <!-- Main Content -->
    <div class="flex-1 flex flex-col">
      <!-- Map Container -->
      <div class="flex-none flex relative h-[90vh]">
        <main class="flex-1 relative z-10 h-full">
          <MapView ref="mapView" :activeLayers="activeLayers" />

          <!-- Legend -->
          <div
            class="absolute top-6 left-6 max-w-md z-[999] bg-white/95 backdrop-blur rounded-lg shadow-xl max-w-md max-lg:right-6 max-lg:left-auto max-lg:max-w-[calc(100vw-6rem)]">
            <!-- Legend Header with Toggle -->
            <div class="p-3 border-b border-slate-200 flex items-center justify-between cursor-pointer"
              @click="legendExpanded = !legendExpanded">
              <h3 class="font-semibold text-slate-800 text-sm flex items-center gap-2">
                <svg class="w-4 h-4 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2">
                  </path>
                </svg>
                Legenda
              </h3>
              <svg class="w-4 h-4 text-slate-500 transition-transform duration-200"
                :class="{ 'rotate-180': legendExpanded }" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path>
              </svg>
            </div>

            <!-- Legend Content -->
            <div v-show="legendExpanded" class="p-4 max-h-[70vh] overflow-y-auto">
              <div class="space-y-3">
                <div v-for="layer in activeLayers" :key="layer.id" class="flex flex-col gap-2">
                  <div class="text-sm font-semibold text-slate-800 border-b border-slate-200 pb-1">{{ layer.name }}
                  </div>

                  <!-- Custom Legend for layers with predefined data -->
                  <div v-if="legendDefinitions[layer.id]" class="space-y-2 py-2">
                    <div v-for="(item, idx) in legendDefinitions[layer.id]" :key="idx" class="flex items-center gap-3">
                      <!-- Line style for boundary layers -->
                      <div v-if="item.lineStyle === 'dashed'" class="w-10 h-0.5 flex-shrink-0 border-dashed border-b-2"
                        :style="{
                          borderColor: item.lineColor
                        }"></div>
                      <div v-else-if="item.lineStyle === 'solid'"
                        class="w-10 h-0.5 flex-shrink-0 border-solid border-b-2" :style="{
                          borderColor: item.lineColor
                        }"></div>
                      <!-- Circle marker style -->
                      <div v-else-if="item.shape === 'circle'" class="w-4 h-4 rounded-full flex-shrink-0 shadow-sm"
                        :style="{
                          backgroundColor: item.color,
                          border: item.borderColor ? `3px solid ${item.borderColor}` : '2px solid #cbd5e1'
                        }"></div>
                      <!-- Fill/rectangle style for polygon layers -->
                      <div v-else class="w-10 h-10 rounded flex-shrink-0 shadow-sm" :style="{
                        backgroundColor: item.color,
                        border: item.borderColor ? `3px solid ${item.borderColor}` : '2px solid #cbd5e1'
                      }"></div>
                      <span class="text-sm text-slate-700 font-medium">{{ item.label }}</span>
                    </div>
                  </div>

                  <!-- Fallback to image legend for boundary layers or layers without predefined data -->
                  <img v-else :src="getLegendUrl(layer.id)" :alt="layer.name"
                    class="w-full rounded border border-slate-100" @error="handleLegendError" />
                </div>
                <div v-if="activeLayers.length === 0" class="text-xs text-slate-500 italic text-center py-2">
                  Tidak ada layer aktif
                </div>
              </div>
            </div>
          </div>

          <!-- Basemap Switcher -->
          <div class="absolute top-4 right-16 z-[999]">
            <select v-model="selectedBasemap"
              class="bg-white px-3 py-1.5 rounded-lg shadow text-xs border border-slate-200 focus:outline-none focus:ring-2 focus:ring-blue-500">
              <option value="osm">Peta Jalan (OSM)</option>
              <option value="hot">Peta Bantuan Kemanusiaan (HOT)</option>
            </select>
          </div>
        </main>

        <!-- Sidebar Layer Control -->
        <aside :class="[
          'fixed lg:relative right-0 top-0 h-full bg-white shadow-xl transition-transform duration-300 ease-in-out',
          'w-80 flex flex-col border-l border-slate-200 overflow-hidden min-h-0',
          sidebarOpen ? 'translate-x-0 z-50' : 'translate-x-full lg:translate-x-0 z-40'
        ]">
          <!-- Sidebar Header -->
          <div class="p-4 border-b border-slate-200 bg-slate-50">
            <h2 class="font-semibold text-slate-800 flex items-center gap-2">
              <svg class="w-5 h-5 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                  d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2">
                </path>
              </svg>
              Kontrol Layer
            </h2>
          </div>

          <!-- Layer Groups -->
          <div class="flex-1 overflow-y-auto p-4 space-y-4">
            <LayerGroup v-for="group in layerGroups" :key="group.id" :group="group" :activeLayers="activeLayers"
              @toggle-layer="toggleLayer" />
          </div>
        </aside>

        <!-- Mobile Overlay -->
        <div v-if="sidebarOpen" @click="toggleSidebar()" class="fixed inset-0 bg-black/50 z-40 lg:hidden"></div>
      </div>

      <!-- Data Table Section -->
      <DataTable />
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watch, onMounted } from 'vue'
import MapView from './components/MapView.vue'
import LayerGroup from './components/LayerGroup.vue'
import DataTable from './components/DataTable.vue'
import bpsLogo from './assets/bps-logo.svg'

const sidebarOpen = ref(false)
const legendExpanded = ref(true)
const selectedBasemap = ref('osm')
const activeLayers = reactive([])
const mapView = ref(null)

// Layer groups configuration
const layerGroups = [
  {
    id: 'batas-wilkerstat',
    name: 'Batas Wilkerstat',
    icon: 'M9 20l-5.447-2.724A1 1 0 013 16.382V5.618a1 1 0 011.447-.894L9 7m0 13l6-3m-6 3V7m6 10l4.553 2.276A1 1 0 0021 18.382V7.618a1 1 0 00-.553-.894L15 4m0 13V4m0 0L9 7',
    layers: [
      {
        id: 'provinsi',
        name: 'Batas Wilayah Provinsi',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:provinsi&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'kabupaten_kota',
        name: 'Batas Wilayah Kabupaten/Kota',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:kabupaten_kota&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'kecamatan',
        name: 'Batas Wilayah Kecamatan',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:kecamatan&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'desa_kelurahan_nagari',
        name: 'Batas Wilayah Desa/Kelurahan',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:desa_kelurahan_nagari&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      }
    ]
  },
  {
    id: 'desa-tematik',
    name: 'Desa Tematik',
    icon: 'M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z',
    layers: [
      {
        id: 'desa_pendataan_bps-stis',
        name: 'Desa Pendataan BPS-STIS',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:desa_pendataan_bps-stis&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'desa_wilayah_bencana',
        name: 'Desa Wilayah Bencana',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:desa_wilayah_bencana&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      }
    ]
  },
  {
    id: 'geotagging',
    name: 'Geotagging',
    icon: 'M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z',
    layers: [
      {
        id: 'perkiraan_bangunan_infrastruktur_terdampak',
        name: 'Perkiraan Bangunan Infrastruktur Terdampak',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:perkiraan_bangunan_infrastruktur_terdampak&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
        opacity: 1
      },
      {
        id: 'perkiraan_bangunan_tempat_tinggal_terdampak',
        name: 'Perkiraan Bangunan Tempat Tinggal Terdampak',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:perkiraan_bangunan_tempat_tinggal_terdampak&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
        opacity: 1
      }
    ]
  },
  {
    id: 'poligon-tambahan',
    name: 'Poligon Tambahan',
    icon: 'M3.055 11H5a2 2 0 012 2v1a2 2 0 002 2 2 2 0 012 2v2.945M8 3.935V5.5A2.5 2.5 0 0010.5 8h.5a2 2 0 012 2 2 2 0 104 0 2 2 0 012-2h1.064M15 20.488V18a2 2 0 012-2h3.064',
    layers: [
      {
        id: 'prediksi_wilayah_banjir',
        name: 'Prediksi Wilayah Banjir',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:prediksi_wilayah_banjir&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'lahan_baku_sawah__lbs_',
        name: 'Lahan Baku Sawah',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:lahan_baku_sawah__lbs_&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      }
    ]
  }
]

// Initialize active layers with visible layers
onMounted(() => {
  layerGroups.forEach(group => {
    group.layers.forEach(layer => {
      if (layer.visible) {
        activeLayers.push(layer)
      }
    })
  })
})

const toggleSidebar = () => {
  sidebarOpen.value = !sidebarOpen.value
}

const toggleLayer = (layer) => {
  const index = activeLayers.findIndex(l => l.id === layer.id)
  if (index > -1) {
    activeLayers.splice(index, 1)
    layer.visible = false
  } else {
    activeLayers.push(layer)
    layer.visible = true
  }
}

// Predefined legend data for each layer
const legendDefinitions = {
  'prediksi_wilayah_banjir': [
    { label: 'Prediksi Wilayah Banjir', color: 'rgba(0, 159, 238, 0.714)' }
  ],
  'lahan_baku_sawah__lbs_': [
    { label: 'Lahan Sawah Tidak Terdampak', color: 'rgb(26, 122, 20)' },
    { label: 'Lahan Sawah Terdampak', color: 'rgb(230, 162, 41)' }
  ],
  'perkiraan_bangunan_infrastruktur_terdampak': [
    { label: 'Ekonomi', color: 'rgb(57, 106, 212)', shape: 'circle', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Infra Pertanian', color: 'rgb(40, 222, 192)', shape: 'circle', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Kesehatan', color: 'rgb(149, 84, 201)', shape: 'circle', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Pendidikan Negeri', color: 'rgb(97, 236, 94)', shape: 'circle', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Pendidikan Swasta', color: 'rgb(196, 212, 73)', shape: 'circle', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Perbankan', color: 'rgb(233, 94, 43)', shape: 'circle', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Lainnya', color: 'rgb(215, 34, 139)', shape: 'circle', borderColor: 'rgb(35, 35, 35)' }
  ],
  'perkiraan_bangunan_tempat_tinggal_terdampak': [
    { label: 'Tempat Tinggal Terdampak', color: 'rgb(255, 239, 4)', shape: 'circle', borderColor: 'rgb(179, 167, 3)' }
  ],
  'desa_pendataan_bps-stis': [
    { label: 'Desa Pendataan Organik BPS', color: 'rgb(207, 80, 182)', borderColor: 'rgb(32, 6, 6)' },
    { label: 'Desa Pendataan STIS', color: 'rgb(190, 207, 80)', borderColor: 'rgb(35, 35, 35)' }
  ],
  'desa_wilayah_bencana': [
    { label: 'Desa Terkonfirmasi Terdampak (BNPB)', color: 'rgba(255, 0, 0, 0.698)', borderColor: 'rgb(32, 6, 6)' }
  ],
  'desa_kelurahan_nagari': [
    { label: 'Batas Wilayah Desa/Kelurahan', lineStyle: 'dashed', lineColor: 'rgb(215, 25, 28)' }
  ],
  'kecamatan': [
    { label: 'Batas Wilayah Kecamatan', lineStyle: 'dashed', lineColor: 'rgb(255, 127, 0)' }
  ],
  'kabupaten_kota': [
    { label: 'Batas Wilayah Kabupaten/Kota', lineStyle: 'dashed', lineColor: 'rgb(0, 0, 0)' }
  ],
  'provinsi': [
    { label: 'Batas Wilayah Provinsi', lineStyle: 'solid', lineColor: 'rgb(0, 0, 0)' }
  ]
}

// Get GeoServer Legend URL
const getLegendUrl = (layerId) => {
  const layerName = `floodmap_sumatera_2025:${layerId}`
  // Check if this is a boundary layer (batas wilayah)
  const isBoundaryLayer = layerId.startsWith('batas_wilayah_')

  // For boundary layers, don't show labels. For other layers, show labels with better formatting.
  if (isBoundaryLayer) {
    return `https://geoserver.bps.go.id/wms?REQUEST=GetLegendGraphic&LAYER=${layerName}&FORMAT=image/png&WIDTH=350&HEIGHT=60&LEGEND_OPTIONS=dx:0.5;dy:0.5;mx:4;my:4;fontSize:18;forceLabels:off`
  } else {
    return `https://geoserver.bps.go.id/wms?REQUEST=GetLegendGraphic&LAYER=${layerName}&FORMAT=image/png&WIDTH=400&HEIGHT=80&LEGEND_OPTIONS=dx:0.5;dy:0.5;mx:5;my:5;fontSize:20;forceLabels:on;fontStyle:bold;labelMargin:10`
  }
}

// Handle legend image load error
const handleLegendError = (event) => {
  event.target.style.display = 'none'
}

// Watch for basemap changes
watch(selectedBasemap, (newBasemap) => {
  if (mapView.value) {
    mapView.value.changeBasemap(newBasemap)
  }
})
</script>
