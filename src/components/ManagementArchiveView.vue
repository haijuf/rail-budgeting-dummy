<template>
  <section class="page-body inner-page archive-page">
    <div class="page-heading">
      <div>
        <p class="eyebrow">ADMINISTRATION / DOCUMENT ARCHIVE</p>
        <h1>Management Archive</h1>
        <p class="page-description">Lihat seluruh form prognosa yang telah disetujui pada tahun-tahun sebelumnya.</p>
      </div>
    </div>

    <div class="user-summary">
      <div><strong>{{ approvedForms.length }}</strong><span>Total form approved</span></div>
      <div><strong>{{ filteredForms.length }}</strong><span>Hasil filter</span></div>
      <div><strong>{{ previousYears.length }}</strong><span>Tahun tersedia</span></div>
    </div>

    <section class="panel archive-filter-panel">
      <div class="panel-heading">
        <div><h2>Filter arsip prognosa</h2><p>Gunakan satu atau beberapa filter untuk mempersempit daftar.</p></div>
        <button v-if="hasActiveFilters" class="clear-filter-button" type="button" @click="resetFilters">Reset filter</button>
      </div>
      <div class="archive-filters">
        <label>
          <span>Nama formulir</span>
          <input v-model="filters.name" type="search" placeholder="Cari nama formulir..." />
        </label>
        <label>
          <span>Jenis sarana</span>
          <select v-model="filters.sarana">
            <option value="">Semua jenis sarana</option>
            <option v-for="item in saranaOptions" :key="item" :value="item">{{ item }}</option>
          </select>
        </label>
        <label>
          <span>Area bisnis</span>
          <select v-model="filters.area">
            <option value="">Semua area bisnis</option>
            <option v-for="item in areaOptions" :key="item" :value="item">{{ item }}</option>
          </select>
        </label>
        <label>
          <span>Balai yasa</span>
          <select v-model="filters.balai">
            <option value="">Semua balai yasa</option>
            <option v-for="item in balaiOptions" :key="item" :value="item">{{ item }}</option>
          </select>
        </label>
        <label>
          <span>Tahun</span>
          <select v-model="filters.year">
            <option value="">Semua tahun</option>
            <option v-for="year in previousYears" :key="year" :value="String(year)">{{ year }}</option>
          </select>
        </label>
      </div>
    </section>

    <section class="panel data-panel archive-table-panel">
      <div class="panel-heading">
        <div><h2>Form Prognosa Approved</h2><p>Hanya data dengan status Approved dan periode tahun sebelumnya yang ditampilkan.</p></div>
        <small>{{ filteredForms.length }} form</small>
      </div>
      <div class="table-wrap">
        <table class="archive-table">
          <thead>
            <tr>
              <th>Nama Formulir</th>
              <th>Jenis Sarana</th>
              <th>Area Bisnis</th>
              <th>Balai Yasa</th>
              <th>Tahun</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="filteredForms.length === 0">
              <td colspan="6" class="empty-row">Tidak ada form approved yang sesuai dengan filter.</td>
            </tr>
            <tr v-for="form in filteredForms" :key="form.id">
              <td class="archive-form-name">{{ form.name }}</td>
              <td>{{ form.sarana }}</td>
              <td>{{ form.area }}</td>
              <td>{{ form.balai }}</td>
              <td>{{ form.year }}</td>
              <td><span class="approved-status">{{ form.status }}</span></td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="table-footer">Menampilkan {{ filteredForms.length }} dari {{ approvedForms.length }} form approved</div>
    </section>
  </section>
</template>

<script setup>
import { computed, reactive } from 'vue'

const filters = reactive({ name: '', sarana: '', area: '', balai: '', year: '' })
const currentYear = 2025

const approvedForms = [
  { id: 1, name: 'Form Prognosa Sarana 2024', sarana: 'Lokomotif', area: 'Sarana', balai: 'Balai Yasa Yogyakarta', year: 2024, status: 'Approved' },
  { id: 2, name: 'Form Prognosa Prasarana 2024', sarana: 'Jembatan', area: 'Prasarana', balai: 'Balai Yasa Manggarai', year: 2024, status: 'Approved' },
  { id: 3, name: 'Form Prognosa Operasi 2024', sarana: 'Kereta', area: 'Operasi', balai: 'Balai Yasa Surabaya', year: 2024, status: 'Approved' },
  { id: 4, name: 'Form Prognosa Sarana 2023', sarana: 'Gerbong', area: 'Sarana', balai: 'Balai Yasa Manggarai', year: 2023, status: 'Approved' },
  { id: 5, name: 'Form Prognosa Prasarana 2023', sarana: 'Jalur dan Bangunan', area: 'Prasarana', balai: 'Balai Yasa Bandung', year: 2023, status: 'Approved' },
  { id: 6, name: 'Form Prognosa Operasi 2023', sarana: 'Kereta', area: 'Operasi', balai: 'Balai Yasa Yogyakarta', year: 2023, status: 'Approved' },
  { id: 7, name: 'Form Prognosa Sarana 2022', sarana: 'Lokomotif', area: 'Sarana', balai: 'Balai Yasa Surabaya', year: 2022, status: 'Approved' },
  { id: 8, name: 'Form Prognosa Prasarana 2022', sarana: 'Sinyal dan Telekomunikasi', area: 'Prasarana', balai: 'Balai Yasa Bandung', year: 2022, status: 'Approved' },
  { id: 9, name: 'Form Prognosa Operasi 2022', sarana: 'Gerbong', area: 'Operasi', balai: 'Balai Yasa Manggarai', year: 2022, status: 'Approved' },
  { id: 10, name: 'Form Prognosa Sarana 2021', sarana: 'Gerbong', area: 'Sarana', balai: 'Balai Yasa Yogyakarta', year: 2021, status: 'Approved' },
  { id: 11, name: 'Form Prognosa Prasarana 2021', sarana: 'Jembatan', area: 'Prasarana', balai: 'Balai Yasa Surabaya', year: 2021, status: 'Approved' },
  { id: 12, name: 'Form Prognosa Operasi 2021', sarana: 'Lokomotif', area: 'Operasi', balai: 'Balai Yasa Bandung', year: 2021, status: 'Approved' },
]

const previousYears = computed(() => [...new Set(approvedForms.filter((form) => form.year < currentYear).map((form) => form.year))].sort((a, b) => b - a))
const saranaOptions = computed(() => [...new Set(approvedForms.map((form) => form.sarana))].sort())
const areaOptions = computed(() => [...new Set(approvedForms.map((form) => form.area))].sort())
const balaiOptions = computed(() => [...new Set(approvedForms.map((form) => form.balai))].sort())
const filteredForms = computed(() => approvedForms.filter((form) => {
  const nameMatch = !filters.name || form.name.toLowerCase().includes(filters.name.trim().toLowerCase())
  return nameMatch && (!filters.sarana || form.sarana === filters.sarana) && (!filters.area || form.area === filters.area) && (!filters.balai || form.balai === filters.balai) && (!filters.year || String(form.year) === filters.year)
}))
const hasActiveFilters = computed(() => Object.values(filters).some(Boolean))

function resetFilters() {
  Object.keys(filters).forEach((key) => { filters[key] = '' })
}
</script>

<style scoped>
.archive-filter-panel { margin-bottom: 18px; }
.archive-filter-panel .panel-heading { align-items: flex-start; }
.clear-filter-button { border: 1px solid #d8e5e2; border-radius: 5px; padding: 8px 11px; background: #fff; color: #3d837b; font-size: 10px; font-weight: 700; }
.archive-filters { display: grid; grid-template-columns: 1.4fr repeat(4, 1fr); gap: 10px; }
.archive-filters label { display: flex; flex-direction: column; gap: 7px; color: #668084; font-size: 10px; font-weight: 700; }
.archive-filters input, .archive-filters select { min-width: 0; width: 100%; border: 1px solid #d9e3e1; border-radius: 6px; padding: 10px; outline: 0; background: #fff; color: #456269; font-size: 11px; }
.archive-filters input:focus, .archive-filters select:focus { border-color: #5aa39b; box-shadow: 0 0 0 3px #5aa39b18; }
.archive-table { min-width: 920px; }
.archive-form-name { color: #294650; font-weight: 700; }
.approved-status { display: inline-block; border-radius: 20px; padding: 5px 9px; background: #e5f4ef; color: #328c7e; font-size: 9px; font-weight: 700; }
@media (max-width: 1000px) { .archive-filters { grid-template-columns: repeat(3, 1fr); } }
@media (max-width: 760px) { .archive-filters { grid-template-columns: 1fr; } .archive-filter-panel .panel-heading { gap: 10px; } }
</style>
