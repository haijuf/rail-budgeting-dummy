<template>
  <section class="page-body inner-page user-page">
    <div class="page-heading">
      <div>
        <p class="eyebrow">ADMINISTRATION / USER DIRECTORY</p>
        <h1>Management User</h1>
        <p class="page-description">Daftarkan dan kelola akses user Rail Budgeting berdasarkan NPP karyawan.</p>
      </div>
      <button class="primary-button" @click="$emit('add-user')">＋ Tambah user</button>
    </div>

    <!-- Total data user berdasarkan status -->
    <div class="user-summary">
      <div>
        <strong>{{ stats.total }}</strong>
        <span>Total user terdaftar</span>
      </div>
      <div>
        <strong>{{ stats.active }}</strong>
        <span>Active</span>
      </div>
      <div>
        <strong>{{ stats.nonActive }}</strong>
        <span>Non Active</span>
      </div>
    </div>

    <div class="toolbar">
      <div class="search-box">
        ⌕
        <input
          :value="searchQuery"
          placeholder="Cari NPP atau nama..."
          @input="$emit('update:search-query', $event.target.value)"
        />
      </div>
      <span v-if="selectedCount > 0" class="selection-count">{{ selectedCount }} user dipilih</span>
      <button :disabled="!activeSelected" class="danger-button" @click="$emit('deactivate')">
        Non aktifkan terpilih
      </button>
      <button :disabled="!inactiveSelected" class="toolbar-activate" @click="$emit('activate')">
        Aktifkan terpilih
      </button>
      <button @click="exportCsv">Export ↓</button>
    </div>

    <section class="panel data-panel">
      <div class="panel-heading">
        <div>
          <h2>Daftar user Rail Budgeting</h2>
          <p>User baru otomatis mendapatkan status Active.</p>
        </div>
        <small>{{ stats.total }} user</small>
      </div>

      <div class="table-wrap">
        <table class="user-table">
          <thead>
            <tr>
              <th><input type="checkbox" :checked="allSelected" @change="$emit('toggle-all')" /></th>
              <th>NPP</th>
              <th>Nama</th>
              <th>Perusahaan</th>
              <th>Sub Area</th>
              <th>Jabatan</th>
              <th>Status</th>
              <th>Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="users.length === 0">
              <td colspan="8" class="empty-row">Tidak ada user yang cocok dengan pencarian.</td>
            </tr>
            <tr v-for="user in users" :key="user.npp" @click="$emit('select-user', user)">
              <td>
                <input
                  v-model="selectedUsersProxy"
                  type="checkbox"
                  :value="user.npp"
                  @click.stop
                />
              </td>
              <td class="npp-cell">{{ user.npp }}</td>
              <td class="name-cell">{{ user.name }}</td>
              <td>{{ user.company }}</td>
              <td>{{ user.area }}</td>
              <td>{{ user.position }}</td>
              <td>
                <span
                  class="status-label"
                  :class="user.status === 'Active' ? 'active-status' : 'inactive-status'"
                >
                  {{ user.status }}
                </span>
              </td>
              <td class="action-cell">
                <button class="detail-button" @click.stop="$emit('view-user', user)">Detail</button>
                <button class="deactivate-button" @click.stop="$emit('change-status', user)">
                  {{ user.status === 'Active' ? 'Nonaktifkan' : 'Aktifkan' }}
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div class="table-footer">
        Menampilkan {{ users.length }} dari {{ totalFiltered }} user
        <span class="pagination">
          <button :disabled="page <= 1" @click="$emit('prev-page')">←</button>
          <b>{{ page }}</b> / {{ totalPages }}
          <button :disabled="page >= totalPages" @click="$emit('next-page')">→</button>
        </span>
      </div>
    </section>
  </section>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  users: { type: Array, required: true }, // current page rows
  selectedUsers: { type: Array, required: true },
  allSelected: Boolean,
  activeSelected: Boolean,
  inactiveSelected: Boolean,
  selectedCount: { type: Number, default: 0 },
  stats: {
    type: Object,
    default: () => ({ total: 0, active: 0, nonActive: 0 }),
  },
  searchQuery: { type: String, default: '' },
  page: { type: Number, default: 1 },
  totalPages: { type: Number, default: 1 },
  totalFiltered: { type: Number, default: 0 },
  exportUsers: { type: Array, default: () => [] },
})

const emit = defineEmits([
  'update:selectedUsers',
  'update:search-query',
  'add-user',
  'deactivate',
  'activate',
  'toggle-all',
  'select-user',
  'view-user',
  'change-status',
  'prev-page',
  'next-page',
])

const selectedUsersProxy = computed({
  get: () => props.selectedUsers,
  set: (value) => emit('update:selectedUsers', value),
})

function exportCsv() {
  const source = props.exportUsers.length ? props.exportUsers : props.users
  if (source.length === 0) return

  const header = ['NPP', 'Nama', 'Perusahaan', 'Sub Area', 'Jabatan', 'Status']
  const rows = source.map((u) => [u.npp, u.name, u.company, u.area, u.position, u.status])
  const csv = [header, ...rows]
    .map((row) => row.map((cell) => `"${String(cell).replace(/"/g, '""')}"`).join(','))
    .join('\n')

  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')
  link.href = url
  link.download = 'daftar-user-rail-budgeting.csv'
  link.click()
  URL.revokeObjectURL(url)
}
</script>

<style scoped>
/* Minimal fallback styling for pieces not covered by the host app's
   existing global stylesheet (this file otherwise relies on that CSS,
   e.g. .primary-button, .panel, .user-table, .status-label, etc.) */
.action-cell { display: flex; gap: 6px; white-space: nowrap; }
.detail-button {
  border: 1px solid #d5dbdb;
  background: #fff;
  border-radius: 6px;
  padding: 4px 10px;
  font-size: 12px;
  cursor: pointer;
}
.detail-button:hover { background: #f2f5f5; }
.selection-count { font-size: 12.5px; color: #2f817a; font-weight: 700; }
.empty-row { text-align: center; color: #8a999d; padding: 20px !important; }
.pagination { display: inline-flex; align-items: center; gap: 6px; margin-left: 12px; }
.pagination button {
  border: 1px solid #d5dbdb;
  background: #fff;
  border-radius: 6px;
  padding: 2px 8px;
  cursor: pointer;
}
.pagination button:disabled { opacity: .4; cursor: default; }
</style>