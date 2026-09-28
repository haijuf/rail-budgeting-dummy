<template>
  <div class="dash">
    <!-- Top bar -->
    <header class="topbar">
      <div class="topbar__left">
        <button class="icon-btn" aria-label="Menu">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
            <rect x="3" y="4" width="7" height="7" rx="1" stroke="currentColor" stroke-width="1.6"/>
            <rect x="14" y="4" width="7" height="7" rx="1" stroke="currentColor" stroke-width="1.6"/>
            <rect x="3" y="13" width="7" height="7" rx="1" stroke="currentColor" stroke-width="1.6"/>
            <rect x="14" y="13" width="7" height="7" rx="1" stroke="currentColor" stroke-width="1.6"/>
          </svg>
        </button>
        <span class="topbar__title">Dashboard</span>
      </div>
      <div class="topbar__right">
        <span class="user__name">{{ user.name }}</span>
        <span class="user__role">{{ user.role }}</span>
        <div class="user__avatar">{{ userInitials }}</div>
      </div>
    </header>

    <div class="breadcrumb">Dashboard</div>

    <!-- transient notice -->
    <transition name="fade">
      <div v-if="notice" class="notice" :class="notice.type">
        {{ notice.msg }}
      </div>
    </transition>

    <main class="content">
      <!-- Stat cards: total data per status -->
      <section class="stats">
        <article
          v-for="card in statCards"
          :key="card.label"
          class="stat-card"
          :style="{ background: card.color }"
        >
          <div class="stat-card__value">{{ card.value }}</div>
          <div class="stat-card__label">{{ card.label }}</div>
          <svg class="stat-card__icon" width="18" height="18" viewBox="0 0 24 24" fill="none">
            <path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8l-6-6Z" stroke="white" stroke-width="1.6" stroke-linejoin="round"/>
            <path d="M14 2v6h6" stroke="white" stroke-width="1.6" stroke-linejoin="round"/>
          </svg>
        </article>
      </section>

      <!-- Progress form: pengisian formulir per kategori -->
      <section class="panel">
        <div class="panel__heading">
          <div>
            <h2 class="panel__title">Progress Form</h2>
            <p class="panel__description">Pantau progres pengisian setiap jenis usulan anggaran.</p>
          </div>
          <div class="progress-scale" aria-label="Skala progres">
            <span>0%</span>
            <span>50%</span>
            <span>100%</span>
          </div>
        </div>
        <div class="progress-grid">
          <div class="progress-col">
            <ProgressRow v-for="item in progressLeft" :key="item.label" :item="item" />
          </div>
          <div class="progress-col">
            <ProgressRow v-for="item in progressRight" :key="item.label" :item="item" />
          </div>
        </div>
      </section>

      <!-- Approval table -->
      <section class="panel panel--table">
        <div class="table-toolbar">
          <div class="table-toolbar__info">
            <h2 class="panel__title panel__title--inline">Form Menunggu Approval</h2>
            <span class="table-toolbar__count">{{ selectedIds.size }} dipilih dari {{ pendingForms.length }}</span>
          </div>
          <div class="table-toolbar__actions">
            <button
              class="btn btn--approve"
              :disabled="selectedIds.size === 0"
              @click="bulkApprove"
            >
              Approve Terpilih
            </button>
            <button
              class="btn btn--reject"
              :disabled="selectedIds.size === 0"
              @click="bulkReject"
            >
              Reject Terpilih
            </button>
          </div>
        </div>

        <div class="table-scroll">
          <table class="table">
            <thead>
              <tr>
                <th class="table__checkbox-col">
                  <input
                    type="checkbox"
                    :checked="allSelected"
                    :disabled="pendingForms.length === 0"
                    @change="toggleSelectAll($event)"
                    aria-label="Pilih semua"
                  />
                </th>
                <th v-for="col in columns" :key="col">{{ col }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-if="pendingForms.length === 0">
                <td class="table__empty" :colspan="columns.length + 1">
                  Tidak ada form yang menunggu approval saat ini.
                </td>
              </tr>
              <tr
                v-for="(form, idx) in pendingForms"
                :key="form.id"
                :class="{ 'row--selected': isSelected(form.id) }"
              >
                <td class="table__checkbox-col">
                  <input
                    type="checkbox"
                    :checked="isSelected(form.id)"
                    @change="toggleRow(form.id)"
                    :aria-label="`Pilih ${form.namaForm}`"
                  />
                </td>
                <td>{{ idx + 1 }}</td>
                <td>{{ form.namaForm }}</td>
                <td>{{ form.jenisForm }}</td>
                <td>{{ form.divisi }}</td>
                <td>{{ form.lokasi }}</td>
                <td>
                  <span :class="{ 'sisa-waktu--urgent': form.sisaWaktuHari <= 1 }">
                    {{ form.sisaWaktu }}
                  </span>
                </td>
                <td class="table__action">
                  <button class="icon-btn icon-btn--approve" aria-label="Approve" title="Approve" @click="approveForm(form)">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                      <path d="M20 6 9 17l-5-5" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
                    </svg>
                  </button>
                  <button class="icon-btn icon-btn--reject" aria-label="Reject" title="Reject" @click="rejectForm(form)">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                      <path d="M18 6 6 18M6 6l12 12" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
                    </svg>
                  </button>
                  <button class="icon-btn icon-btn--view" aria-label="Lihat detail" title="Lihat detail" @click="$emit('view', form)">
                    <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                      <path d="M1 12s4-7 11-7 11 7 11 7-4 7-11 7-11-7-11-7Z" stroke="currentColor" stroke-width="1.6"/>
                      <circle cx="12" cy="12" r="3" stroke="currentColor" stroke-width="1.6"/>
                    </svg>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <footer class="table__footer">
          <label class="rows-per-page">
            Rows per page:
            <select v-model.number="pageSize">
              <option :value="10">10</option>
              <option :value="25">25</option>
              <option :value="50">50</option>
            </select>
          </label>
          <span class="table__range">{{ rangeLabel }}</span>
          <button class="icon-btn icon-btn--sm" :disabled="page === 1" @click="page--" aria-label="Halaman sebelumnya">‹</button>
          <button class="icon-btn icon-btn--sm" :disabled="page === totalPages" @click="page++" aria-label="Halaman berikutnya">›</button>
        </footer>
      </section>
    </main>
  </div>
</template>

<script setup>
import { computed, ref, h } from 'vue'

const emit = defineEmits(['view', 'approve', 'reject', 'approve-bulk', 'reject-bulk'])

// ---- user ----
const user = { name: 'NANDA', role: 'Super User - Anggaran' }
const userInitials = computed(() =>
  user.name.split(' ').map(w => w[0]).join('').slice(0, 2)
)

// ---- progress-per-category data (static demo content; wire to your API) ----
const progressLeft = [
  { label: 'Usulan Hasil Kerja Balai Yasa', pct: 2 },
  { label: 'Usulan Produksi Sarana', pct: 2 },
  { label: 'Usulan Pengesahan Sarana Pemerintah', pct: 0 },
  { label: 'Usulan Prasarana Hasil KAI', pct: 0 },
  { label: 'Usulan Prasarana Jembatan KAI', pct: 9.02 },
  { label: 'Usulan Produksi/Aset Prasarana Sarana', pct: 0 },
  { label: 'Usulan Produksi Penumpang', pct: 7.11 },
  { label: 'Usulan Produksi Pelayanan', pct: 0 },
]

const progressRight = [
  { label: 'Usulan Produksi Perawatan Sarana', pct: 0 },
  { label: 'Usulan Tanggungan Rehab Yasa', pct: 14.29 },
  { label: 'Usulan Prasarana Rel Pemerintah', pct: 43.33 },
  { label: 'Usulan Produksi Prasarana Jembatan Pemerintah', pct: 3.33 },
  { label: 'Usulan Produksi Aset Prasarana Jalan Jembatan', pct: 0 },
  { label: 'Usulan Produksi Sumber Daya Manusia', pct: 0 },
  { label: 'Usulan Produksi Barang', pct: 1.15 },
  { label: 'Usulan Norma Kendali Kecelakaan Kereta Api', pct: 0 },
]

// ---- master form data (single source of truth) ----
// status: 'draft' | 'approval_manager' | 'approval_vp' | 'terkirim' | 'rejected'
// jenisForm: 'Sarana' | 'Prasarana' | 'Balai Yasa'
// lokasi: nilai kontekstual untuk kolom "Area Bisnis/Jenis Sarana/Balai Yasa"
const forms = ref([
  // --- menunggu approval manager ---
  { id: 1, namaForm: 'Usulan Produksi Sarana', jenisForm: 'Sarana', divisi: 'Angkutan Barang', lokasi: 'DAOP 1 JAKARTA', sisaWaktu: '3 hari', sisaWaktuHari: 3, status: 'approval_manager' },
  { id: 2, namaForm: 'Usulan Prasarana Jembatan KAI', jenisForm: 'Prasarana', divisi: 'Prasarana', lokasi: 'DAOP 4 SEMARANG', sisaWaktu: '2 hari', sisaWaktuHari: 2, status: 'approval_manager' },
  { id: 3, namaForm: 'Usulan Produksi Penumpang', jenisForm: 'Sarana', divisi: 'Angkutan Penumpang', lokasi: 'DAOP 6 YOGYAKARTA', sisaWaktu: '5 hari', sisaWaktuHari: 5, status: 'approval_manager' },
  // --- menunggu approval VP ---
  { id: 4, namaForm: 'Usulan Hasil Kerja Balai Yasa Yogyakarta', jenisForm: 'Balai Yasa', divisi: 'Sarana', lokasi: 'Balai Yasa Yogyakarta', sisaWaktu: '1 hari', sisaWaktuHari: 1, status: 'approval_vp' },
  { id: 5, namaForm: 'Usulan Prasarana Rel Pemerintah', jenisForm: 'Prasarana', divisi: 'Prasarana', lokasi: 'DAOP 3 CIREBON', sisaWaktu: '4 hari', sisaWaktuHari: 4, status: 'approval_vp' },
  { id: 6, namaForm: 'Usulan Produksi Barang', jenisForm: 'Sarana', divisi: 'Angkutan Barang', lokasi: 'DIVRE 1 SUMATERA UTARA', sisaWaktu: '5 hari', sisaWaktuHari: 5, status: 'approval_vp' },
  { id: 7, namaForm: 'Usulan Produksi/Aset Prasarana Sarana', jenisForm: 'Prasarana', divisi: 'Prasarana', lokasi: 'DAOP 8 SURABAYA', sisaWaktu: '2 hari', sisaWaktuHari: 2, status: 'approval_vp' },
  // --- draft (belum diajukan, tidak tampil di tabel approval) ---
  { id: 8,  namaForm: 'Usulan Produksi Pelayanan', jenisForm: 'Sarana', divisi: 'Pelayanan', lokasi: 'DAOP 2 BANDUNG', sisaWaktu: '-', sisaWaktuHari: null, status: 'draft' },
  { id: 9,  namaForm: 'Usulan Pengesahan Sarana Pemerintah', jenisForm: 'Sarana', divisi: 'Sarana', lokasi: 'DAOP 5 PURWOKERTO', sisaWaktu: '-', sisaWaktuHari: null, status: 'draft' },
  { id: 10, namaForm: 'Usulan Norma Kendali Kecelakaan Kereta Api', jenisForm: 'Prasarana', divisi: 'Prasarana', lokasi: 'DAOP 7 MADIUN', sisaWaktu: '-', sisaWaktuHari: null, status: 'draft' },
  { id: 11, namaForm: 'Usulan Produksi Sumber Daya Manusia', jenisForm: 'Sarana', divisi: 'SDM', lokasi: 'DAOP 9 JEMBER', sisaWaktu: '-', sisaWaktuHari: null, status: 'draft' },
  { id: 12, namaForm: 'Usulan Prasarana Hasil KAI', jenisForm: 'Prasarana', divisi: 'Prasarana', lokasi: 'DIVRE 1 SUMATERA UTARA', sisaWaktu: '-', sisaWaktuHari: null, status: 'draft' },
  // --- sudah terkirim ---
  { id: 13, namaForm: 'Usulan Tanggungan Rehab Yasa', jenisForm: 'Balai Yasa', divisi: 'Sarana', lokasi: 'Balai Yasa Manggarai', sisaWaktu: '-', sisaWaktuHari: null, status: 'terkirim' },
  { id: 14, namaForm: 'Usulan Produksi Prasarana Jembatan Pemerintah', jenisForm: 'Prasarana', divisi: 'Prasarana', lokasi: 'DAOP 1 JAKARTA', sisaWaktu: '-', sisaWaktuHari: null, status: 'terkirim' },
])

// ---- stat cards: total data per status, computed live from `forms` ----
const statCards = computed(() => [
  { label: 'Total Form (Aktif)', value: forms.value.filter(f => f.status !== 'rejected').length, color: '#173d47' },
  { label: 'Drafting', value: forms.value.filter(f => f.status === 'draft').length, color: '#2f817a' },
  { label: 'Approval Manager', value: forms.value.filter(f => f.status === 'approval_manager').length, color: '#c77b38' },
  { label: 'Approval VP', value: forms.value.filter(f => f.status === 'approval_vp').length, color: '#bd6b4e' },
  { label: 'Terkirim', value: forms.value.filter(f => f.status === 'terkirim').length, color: '#3d9a8c' },
])

// ---- forms currently needing approval action ----
const pendingForms = computed(() =>
  forms.value.filter(f => f.status === 'approval_manager' || f.status === 'approval_vp')
)

const columns = ['No', 'Nama Form', 'Jenis Form', 'Divisi', 'Area Bisnis/Jenis Sarana/Balai Yasa', 'Sisa Waktu', 'Action']

function statusClass(status) {
  return { 'stage-manager': status === 'approval_manager', 'stage-vp': status === 'approval_vp' }
}

function progressColor(value) {
  if (value === 0) return '#c7d5d2'
  if (value < 10) return '#e3a04a'
  if (value < 50) return '#3d9a8c'
  return '#2f817a'
}

// ---- multi-select ----
const selectedIds = ref(new Set())

function isSelected(id) {
  return selectedIds.value.has(id)
}
function toggleRow(id) {
  const next = new Set(selectedIds.value)
  next.has(id) ? next.delete(id) : next.add(id)
  selectedIds.value = next
}
function toggleSelectAll(event) {
  selectedIds.value = event.target.checked
    ? new Set(pendingForms.value.map(f => f.id))
    : new Set()
}
const allSelected = computed(
  () => pendingForms.value.length > 0 && pendingForms.value.every(f => selectedIds.value.has(f.id))
)

// ---- notice banner ----
const notice = ref(null)
let noticeTimer = null
function showNotice(msg, type = 'success') {
  notice.value = { msg, type }
  clearTimeout(noticeTimer)
  noticeTimer = setTimeout(() => (notice.value = null), 3000)
}

// ---- approve / reject workflow ----
// alur: approval_manager -> approval_vp -> terkirim
function advanceStatus(form) {
  if (form.status === 'approval_manager') form.status = 'approval_vp'
  else if (form.status === 'approval_vp') form.status = 'terkirim'
}
function deselect(id) {
  const next = new Set(selectedIds.value)
  next.delete(id)
  selectedIds.value = next
}

function approveForm(form) {
  advanceStatus(form)
  deselect(form.id)
  showNotice(`"${form.namaForm}" disetujui.`, 'success')
  emit('approve', [form.id])
}
function rejectForm(form) {
  if (!confirm(`Tolak formulir "${form.namaForm}"?`)) return
  form.status = 'rejected'
  deselect(form.id)
  showNotice(`"${form.namaForm}" ditolak.`, 'reject')
  emit('reject', [form.id])
}

function bulkApprove() {
  const targets = pendingForms.value.filter(f => selectedIds.value.has(f.id))
  if (targets.length === 0) return
  targets.forEach(advanceStatus)
  const ids = targets.map(f => f.id)
  selectedIds.value = new Set()
  showNotice(`${targets.length} formulir disetujui.`, 'success')
  emit('approve-bulk', ids)
}
function bulkReject() {
  const targets = pendingForms.value.filter(f => selectedIds.value.has(f.id))
  if (targets.length === 0) return
  if (!confirm(`Tolak ${targets.length} formulir terpilih?`)) return
  const ids = targets.map(f => f.id)
  targets.forEach(f => (f.status = 'rejected'))
  selectedIds.value = new Set()
  showNotice(`${targets.length} formulir ditolak.`, 'reject')
  emit('reject-bulk', ids)
}

// ---- pagination footer (client-side; swap for server pagination as needed) ----
const page = ref(1)
const pageSize = ref(10)
const totalPages = computed(() => Math.max(1, Math.ceil(pendingForms.value.length / pageSize.value)))
const rangeLabel = computed(() => {
  if (pendingForms.value.length === 0) return '0 of 0'
  const start = (page.value - 1) * pageSize.value + 1
  const end = Math.min(page.value * pageSize.value, pendingForms.value.length)
  return `${start}-${end} of ${pendingForms.value.length}`
})

function formatPct(value) {
  return `${Number(value.toFixed(2))}%`
}

// ---- small inline sub-component for a progress row ----
const ProgressRow = {
  props: { item: { type: Object, required: true } },
  setup(props) {
    return () =>
      h('div', { class: 'progress-row' }, [
        h('div', { class: 'progress-row__top' }, [
          h('span', { class: 'progress-row__label' }, props.item.label),
        ]),
        h('div', { class: 'progress-row__bar' }, [
          h('div', {
            class: 'progress-row__fill',
            style: {
              width: `${Math.min(props.item.pct, 100)}%`,
              minWidth: props.item.pct > 0 ? '4px' : '0',
              background: progressColor(props.item.pct),
            },
          }),
        ]),
        h('span', { class: 'progress-row__pct' }, formatPct(props.item.pct)),
        h(
          'button',
          { class: 'icon-btn icon-btn--sm', 'aria-label': 'Info', type: 'button' },
          [
            h('svg', { width: 14, height: 14, viewBox: '0 0 24 24', fill: 'none' }, [
              h('circle', { cx: 12, cy: 12, r: 9, stroke: 'currentColor', 'stroke-width': 1.6 }),
              h('path', { d: 'M12 11v5M12 8h.01', stroke: 'currentColor', 'stroke-width': 1.6, 'stroke-linecap': 'round' }),
            ]),
          ]
        ),
      ])
  },
}
</script>

<style scoped>
* { box-sizing: border-box; }

.dash {
  font-family: 'Trebuchet MS', 'Segoe UI', sans-serif;
  background: #f5f7f8;
  min-height: 100vh;
  color: #17313b;
}

/* Top bar */
.topbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #fbfcfc;
  padding: 14px 24px;
  border-bottom: 1px solid #e4e9e9;
}
.topbar__left { display: flex; align-items: center; gap: 12px; }
.topbar__title { font-size: 18px; font-weight: 700; color: #173d47; }
.topbar__right { display: flex; align-items: center; gap: 8px; }
.user__name { font-weight: 700; font-size: 13px; }
.user__role { font-size: 11px; color: #819196; margin-right: 4px; }
.user__avatar {
  width: 32px; height: 32px; border-radius: 50%;
  background: #e3a04a; color: #17313b; font-size: 12px; font-weight: 700;
  display: flex; align-items: center; justify-content: center;
}

.icon-btn {
  border: none; background: transparent; cursor: pointer;
  color: #587176; display: inline-flex; align-items: center; justify-content: center;
  padding: 4px; border-radius: 6px;
}
.icon-btn:hover { background: #e8f1ef; }
.icon-btn--sm { color: #2f817a; }
.icon-btn--approve { color: #16a34a; }
.icon-btn--approve:hover { background: #e7f7ee; }
.icon-btn--reject { color: #d0453d; }
.icon-btn--reject:hover { background: #fbeceb; }
.icon-btn--view { color: #2f817a; }
.icon-btn:disabled { opacity: .35; cursor: default; }
.icon-btn:disabled:hover { background: transparent; }

.breadcrumb {
  padding: 10px 24px 0;
  font-size: 12px;
  color: #8a999d;
}

/* notice banner */
.notice {
  margin: 10px 24px 0;
  padding: 10px 16px;
  border-radius: 8px;
  font-size: 12.5px;
  font-weight: 600;
}
.notice.success { background: #e7f7ee; color: #157347; }
.notice.reject { background: #fbeceb; color: #b3261e; }
.fade-enter-active, .fade-leave-active { transition: opacity .2s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }

.content { max-width: 1450px; margin: 0 auto; padding: 16px 42px 42px; display: flex; flex-direction: column; gap: 18px; }

/* Stat cards */
.stats {
  display: grid;
  grid-template-columns: repeat(5, minmax(140px, 1fr));
  gap: 14px;
}
.stat-card {
  position: relative;
  border-radius: 10px;
  padding: 16px 18px;
  color: #fff;
  min-height: 84px;
  box-shadow: 0 8px 18px rgba(31, 75, 76, .12);
}
.stat-card__value { font-size: 30px; font-weight: 700; line-height: 1.1; }
.stat-card__label { font-size: 13px; margin-top: 4px; opacity: .95; }
.stat-card__icon { position: absolute; top: 12px; right: 14px; opacity: .85; }

/* Panels */
.panel {
  background: #fff;
  border-radius: 10px;
  padding: 18px 22px;
  border: 1px solid #e1e8e7;
  box-shadow: 0 6px 18px rgba(47, 79, 77, .05);
}
.panel__heading { display: flex; align-items: flex-start; justify-content: space-between; gap: 20px; margin-bottom: 12px; }
.panel__title { font-size: 15px; font-weight: 700; margin: 0; color: #203641; }
.panel__title--inline { margin-bottom: 2px; }
.panel__description { margin: 4px 0 0; color: #98a5a7; font-size: 10px; }
.progress-scale { display: flex; justify-content: space-between; width: min(190px, 30%); padding-top: 2px; color: #9aa8a9; font-size: 9px; }

.progress-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  column-gap: 32px;
}
.progress-col { display: flex; flex-direction: column; }

.progress-row {
  display: grid;
  grid-template-columns: minmax(0, 1fr) 52px 24px;
  align-items: center;
  column-gap: 10px;
  padding: 11px 0;
  border-bottom: 1px solid #edf1f0;
}
.progress-row:last-child { border-bottom: none; }
.progress-row__top { grid-column: 1 / span 3; }
.progress-row__label { font-size: 11px; color: #36545a; }
.progress-row__bar {
  grid-column: 1 / span 1;
  height: 8px;
  background: #e7efed;
  border-radius: 5px;
  overflow: hidden;
}
.progress-row__fill { height: 100%; border-radius: 5px; transition: width .2s ease, background .2s ease; }
.progress-row__pct { color: #527176; font-size: 10px; font-weight: 700; text-align: right; white-space: nowrap; }

/* Table toolbar */
.table-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 16px 22px 0;
  flex-wrap: wrap;
}
.table-toolbar__info { display: flex; flex-direction: column; }
.table-toolbar__count { font-size: 11px; color: #8a999d; }
.table-toolbar__actions { display: flex; gap: 10px; }

.btn {
  border: none;
  border-radius: 6px;
  padding: 8px 16px;
  font-size: 12.5px;
  font-weight: 700;
  cursor: pointer;
  color: #fff;
}
.btn:disabled { opacity: .4; cursor: default; }
.btn--approve { background: #16a34a; }
.btn--approve:not(:disabled):hover { background: #128a3e; }
.btn--reject { background: #d0453d; }
.btn--reject:not(:disabled):hover { background: #b3372f; }

/* Table */
.panel--table { padding: 0 0 4px; overflow: hidden; }
.table-scroll { overflow-x: auto; margin-top: 12px; }
.table { width: 100%; border-collapse: collapse; font-size: 12.5px; min-width: 720px; }
.table thead th {
  background: #173d47;
  color: #fff;
  text-align: left;
  padding: 12px 16px;
  font-weight: 600;
  white-space: nowrap;
}
.table tbody td {
  padding: 12px 16px;
  border-bottom: 1px solid #eef0f4;
  color: #3a3f52;
  vertical-align: middle;
}
.table tbody tr:hover { background: #f8f9fc; }
.table tbody tr.row--selected { background: #eef8f4; }
.table__checkbox-col { width: 40px; text-align: center; padding-left: 18px !important; padding-right: 6px !important; }
.table__action { text-align: center; white-space: nowrap; }
.table__empty { text-align: center; color: #8a999d; padding: 28px 16px !important; }

.cell-stack { display: flex; flex-direction: column; gap: 4px; }

.badge {
  display: inline-block;
  padding: 3px 12px;
  border-radius: 12px;
  font-size: 11px;
  font-weight: 600;
  color: #fff;
  width: fit-content;
}
.badge--stage { font-size: 10px; padding: 2px 9px; }
.badge.stage-manager { background: #c77b38; }
.badge.stage-vp { background: #bd6b4e; }

.sisa-waktu--urgent { color: #d0453d; font-weight: 700; }

.table__footer {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  gap: 16px;
  padding: 12px 16px;
  font-size: 12px;
  color: #6b7280;
}
.rows-per-page select {
  margin-left: 6px;
  border: 1px solid #e2e4eb;
  border-radius: 6px;
  padding: 2px 6px;
  font-size: 12px;
}

/* Responsive */
@media (max-width: 960px) {
  .stats { grid-template-columns: repeat(2, 1fr); }
  .progress-grid { grid-template-columns: 1fr; }
}
@media (max-width: 560px) {
  .stats { grid-template-columns: 1fr; }
  .content { padding: 12px 18px 32px; }
  .panel__heading { display: block; }
  .progress-scale { width: 100%; margin-top: 12px; }
  .table-toolbar { flex-direction: column; align-items: flex-start; }
}
</style>