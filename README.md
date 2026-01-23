# WebGIS Bencana Sumatera

Aplikasi WebGIS untuk pemetaan sebaran dan dampak banjir di pulau Sumatera, Indonesia.

## 🌟 Fitur

- **Peta Interaktif**: Peta interaktif menggunakan Leaflet.js dengan berbagai pilihan basemap
- **Multi-Layer**: Tampilan layer banjir dari GeoServer BPS
- **Kontrol Layer**: Panel kontrol layer yang mudah digunakan dengan grouping
- **Legenda Dinamis**: Legenda otomatis dari GeoServer WMS GetLegendGraphic
- **Responsif**: Tampilan optimal di desktop dan mobile
- **Multiple Basemap**: Pilihan basemap:
  - Peta Jalan (OSM)
  - Peta Bantuan Kemanusiaan (HOT)
  - Peta Jalan (Esri)
  - Citra Satelit
  - Hibrida
  - Peta Topografi
  - Peta Topo
  - Peta Gelap

## 🗺️ Layer Tersedia

### Batas Wilkerstat
- Batas Wilayah Provinsi
- Batas Wilayah Kabupaten/Kota
- Batas Wilayah Kecamatan
- Batas Wilayah Desa/Kelurahan

### Desa Tematik
- Desa Sampel PKL Polstat STIS
- Perkiraan Desa Terdampak
- Perkiraan Jumlah Keluarga Terdampak
- Jumlah KK DTSEN

### Geotagging
- Rumah Tangga DTSEN
- Rumah Tangga Pertanian
- Infrastruktur yang Diperkirakan Terdampak

### Peta Lainnya
- Prediksi Wilayah Banjir
- Lahan Baku Sawah (LBS)

## 🛠️ Teknologi

- **Frontend Framework**: Vue 3 (Composition API)
- **Build Tool**: Vite
- **Styling**: TailwindCSS
- **Mapping Library**: Leaflet.js
- **Map Server**: GeoServer BPS

## 📦 Instalasi

### Prerequisites

- Node.js (v16 atau lebih tinggi)
- npm atau yarn

### Langkah-langkah Instalasi

1. Clone repository:
```bash
git clone <repository-url>
cd webgis-bencana-sumatera
```

2. Install dependencies:
```bash
npm install
```

3. Jalankan development server:
```bash
npm run dev
```

4. Build untuk production:
```bash
npm run build
```

5. Preview production build:
```bash
npm run preview
```

## 🚀 Penggunaan

### Mengontrol Layer

1. Buka panel **Kontrol Layer** di sebelah kanan peta
2. Klik pada nama group untuk expand/collapse layer
3. Centang/uncentang checkbox untuk menampilkan/menyembunyikan layer

### Mengubah Basemap

1. Klik dropdown basemap di pojok kanan atas
2. Pilih basemap yang diinginkan

### Menggunakan Legenda

1. Legenda ditampilkan secara otomatis di pojok kiri bawah
2. Klik header legenda untuk minimize/maximize
3. Legenda menampilkan simbol untuk semua layer yang aktif

### Informasi Koordinat

- Koordinat kursor ditampilkan di pojok kiri atas (desktop only)
- Koordinat ditampilkan dalam format desimal dengan 6 digit presisi

## 📁 Struktur Project

```
webgis-bencana-sumatera/
├── public/
├── src/
│   ├── assets/
│   │   └── bps-logo.svg          # Logo BPS
│   ├── components/
│   │   ├── MapView.vue           # Komponen peta Leaflet
│   │   └── LayerGroup.vue        # Komponen grup layer
│   ├── App.vue                   # Komponen utama
│   └── style.css                 # Global styles
├── index.html
├── package.json
├── vite.config.js
├── tailwind.config.js
└── postcss.config.js
```

## 🌐 Data Source

Data peta disediakan oleh:
- **GeoServer BPS**: https://geoserver.bps.go.id
- **Workspace**: floodmap_sumatera_2025

## 🎨 Kustomisasi

### Mengubah Layer

Edit file `src/App.vue` dan modifikasi konfigurasi `layerGroups`:

```javascript
const layerGroups = [
  {
    id: 'custom-group',
    name: 'Nama Group',
    icon: 'path-icon',
    layers: [
      {
        id: 'layer-id',
        name: 'Nama Layer',
        url: 'url-wmts-layer',
        visible: true,
        opacity: 1
      }
    ]
  }
]
```

### Mengubah Basemap

Edit file `src/components/MapView.vue` dan modifikasi objek `basemaps` dan `basemapAttributions`.

### Mengubah Extent Peta

Edit file `src/components/MapView.vue` dan modifikasi variabel `sabangToSumbarBounds`:

```javascript
const sabangToSumbarBounds = [
  [lat_min, lng_min],  // Southwest corner
  [lat_max, lng_max]   // Northeast corner
]
```

## 📱 Browser Support

- Chrome (latest)
- Firefox (latest)
- Safari (latest)
- Edge (latest)

## 🤝 Kontribusi

Kontribusi sangat diapresiasi! Silakan:
1. Fork project
2. Buat feature branch
3. Commit changes
4. Push ke branch
5. Buat Pull Request

## 📄 Lisensi

Copyright © 2025 Badan Pusat Statistik

## 👥 Pengembang

Dikembangkan oleh **Direktorat Metodologi Statistik dan Sains Data**
Badan Pusat Statistik (BPS)

## 📞 Kontak

Untuk informasi lebih lanjut, hubungi:
- Website: https://www.bps.go.id
- Email: [bps@bps.go.id](mailto:bps@bps.go.id)

## 🙏 Acknowledgments

- Data spasial dari GeoServer BPS
- Peta basemap dari OpenStreetMap, Esri, dan penyedia lainnya
- Leaflet.js untuk library mapping
- Vue.js dan Vite untuk framework frontend
- TailwindCSS untuk styling
