<template>
  <div v-if="show" class="modal-backdrop" @click.self="handleClose">
    <section class="user-modal add-user-modal" role="dialog" aria-modal="true">
      <h2>Tambah user baru</h2>
      <p class="modal-subtitle">
        Masukkan NIPP karyawan &mdash; nama, perusahaan, sub area, dan jabatan akan terisi otomatis dari data
        pegawai. Tambahkan beberapa baris sekaligus jika perlu.
      </p>

      <div class="add-user-rows">
        <div class="add-user-row-header">
          <span>NIPP</span>
          <span>Nama (otomatis)</span>
          <span>Email</span>
          <span></span>
        </div>

        <div v-for="(row, index) in rows" :key="row.id" class="add-user-row">
          <div class="field">
            <input
              v-model="row.npp"
              type="text"
              inputmode="numeric"
              placeholder="Contoh: 12345678"
              :class="{ 'field--error': row.touched && nippError(row) }"
              @input="onNippInput(row)"
              @blur="row.touched = true"
            />
            <small v-if="row.touched && nippError(row)" class="field-error">{{ nippError(row) }}</small>
          </div>

          <div class="field">
            <input :value="row.name || '-'" type="text" readonly class="readonly-field" />
          </div>

          <div class="field">
            <input
              v-model="row.email"
              type="email"
              placeholder="nama@kai.id"
              :class="{ 'field--error': row.touched && emailError(row) }"
              @blur="row.touched = true"
            />
            <small v-if="row.touched && emailError(row)" class="field-error">{{ emailError(row) }}</small>
          </div>

          <button
            class="remove-row-button"
            type="button"
            :disabled="rows.length === 1"
            @click="removeRow(row.id)"
            aria-label="Hapus baris"
          >
            ✕
          </button>
        </div>
      </div>

      <button type="button" class="add-row-button" @click="addRow">+ Tambah baris</button>

      <p v-if="formError" class="form-error">{{ formError }}</p>

      <div class="modal-actions">
        <button class="cancel-button" @click="handleClose">Batal</button>
        <button class="confirm-activate-button" @click="handleSubmit">Tambahkan User</button>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'

const props = defineProps({
  show: Boolean,
  existingNpps: { type: Array, default: () => [] },
  employeeDirectory: { type: Object, default: () => ({}) },
})
const emit = defineEmits(['close', 'submit'])

const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/

function emptyRow() {
  return {
    id: crypto.randomUUID ? crypto.randomUUID() : `row-${Date.now()}-${Math.random()}`,
    npp: '',
    name: '',
    company: '',
    area: '',
    position: '',
    email: '',
    touched: false,
  }
}

const rows = ref([emptyRow()])
const formError = ref('')

watch(
  () => props.show,
  (visible) => {
    if (visible) {
      rows.value = [emptyRow()]
      formError.value = ''
    }
  }
)

function onNippInput(row) {
  const digitsOnly = row.npp
  const match = /^\d{6,9}$/.test(digitsOnly) ? props.employeeDirectory[digitsOnly] : null
  if (match) {
    row.name = match.name
    row.company = match.company
    row.area = match.area
    row.position = match.position
  } else {
    row.name = ''
    row.company = ''
    row.area = ''
    row.position = ''
  }
}

function nippError(row) {
  if (!row.npp) return 'NIPP wajib diisi'
  if (!/^\d+$/.test(row.npp)) return 'NIPP harus berupa angka'
  if (row.npp.length < 6 || row.npp.length > 9) return 'NIPP harus 6-9 digit'
  if (!props.employeeDirectory[row.npp]) return 'NIPP tidak ditemukan di data pegawai'
  return ''
}

function emailError(row) {
  if (!row.email) return 'Email wajib diisi'
  if (!emailRegex.test(row.email)) return 'Format email tidak valid'
  return ''
}

function addRow() {
  rows.value.push(emptyRow())
}

function removeRow(id) {
  if (rows.value.length === 1) return
  rows.value = rows.value.filter((row) => row.id !== id)
}

function handleClose() {
  emit('close')
}

function handleSubmit() {
  formError.value = ''
  rows.value.forEach((row) => (row.touched = true))

  const hasFieldErrors = rows.value.some((row) => nippError(row) || emailError(row))
  if (hasFieldErrors) {
    formError.value = 'Perbaiki data yang belum valid sebelum melanjutkan.'
    return
  }

  const nppsInBatch = rows.value.map((row) => row.npp)
  const hasDuplicateInBatch = new Set(nppsInBatch).size !== nppsInBatch.length
  if (hasDuplicateInBatch) {
    formError.value = 'Terdapat NIPP yang sama di lebih dari satu baris.'
    return
  }

  const alreadyRegistered = nppsInBatch.find((npp) => props.existingNpps.includes(npp))
  if (alreadyRegistered) {
    formError.value = `NIPP ${alreadyRegistered} sudah terdaftar sebagai user.`
    return
  }

  const newUsers = rows.value.map((row) => ({
    npp: row.npp,
    name: row.name,
    company: row.company,
    area: row.area,
    position: row.position,
    email: row.email,
    status: 'Active',
  }))

  emit('submit', newUsers)
}
</script>

<style scoped>
.add-user-modal { width: min(720px, 92vw); max-height: 88vh; overflow-y: auto; }
.modal-subtitle { color: #6b7583; font-size: 12.5px; margin: 4px 0 16px; }

.add-user-rows { display: flex; flex-direction: column; gap: 10px; }
.add-user-row-header,
.add-user-row {
  display: grid;
  grid-template-columns: 1fr 1.2fr 1.3fr 32px;
  gap: 10px;
  align-items: start;
}
.add-user-row-header {
  font-size: 11px;
  font-weight: 700;
  color: #8a999d;
  text-transform: uppercase;
  letter-spacing: .02em;
}
.add-user-row { padding-bottom: 6px; border-bottom: 1px solid #eef0f4; }

.field { display: flex; flex-direction: column; gap: 4px; }
.field input {
  border: 1px solid #d5dbdb;
  border-radius: 6px;
  padding: 8px 10px;
  font-size: 13px;
  width: 100%;
}
.field input:focus { outline: none; border-color: #2f817a; }
.field input.field--error { border-color: #d0453d; }
.readonly-field { background: #f5f7f8; color: #4b5563; }
.field-error { color: #d0453d; font-size: 11px; }

.remove-row-button {
  border: none;
  background: transparent;
  color: #d0453d;
  cursor: pointer;
  font-size: 14px;
  padding: 8px 4px;
}
.remove-row-button:disabled { opacity: .3; cursor: default; }

.add-row-button {
  margin-top: 12px;
  border: 1px dashed #2f817a;
  color: #2f817a;
  background: transparent;
  border-radius: 6px;
  padding: 8px 14px;
  font-size: 12.5px;
  font-weight: 600;
  cursor: pointer;
}
.add-row-button:hover { background: #eef8f4; }

.form-error {
  margin-top: 12px;
  color: #d0453d;
  font-size: 12.5px;
  font-weight: 600;
}
</style>