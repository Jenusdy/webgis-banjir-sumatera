<template>
  <div class="h-screen w-screen overflow-hidden flex flex-col">
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
        <button
          @click="toggleSidebar()"
          class="lg:hidden p-2 hover:bg-blue-700 rounded-lg transition-colors relative z-50"
        >
          <svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16"></path>
          </svg>
        </button>
      </div>
    </header>

    <!-- Main Content -->
    <div class="flex-1 flex relative">
      <!-- Map Container -->
      <main class="flex-1 relative z-10">
        <MapView ref="mapView" :activeLayers="activeLayers" />

        <!-- Overlay Info -->
        <div class="absolute top-4 left-4 bg-white/95 backdrop-blur rounded-lg shadow-lg p-3 z-[1000] max-w-xs max-lg:hidden">
          <div class="text-xs text-slate-500 mb-1">Koordinat</div>
          <div class="font-mono text-sm text-slate-800" id="coordinates">Hover peta untuk mendapatkan koordinat</div>
        </div>

        <!-- Legend -->
        <div class="absolute bottom-12 left-6 max-w-md z-[999] bg-white/95 backdrop-blur rounded-lg shadow-xl max-w-md max-lg:right-6 max-lg:left-auto max-lg:max-w-[calc(100vw-6rem)]">
          <!-- Legend Header with Toggle -->
          <div class="p-3 border-b border-slate-200 flex items-center justify-between cursor-pointer" @click="legendExpanded = !legendExpanded">
            <h3 class="font-semibold text-slate-800 text-sm flex items-center gap-2">
              <svg class="w-4 h-4 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"></path>
              </svg>
              Legenda
            </h3>
            <svg class="w-4 h-4 text-slate-500 transition-transform duration-200" :class="{ 'rotate-180': legendExpanded }" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7"></path>
            </svg>
          </div>

          <!-- Legend Content -->
          <div v-show="legendExpanded" class="p-4 max-h-[50vh] overflow-y-auto">
            <div class="space-y-3">
              <div v-for="layer in activeLayers" :key="layer.id" class="flex flex-col gap-2">
                <div class="text-sm font-semibold text-slate-800 border-b border-slate-200 pb-1">{{ layer.name }}</div>

                <!-- Custom Legend for layers with predefined data -->
                <div v-if="legendDefinitions[layer.id]" class="space-y-2 py-2">
                  <div v-for="(item, idx) in legendDefinitions[layer.id]" :key="idx" class="flex items-center gap-3">
                    <!-- Line style for boundary layers -->
                    <div
                      v-if="item.lineStyle === 'dashed'"
                      class="w-10 h-0.5 flex-shrink-0 border-dashed border-b-2"
                      :style="{
                        borderColor: item.lineColor
                      }"
                    ></div>
                    <div
                      v-else-if="item.lineStyle === 'solid'"
                      class="w-10 h-0.5 flex-shrink-0 border-solid border-b-2"
                      :style="{
                        borderColor: item.lineColor
                      }"
                    ></div>
                    <!-- Circle marker style -->
                    <div
                      v-else-if="item.shape === 'circle'"
                      class="w-4 h-4 rounded-full flex-shrink-0 shadow-sm"
                      :style="{
                        backgroundColor: item.color,
                        border: item.borderColor ? `3px solid ${item.borderColor}` : '2px solid #cbd5e1'
                      }"
                    ></div>
                    <!-- Fill/rectangle style for polygon layers -->
                    <div
                      v-else
                      class="w-10 h-10 rounded flex-shrink-0 shadow-sm"
                      :style="{
                        backgroundColor: item.color,
                        border: item.borderColor ? `3px solid ${item.borderColor}` : '2px solid #cbd5e1'
                      }"
                    ></div>
                    <span class="text-sm text-slate-700 font-medium">{{ item.label }}</span>
                  </div>
                </div>

                <!-- Fallback to image legend for boundary layers or layers without predefined data -->
                <img
                  v-else
                  :src="getLegendUrl(layer.id)"
                  :alt="layer.name"
                  class="w-full rounded border border-slate-100"
                  @error="handleLegendError"
                />
              </div>
              <div v-if="activeLayers.length === 0" class="text-xs text-slate-500 italic text-center py-2">
                Tidak ada layer aktif
              </div>
            </div>
          </div>
        </div>

        <!-- Basemap Switcher -->
        <div class="absolute top-4 right-16 z-[999]">
          <select
            v-model="selectedBasemap"
            class="bg-white px-3 py-1.5 rounded-lg shadow text-xs border border-slate-200 focus:outline-none focus:ring-2 focus:ring-blue-500"
          >
            <option value="osm">Peta Jalan (OSM)</option>
            <option value="hot">Peta Bantuan Kemanusiaan (HOT)</option>
          </select>
        </div>
      </main>

      <!-- Sidebar Layer Control -->
      <aside
        :class="[
          'fixed lg:relative right-0 top-0 h-full bg-white shadow-xl transition-transform duration-300 ease-in-out',
          'w-80 flex flex-col border-l border-slate-200',
          sidebarOpen ? 'translate-x-0 z-50' : 'translate-x-full lg:translate-x-0 z-40'
        ]"
      >
        <!-- Sidebar Header -->
        <div class="p-4 border-b border-slate-200 bg-slate-50">
          <h2 class="font-semibold text-slate-800 flex items-center gap-2">
            <svg class="w-5 h-5 text-blue-600" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2"></path>
            </svg>
            Kontrol Layer
          </h2>
        </div>

        <!-- Layer Groups -->
        <div class="flex-1 overflow-y-auto p-4 space-y-4">
          <LayerGroup
            v-for="group in layerGroups"
            :key="group.id"
            :group="group"
            :activeLayers="activeLayers"
            @toggle-layer="toggleLayer"
          />
        </div>
      </aside>

      <!-- Mobile Overlay -->
      <div
        v-if="sidebarOpen"
        @click="toggleSidebar()"
        class="fixed inset-0 bg-black/50 z-40 lg:hidden"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watch, onMounted } from 'vue'
import MapView from './components/MapView.vue'
import LayerGroup from './components/LayerGroup.vue'
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
        id: 'batas_wilayah_provinsi',
        name: 'Batas Wilayah Provinsi',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:batas_wilayah_provinsi&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'batas_wilayah_kabupaten_kota',
        name: 'Batas Wilayah Kabupaten/Kota',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:batas_wilayah_kabupaten_kota&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'batas_wilayah_kecamatan',
        name: 'Batas Wilayah Kecamatan',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:batas_wilayah_kecamatan&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'batas_wilayah_desa_kelurahan',
        name: 'Batas Wilayah Desa/Kelurahan',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:batas_wilayah_desa_kelurahan&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
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
        id: 'desa_sampel_pkl_polstat_stis_2026',
        name: 'Desa Sampel PKL Polstat STIS',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:desa_sampel_pkl_polstat_stis_2026&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
        opacity: 1
      },
      {
        id: 'perkiraan_desa_terdampak',
        name: 'Perkiraan Desa Terdampak',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:perkiraan_desa_terdampak&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: true,
        opacity: 1
      },
      {
        id: 'perkiraan_jumlah_keluarga_terdampak',
        name: 'Perkiraan Jumlah Keluarga Terdampak',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:perkiraan_jumlah_keluarga_terdampak&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
        opacity: 1
      },
      {
        id: 'jumlah_kk_dtsen',
        name: 'Jumlah KK DTSEN',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:jumlah_kk_dtsen&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
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
        id: 'rumah_tangga_dtsen',
        name: 'Rumah Tangga DTSEN',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:rumah_tangga_dtsen&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
        opacity: 1
      },
      {
        id: 'rumah_tangga_pertanian',
        name: 'Rumah Tangga Pertanian',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:rumah_tangga_pertanian&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
        opacity: 1
      },
      {
        id: 'infrastruktur_yang_diperkirakan_terdampak',
        name: 'Infrastruktur yang Diperkirakan Terdampak',
        url: 'https://geoserver.bps.go.id/gwc/service/wmts?layer=floodmap_sumatera_2025:infrastruktur_yang_diperkirakan_terdampak&style=&tilematrixset=WebMercatorQuad&Service=WMTS&Request=GetTile&Version=1.0.0&Format=image/png&TileMatrix={z}&TileCol={x}&TileRow={y}',
        visible: false,
        opacity: 1
      }
    ]
  },
  {
    id: 'poligon-tambahan',
    name: 'Peta Lainnya',
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
    { label: 'Prediksi Wilayah Banjir', color: 'rgba(166, 206, 227, 0.5)', borderColor: 'rgba(18, 72, 107, 1)' }
  ],
  'perkiraan_desa_terdampak': [
    { label: 'Desa yang Tidak Terdampak', color: 'rgb(74, 200, 57)', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Desa yang Diperkirakan Terdampak', color: 'rgb(179, 20, 23)', borderColor: 'rgb(128, 14, 16)' }
  ],
  'lahan_baku_sawah__lbs_': [
    { label: 'Wilayah Lahan Baku Sawah Tidak Terdampak Banjir', color: 'rgba(97, 208, 77, 0.71)', borderColor: 'rgba(36, 111, 30, 1)' },
    { label: 'Wilayah Lahan Baku Sawah yang Diperkirakan Terdampak Banjir', color: 'rgba(168, 81, 0, 0.74)', borderColor: 'rgba(215, 25, 28, 1)' },
    { label: 'Wilayah Lahan Baku Sawah yang Diperkirakan dalam Radius 500m', color: 'rgba(253, 191, 111, 0.84)', borderColor: 'rgba(255, 166, 0, 1)' }
  ],
  'jumlah_kk_dtsen': [
    { label: '0 - 200', color: 'rgb(255, 255, 255)', borderColor: 'rgb(35, 35, 35)' },
    { label: '200 - 500', color: 'rgb(255, 128, 128)', borderColor: 'rgb(35, 35, 35)' },
    { label: 'Diatas 500', color: 'rgb(255, 0, 0)', borderColor: 'rgb(35, 35, 35)' }
  ],
  'perkiraan_jumlah_keluarga_terdampak': [
    { label: 'Tidak ada Keluarga Terdampak', color: 'rgba(255, 255, 255, 0.346)' },
    { label: '0 - 500 Keluarga Terdampak', color: 'rgb(255, 128, 128)' },
    { label: 'Diatas 500 Keluarga Terdampak', color: 'rgb(255, 0, 0)' }
  ],
  'rumah_tangga_dtsen': [
    { label: 'Rumah Tangga Tani dalam Desa Bencana', color: 'rgb(255, 210, 0)', shape: 'circle', borderColor: 'rgb(255, 255, 255)' },
    { label: 'Rumah Tangga Tani yang Diperkirakan Terdampak', color: 'rgb(231, 65, 90)', shape: 'circle', borderColor: 'rgb(255, 255, 255)' }
  ],
  'rumah_tangga_pertanian': [
    { label: 'Rumah Tangga Tani dalam Desa Bencana', color: 'rgb(51, 160, 44)', shape: 'circle', borderColor: 'rgb(255, 255, 255)' },
    { label: 'Rumah Tangga Tani yang Diperkirakan Terdampak', color: 'rgb(231, 65, 90)', shape: 'circle', borderColor: 'rgb(255, 255, 255)' }
  ],
  'infrastruktur_yang_diperkirakan_terdampak': [
    { label: 'Ekonomi', color: 'rgb(255, 127, 0)', shape: 'circle' },
    { label: 'Kesehatan', color: 'rgb(176, 200, 41)', shape: 'circle' },
    { label: 'Pendidikan', color: 'rgb(152, 72, 213)', shape: 'circle' },
    { label: 'Perbankan', color: 'rgb(206, 175, 238)', shape: 'circle' }
  ],
  'desa_sampel_pkl_polstat_stis_2026': [
    { label: 'Desa Sampel PKL Polstat STIS 2026', color: 'rgb(38, 23, 206)', borderColor: 'rgb(35, 35, 35)' }
  ],
  'batas_wilayah_desa_kelurahan': [
    { label: 'Batas Wilayah Desa/Kelurahan', lineStyle: 'dashed', lineColor: 'rgb(215, 25, 28)' }
  ],
  'batas_wilayah_kecamatan': [
    { label: 'Batas Wilayah Kecamatan', lineStyle: 'dashed', lineColor: 'rgb(255, 127, 0)' }
  ],
  'batas_wilayah_kabupaten_kota': [
    { label: 'Batas Wilayah Kabupaten/Kota', lineStyle: 'dashed', lineColor: 'rgb(0, 0, 0)' }
  ],
  'batas_wilayah_provinsi': [
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
