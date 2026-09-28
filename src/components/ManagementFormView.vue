<template>
  <section class="page-body inner-page form-page">
    <div class="page-heading">
      <div>
        <p class="eyebrow">ADMINISTRATION / FORM MANAGEMENT</p>
        <h1>Management Form</h1>
        <p class="page-description">Kelola 16 formulir tetap beserta publikasi RKAP dan Prognosa per bisnis area.</p>
      </div>
    </div>

    <div class="user-summary">
      <div><strong>{{ forms.length }}</strong><span>Total formulir</span></div>
      <div><strong>{{ businessAreaCount }}</strong><span>Bisnis area</span></div>
      <div>
        <strong>{{ selectedForm ? selectedForm.rkap.filter((i) => effectiveStatus(i) === 'open').length : 0 }}</strong>
        <span>RKAP terbuka pada formulir aktif</span>
      </div>
    </div>

    <div class="toolbar form-toolbar">
      <div class="search-box">
        ⌕
        <input v-model="searchQuery" placeholder="Cari nama formulir..." />
      </div>
      <span class="form-count">{{ filteredForms.length }} dari {{ forms.length }} formulir</span>
    </div>

    <section class="panel data-panel form-table-panel">
      <div class="panel-heading">
        <div>
          <h2>List Nama Formulir</h2>
          <p>Daftar formulir bersifat tetap (16 formulir) dan tidak dapat ditambah. Pilih formulir untuk melihat detail publikasi.</p>
        </div>
        <small>{{ filteredForms.length }} dari {{ forms.length }}</small>
      </div>

      <div class="table-wrap">
        <table class="form-table">
          <thead>
            <tr>
              <th>Nama Formulir</th>
              <th>Divisi Formulir</th>
              <th>Terakhir di Update</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="filteredForms.length === 0">
              <td colspan="4" class="empty-row">Formulir tidak ditemukan.</td>
            </tr>
            <tr v-for="form in filteredForms" :key="form.id" :class="{ 'selected-form-row': selectedForm?.id === form.id }">
              <td class="form-name-cell">{{ form.name }}</td>
              <td>{{ form.divisi }}</td>
              <td>{{ fmtSimpleDate(form.lastUpdated) }}</td>
              <td class="form-actions-cell">
                <button class="detail-button" type="button" @click="selectForm(form)">
                  {{ selectedForm?.id === form.id ? 'Terpilih' : 'Lihat detail' }}
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <div v-if="selectedForm" class="modal-overlay" @click.self="closeDetail">
      <div class="modal-panel detail-modal-panel">
      <div class="panel-heading">
        <div>
          <p class="eyebrow">FORM DETAIL</p>
          <h2>{{ selectedForm.name }}</h2>
          <p>Divisi {{ selectedForm.divisi }} · Terakhir diperbarui {{ fmtSimpleDate(selectedForm.lastUpdated) }}</p>
        </div>
        <div class="detail-header-actions">
          <button
            class="secondary-button"
            type="button"
            :disabled="!canEditForm"
            :title="canEditForm ? 'Edit formulir' : 'Tutup semua RKAP dan Prognosa sebelum mengedit formulir'"
            @click="openEditor"
          >
            Edit formulir
          </button>
          <button class="ghost-button small-button" type="button" @click="closeDetail">Tutup</button>
        </div>
      </div>

      <p v-if="!isPublishWindowOpen" class="window-banner">
        Publikasi RKAP untuk tahun berikutnya baru dibuka setiap tanggal 1 Agustus.
      </p>

      <div class="detail-section">
        <div class="detail-section-heading">
          <div>
            <h3>RKAP</h3>
            <p>Publikasikan RKAP tahunan per bisnis area, atau beberapa area sekaligus. Setelah dipublikasikan, RKAP terbuka otomatis selama 2 minggu.</p>
          </div>
          <div class="bulk-actions">
            <label class="select-all">
              <input
                type="checkbox"
                :checked="allEligibleSelected"
                :disabled="eligibleRkap.length === 0"
                @change="toggleSelectAll"
              />
              Pilih semua yang bisa dipublikasikan
            </label>
            <button
              class="primary-button small-button"
              type="button"
              :disabled="selectedAreas.size === 0 || !isPublishWindowOpen"
              @click="publishSelected"
            >
              Publikasikan terpilih ({{ selectedAreas.size }})
            </button>
          </div>
        </div>

        <div class="area-grid">
          <article v-for="item in selectedForm.rkap" :key="item.area" class="area-card">
            <div class="area-card-top">
              <input
                v-if="['belum', 'closed'].includes(effectiveStatus(item))"
                type="checkbox"
                :checked="selectedAreas.has(item.area)"
                @change="toggleAreaSelection(item.area)"
              />
              <h4>{{ item.area }}</h4>
            </div>
            <span class="status-badge" :class="'status-' + effectiveStatus(item)">{{ statusLabel(item) }}</span>

            <div class="area-card-actions">
              <button
                v-if="['belum', 'closed'].includes(effectiveStatus(item))"
                class="link-button"
                type="button"
                :disabled="!isPublishWindowOpen"
                @click="publishOne(item)"
              >
                Publikasikan
              </button>
              <button
                v-if="effectiveStatus(item) === 'closed'"
                class="link-button"
                type="button"
                @click="openReschedule(item)"
              >
                Atur jadwal buka kembali
              </button>
            </div>

            <div v-if="scheduleTarget === item.area" class="schedule-form">
              <label>Tanggal <input type="date" v-model="scheduleDate" /></label>
              <label>Jam <input type="time" v-model="scheduleTime" /></label>
              <div class="schedule-form-actions">
                <button class="primary-button small-button" type="button" @click="confirmReschedule(item)">Simpan jadwal</button>
                <button class="ghost-button small-button" type="button" @click="cancelReschedule">Batal</button>
              </div>
            </div>
          </article>
        </div>
      </div>

      <div class="detail-section">
        <div class="detail-section-heading">
          <div>
            <h3>Prognosa</h3>
            <p>Status Prognosa otomatis menjadi Open saat RKAP area terkait dipublikasikan, dan hanya dapat ditutup secara manual.</p>
          </div>
        </div>

        <div class="area-grid">
          <article v-for="item in selectedForm.prognosa" :key="item.area" class="area-card">
            <div class="area-card-top">
              <h4>{{ item.area }}</h4>
            </div>
            <span class="status-badge" :class="'status-' + item.status">{{ item.status === 'open' ? 'Open' : 'Closed' }}</span>
            <div class="area-card-actions">
              <button
                class="link-button"
                type="button"
                :disabled="item.status !== 'open'"
                @click="closePrognosa(item)"
              >
                Tutup prognosa
              </button>
            </div>
          </article>
        </div>
      </div>
      </div>
    </div>

    <div v-if="editingForm" class="modal-overlay" @click.self="closeEditor">
      <div class="modal-panel">
        <div class="modal-heading">
          <div>
            <p class="eyebrow">EDIT FORMULIR</p>
            <h3>{{ selectedForm?.name }}</h3>
          </div>
          <button class="ghost-button small-button" type="button" @click="closeEditor">Tutup</button>
        </div>
        <p class="modal-note">Header tabel bersifat tetap. Tambahkan baris uraian, lalu tambahkan label pada tiap baris sesuai kebutuhan.</p>
        <table class="edit-table">
          <thead>
            <tr>
              <th>No</th>
              <th>Uraian</th>
              <th>Label</th>
              <th></th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(row, index) in editableRows" :key="row.id">
              <td>{{ index + 1 }}</td>
              <td><input v-model="row.uraian" placeholder="Nama uraian" /></td>
              <td>
                <div class="label-list">
                  <input v-for="(label, li) in row.labels" :key="li" v-model="row.labels[li]" placeholder="Label" />
                  <button class="ghost-button small-button" type="button" @click="addLabel(row)">+ Label</button>
                </div>
              </td>
              <td><button class="ghost-button small-button" type="button" @click="removeRow(row)">Hapus</button></td>
            </tr>
          </tbody>
        </table>
        <button class="secondary-button" type="button" @click="addRow">+ Tambah baris</button>
      </div>
    </div>
  </section>
</template>

<script setup>
import { computed, reactive, ref } from 'vue'

const businessAreas = [
  'Daop 1 Jakarta',
  'Daop 2 Bandung',
  'Daop 3 Cirebon',
  'Daop 4 Semarang',
  'Daop 5 Purwokerto',
  'Daop 6 Yogyakarta',
  'Daop 7 Madiun',
  'Daop 8 Surabaya',
  'Daop 9 Jember',
  'Divre I Sumatera Utara',
  'Divre II Sumatera Barat',
  'Divre III Palembang',
  'Divre IV Tanjungkarang',
]
const businessAreaCount = businessAreas.length

const formBlueprints = [
  { name: 'Usulan Hari Kerja Balai Yasa', divisi: 'Balai Yasa', lastUpdated: '2026-09-18' },
  { name: 'Usulan Produksi Sarana', divisi: 'Sarana', lastUpdated: '2026-09-20' },
  { name: 'Usulan Produksi Prasarana Sintel Pemerintah', divisi: 'Prasarana', lastUpdated: '2026-09-15' },
  { name: 'Usulan Produksi Prasarana Rel KAI', divisi: 'Prasarana', lastUpdated: '2026-09-12' },
  { name: 'Usulan Produksi Prasarana Jembatan KAI', divisi: 'Prasarana', lastUpdated: '2026-09-12' },
  { name: 'Usulan Produksi Asset Prasarana Sintel', divisi: 'Prasarana', lastUpdated: '2026-09-10' },
  { name: 'Usulan Produksi Penumpang', divisi: 'Komersial', lastUpdated: '2026-09-22' },
  { name: 'Usulan Produksi Pelayanan', divisi: 'Komersial', lastUpdated: '2026-09-21' },
  { name: 'Usulan Produksi Perawatan Sarana', divisi: 'Sarana', lastUpdated: '2026-09-19' },
  { name: 'Usulan Tanggungan Rebei Yasa', divisi: 'Balai Yasa', lastUpdated: '2026-09-08' },
  { name: 'Usulan Produksi Prasarana Rel Pemerintah', divisi: 'Prasarana', lastUpdated: '2026-09-09' },
  { name: 'Usulan Produksi Prasarana Jembatan Pemerintah', divisi: 'Prasarana', lastUpdated: '2026-09-09' },
  { name: 'Usulan Produksi Asset Prasarana Jalan Jembatan', divisi: 'Prasarana', lastUpdated: '2026-09-07' },
  { name: 'Usulan Produksi Sumber Daya Manusia', divisi: 'SDM', lastUpdated: '2026-09-17' },
  { name: 'Usulan Produksi Barang', divisi: 'Logistik', lastUpdated: '2026-09-14' },
  { name: 'Usulan Norma Kendali Kecelakaan Kereta Api', divisi: 'Keselamatan & Operasi', lastUpdated: '2026-09-23' },
]

function createRkapState() {
  return businessAreas.map((area) => ({
    area,
    status: 'belum', // belum | open | closed
    deadline: null,
    scheduledStart: null,
  }))
}

function createPrognosaState() {
  return businessAreas.map((area) => ({ area, status: 'closed' })) // closed | open
}

const forms = reactive(
  formBlueprints.map((f, index) => ({
    id: index + 1,
    name: f.name,
    divisi: f.divisi,
    lastUpdated: f.lastUpdated,
    rkap: createRkapState(),
    prognosa: createPrognosaState(),
  }))
)

const now = new Date()

const searchQuery = ref('')
const selectedFormId = ref(null)
const selectedAreas = ref(new Set())
const scheduleTarget = ref(null)
const scheduleDate = ref('')
const scheduleTime = ref('')
const editingForm = ref(false)
const editableRows = reactive([])

const selectedForm = computed(() => forms.find((f) => f.id === selectedFormId.value) || null)

const filteredForms = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  if (!q) return forms
  return forms.filter((f) => f.name.toLowerCase().includes(q))
})

// RKAP untuk tahun berikutnya hanya bisa dipublikasikan mulai 1 Agustus
const isPublishWindowOpen = computed(() => now.getMonth() > 7 || (now.getMonth() === 7 && now.getDate() >= 1))

const eligibleRkap = computed(() =>
  selectedForm.value ? selectedForm.value.rkap.filter((i) => ['belum', 'closed'].includes(effectiveStatus(i))) : []
)
const allEligibleSelected = computed(
  () => eligibleRkap.value.length > 0 && eligibleRkap.value.every((i) => selectedAreas.value.has(i.area))
)

const canEditForm = computed(() => {
  if (!selectedForm.value) return false
  const rkapClosed = selectedForm.value.rkap.every((i) => ['belum', 'closed'].includes(effectiveStatus(i)))
  const prognosaClosed = selectedForm.value.prognosa.every((p) => p.status === 'closed')
  return rkapClosed && prognosaClosed
})

function fmtSimpleDate(isoStr) {
  const d = new Date(isoStr)
  return d.toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' })
}

function fmtDateTime(d) {
  if (!d) return ''
  return (
    d.toLocaleDateString('id-ID', { day: 'numeric', month: 'long', year: 'numeric' }) +
    ' ' +
    d.toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' })
  )
}

function effectiveStatus(item) {
  if (item.status === 'open') {
    if (item.scheduledStart && item.scheduledStart > now) return 'scheduled'
    if (item.deadline && item.deadline <= now) return 'closed'
    return 'open'
  }
  return item.status
}

function statusLabel(item) {
  const s = effectiveStatus(item)
  if (s === 'belum') return 'Belum dipublikasikan'
  if (s === 'scheduled') return `Terjadwal buka ${fmtDateTime(item.scheduledStart)}`
  if (s === 'open') return `Terbuka s.d. ${fmtDateTime(item.deadline)}`
  return 'Tertutup'
}

function selectForm(form) {
  selectedFormId.value = form.id
  selectedAreas.value = new Set()
  scheduleTarget.value = null
  editingForm.value = false
}

function closeDetail() {
  selectedFormId.value = null
  selectedAreas.value = new Set()
  scheduleTarget.value = null
}

function toggleAreaSelection(areaName) {
  const next = new Set(selectedAreas.value)
  if (next.has(areaName)) next.delete(areaName)
  else next.add(areaName)
  selectedAreas.value = next
}

function toggleSelectAll() {
  selectedAreas.value = allEligibleSelected.value ? new Set() : new Set(eligibleRkap.value.map((i) => i.area))
}

function publishOne(item) {
  if (!isPublishWindowOpen.value || !selectedForm.value) return
  const deadline = new Date(now)
  deadline.setDate(deadline.getDate() + 14)
  item.status = 'open'
  item.deadline = deadline
  item.scheduledStart = null
  const prognosaItem = selectedForm.value.prognosa.find((p) => p.area === item.area)
  if (prognosaItem) prognosaItem.status = 'open'
}

function publishSelected() {
  if (!selectedForm.value) return
  selectedForm.value.rkap.filter((item) => selectedAreas.value.has(item.area)).forEach(publishOne)
  selectedAreas.value = new Set()
}

function closePrognosa(item) {
  item.status = 'closed'
}

function openReschedule(item) {
  scheduleTarget.value = item.area
  scheduleDate.value = ''
  scheduleTime.value = ''
}

function cancelReschedule() {
  scheduleTarget.value = null
}

function confirmReschedule(item) {
  if (!scheduleDate.value || !scheduleTime.value) return
  const start = new Date(`${scheduleDate.value}T${scheduleTime.value}`)
  const deadline = new Date(start)
  deadline.setDate(deadline.getDate() + 14)
  item.status = 'open'
  item.scheduledStart = start
  item.deadline = deadline
  scheduleTarget.value = null
}

function openEditor() {
  if (!canEditForm.value) return
  editableRows.splice(0, editableRows.length, { id: 1, uraian: '', labels: [] })
  editingForm.value = true
}

function addRow() {
  editableRows.push({ id: editableRows.length + 1, uraian: '', labels: [] })
}

function addLabel(row) {
  row.labels.push('')
}

function removeRow(row) {
  const idx = editableRows.indexOf(row)
  if (idx > -1) editableRows.splice(idx, 1)
}

function closeEditor() {
  editingForm.value = false
}
</script>

<style scoped>
.form-toolbar { justify-content: space-between; }
.form-count { color: #718789; font-size: 11px; font-weight: 700; }
.form-table-panel { margin-top: 18px; }
.form-table { min-width: 720px; }
.form-table th:last-child,
.form-table td:last-child,
.form-actions-cell { text-align: right; }
.form-name-cell { color: #294650; font-weight: 700; }
.selected-form-row { background: #f1f8f5; }

.window-banner {
  margin: 0 0 16px;
  padding: 10px 14px;
  border-radius: 8px;
  background: #fdf3e7;
  color: #9a6a20;
  font-size: 11px;
  font-weight: 600;
}

.detail-section { margin-top: 22px; }
.detail-section:first-of-type { margin-top: 18px; }
.detail-section-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 16px;
  flex-wrap: wrap;
  margin-bottom: 12px;
}
.detail-section-heading h3 { margin: 0 0 4px; color: #294650; font-size: 15px; }
.detail-section-heading p { margin: 0; max-width: 520px; color: #7d9092; font-size: 11px; line-height: 1.5; }

.bulk-actions { display: flex; align-items: center; gap: 12px; flex-wrap: wrap; }
.select-all { display: flex; align-items: center; gap: 6px; color: #4c6265; font-size: 11px; font-weight: 600; }
.select-all input { accent-color: #2e8178; }

.small-button { padding: 7px 12px; font-size: 11px; border-radius: 6px; }
.secondary-button {
  border: 1px solid #2e8178;
  background: #fff;
  color: #2e8178;
  font-weight: 700;
  font-size: 12px;
  padding: 9px 16px;
  border-radius: 8px;
  cursor: pointer;
}
.secondary-button:disabled { opacity: 0.45; cursor: not-allowed; }
.ghost-button {
  border: 1px solid #dbe3e2;
  background: #fff;
  color: #4c6265;
  font-weight: 600;
  padding: 9px 14px;
  border-radius: 8px;
  cursor: pointer;
}
.link-button {
  border: none;
  background: none;
  color: #2e8178;
  font-weight: 700;
  font-size: 11px;
  padding: 0;
  cursor: pointer;
}
.link-button:disabled { color: #b7c2c1; cursor: not-allowed; }

.area-grid { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); gap: 12px; }
.area-card {
  display: flex;
  flex-direction: column;
  gap: 8px;
  border: 1px solid #e1e9e7;
  border-radius: 8px;
  padding: 14px;
  background: #fff;
}
.area-card-top { display: flex; align-items: center; gap: 8px; }
.area-card-top input { accent-color: #2e8178; }
.area-card-top h4 { margin: 0; color: #294650; font-size: 12px; }

.status-badge {
  align-self: flex-start;
  display: inline-block;
  border-radius: 20px;
  padding: 4px 9px;
  font-size: 9px;
  font-weight: 700;
}
.status-belum { background: #eef1f2; color: #5b6b6d; }
.status-open { background: #e5f4ef; color: #328c7e; }
.status-scheduled { background: #eaf1fb; color: #3a6ea8; }
.status-closed { background: #fdece9; color: #c1512e; }

.area-card-actions { display: flex; gap: 12px; }

.schedule-form {
  margin-top: 4px;
  padding-top: 8px;
  border-top: 1px dashed #e1e9e7;
  display: flex;
  flex-direction: column;
  gap: 8px;
}
.schedule-form label { display: flex; flex-direction: column; gap: 4px; color: #4c6265; font-size: 10px; font-weight: 600; }
.schedule-form input { border: 1px solid #dbe3e2; border-radius: 6px; padding: 6px 8px; font-size: 11px; }
.schedule-form-actions { display: flex; gap: 8px; }

.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(23, 38, 41, 0.45);
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  z-index: 50;
}
.modal-panel {
  width: 100%;
  max-width: 640px;
  max-height: 82vh;
  overflow-y: auto;
  background: #fff;
  border-radius: 12px;
  padding: 22px;
}
.detail-modal-panel { max-width: 980px; }
.detail-modal-panel .panel-heading { align-items: flex-start; }
.detail-header-actions { display: flex; align-items: center; gap: 10px; flex-wrap: wrap; }
.modal-heading { display: flex; align-items: flex-start; justify-content: space-between; gap: 12px; margin-bottom: 6px; }
.modal-heading h3 { margin: 2px 0 0; color: #294650; font-size: 15px; }
.modal-note { margin: 0 0 14px; color: #7d9092; font-size: 11px; }

.edit-table { width: 100%; border-collapse: collapse; margin-bottom: 14px; }
.edit-table th { text-align: left; font-size: 10px; color: #7d9092; padding: 6px 8px; border-bottom: 1px solid #e1e9e7; }
.edit-table td { padding: 6px 8px; vertical-align: top; border-bottom: 1px solid #f1f5f4; }
.edit-table input {
  width: 100%;
  border: 1px solid #dbe3e2;
  border-radius: 6px;
  padding: 6px 8px;
  font-size: 11px;
  margin-bottom: 6px;
}
.label-list { display: flex; flex-direction: column; }

@media (max-width: 960px) {
  .area-grid { grid-template-columns: repeat(2, minmax(0, 1fr)); }
}
@media (max-width: 760px) {
  .area-grid { grid-template-columns: 1fr; }
  .form-toolbar { align-items: stretch; flex-wrap: wrap; }
  .form-toolbar .search-box { flex: 1 1 100%; }
}
</style>