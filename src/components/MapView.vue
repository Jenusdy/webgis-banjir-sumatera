<template>
  <div ref="mapContainer" class="w-full h-full"></div>
</template>

<script setup>
import { ref, onMounted, onUnmounted, watch, nextTick } from 'vue'
import L from 'leaflet'

const props = defineProps({
  activeLayers: {
    type: Array,
    default: () => []
  }
})

const mapContainer = ref(null)
let map = null
let basemapLayer = null
let layerObjects = {}

// Basemap configurations
const basemaps = {
  osm: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
  satellite: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
  topo: 'https://{s}.tile.opentopomap.org/{z}/{x}/{y}.png',
  dark: 'https://{s}.basemaps.cartocdn.com/dark_all/{z}/{x}/{y}{r}.png',
  streets: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Street_Map/MapServer/tile/{z}/{y}/{x}',
  terrain: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Terrain_Base/MapServer/tile/{z}/{y}/{x}',
  esri_satellite: 'https://server.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}',
  hybrid: 'https://server.arcgisonline.com/ArcGIS/rest/services/Reference/World_Boundaries_and_Places/MapServer/tile/{z}/{y}/{x}',
  hot: 'https://{s}.tile.openstreetmap.fr/hot/{z}/{x}/{y}.png'
}

const basemapAttributions = {
  osm: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors',
  satellite: 'Tiles &copy; Esri',
  topo: 'Map data: &copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>, <a href="http://viewfinderpanoramas.org">SRTM</a> | Map style: &copy; <a href="https://opentopomap.org">OpenTopoMap</a>',
  dark: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors &copy; <a href="https://carto.com/attributions">CARTO</a>',
  streets: 'Tiles &copy; Esri',
  terrain: 'Tiles &copy; Esri',
  esri_satellite: 'Tiles &copy; Esri',
  hybrid: 'Tiles &copy; Esri',
  hot: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors | Tiles courtesy of <a href="https://hot.openstreetmap.org/" target="_blank">Humanitarian OpenStreetMap Team</a>'
}

// Initialize map
const initMap = () => {
  // Create map without initial center/zoom
  map = L.map(mapContainer.value, {
    zoomControl: false,
    attributionControl: false
  })

  // Add zoom control to top-right
  L.control.zoom({
    position: 'topright'
  }).addTo(map)

  // Add scale control
  L.control.scale({
    position: 'bottomleft',
    imperial: false
  }).addTo(map)

  // Add default basemap
  basemapLayer = L.tileLayer(basemaps.osm, {
    attribution: basemapAttributions.osm,
    maxZoom: 19
  }).addTo(map)

  // Fit bounds to Pulau Sabang (Aceh) - Sumatera Barat region
  // Covers from Sabang (westernmost island) to West Sumatera province
  const sabangToSumbarBounds = [
    [-2.0, 97.0],   // Southwest corner (South Sumatera Barat)
    [6.0, 100.5]    // Northeast corner (Pulau Sabang/Aceh)
  ]
  map.fitBounds(sabangToSumbarBounds, { padding: [50, 50] })

  // Update coordinates on mouse move
  map.on('mousemove', (e) => {
    const coordDiv = document.getElementById('coordinates')
    if (coordDiv) {
      coordDiv.textContent = `${e.latlng.lat.toFixed(6)}, ${e.latlng.lng.toFixed(6)}`
    }
  })

  // Load initial layers
  loadLayers()
}

// Load WMTS layers
const loadLayers = () => {
  // Guard clause - don't proceed if map isn't initialized
  if (!map) return

  // Remove existing layers
  Object.values(layerObjects).forEach(layer => {
    if (map.hasLayer(layer)) {
      map.removeLayer(layer)
    }
  })
  layerObjects = {}

  // Sort layers: Batas Wilkerstat layers last (on top), others first
  const sortedLayers = [...props.activeLayers].sort((a, b) => {
    const aIsBoundary = a.id.startsWith('batas_wilayah_')
    const bIsBoundary = b.id.startsWith('batas_wilayah_')

    if (aIsBoundary && !bIsBoundary) return 1  // a comes after b
    if (!aIsBoundary && bIsBoundary) return -1 // b comes after a
    return 0  // keep original order
  })

  // Add active layers in sorted order
  sortedLayers.forEach(layer => {
    const tileLayer = L.tileLayer(layer.url, {
      maxZoom: 19,
      opacity: layer.opacity || 1
    })
    tileLayer.addTo(map)
    layerObjects[layer.id] = tileLayer
  })

  // Ensure basemap stays at the bottom after loading all layers
  if (basemapLayer) {
    basemapLayer.bringToBack()
  }
}

// Change basemap
const changeBasemap = (basemapKey) => {
  if (basemapLayer) {
    map.removeLayer(basemapLayer)
  }

  basemapLayer = L.tileLayer(basemaps[basemapKey], {
    attribution: basemapAttributions[basemapKey],
    maxZoom: 19
  }).addTo(map)

  // Ensure basemap stays at the bottom
  basemapLayer.bringToBack()
}

// Watch for layer changes
watch(() => props.activeLayers, () => {
  loadLayers()
}, { deep: true })

onMounted(() => {
  nextTick(() => {
    initMap()
  })
})

onUnmounted(() => {
  if (map) {
    map.remove()
  }
})

defineExpose({
  changeBasemap
})
</script>

<style>
@import 'leaflet/dist/leaflet.css';

/* Custom zoom control styling */
.leaflet-control-zoom {
  border: none !important;
  box-shadow: 0 4px 6px -1px rgb(0 0 0 / 0.1) !important;
  border-radius: 0.5rem !important;
  overflow: hidden;
}

.leaflet-control-zoom a {
  background-color: white !important;
  color: #1e293b !important;
  border: none !important;
  width: 32px !important;
  height: 32px !important;
  line-height: 32px !important;
  font-size: 18px !important;
}

.leaflet-control-zoom a:hover {
  background-color: #f1f5f9 !important;
}

/* Scale control styling */
.leaflet-control-scale {
  background-color: rgba(255, 255, 255, 0.9) !important;
  padding: 4px 8px !important;
  border-radius: 4px !important;
  border: none !important;
  box-shadow: 0 2px 4px rgb(0 0 0 / 0.1) !important;
}

/* Attribution styling */
.leaflet-control-attribution {
  background-color: rgba(255, 255, 255, 0.9) !important;
  font-size: 10px !important;
}
</style>
