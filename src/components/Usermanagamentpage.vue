<template>
  <div>
    <transition name="fade">
      <div v-if="notice" class="notice" :class="notice.type">{{ notice.msg }}</div>
    </transition>

    <ManagementUserView
      v-model:selected-users="selectedUsers"
      :users="pagedUsers"
      :stats="stats"
      :all-selected="allSelectedOnPage"
      :active-selected="activeSelected"
      :inactive-selected="inactiveSelected"
      :selected-count="selectedUsers.length"
      :search-query="searchQuery"
      :page="page"
      :total-pages="totalPages"
      :total-filtered="filteredUsers.length"
      :export-users="filteredUsers"
      @update:search-query="onSearch"
      @add-user="showAddModal = true"
      @toggle-all="toggleAllOnPage"
      @select-user="openDetail"
      @view-user="openDetail"
      @change-status="requestSingleStatusChange"
      @deactivate="requestBulkDeactivate"
      @activate="requestBulkActivate"
      @prev-page="page = Math.max(1, page - 1)"
      @next-page="page = Math.min(totalPages, page + 1)"
    />

    <AddUserModal
      :show="showAddModal"
      :existing-npps="existingNpps"
      :employee-directory="employeeDirectory"
      @close="showAddModal = false"
      @submit="handleAddUsers"
    />

    <ConfirmStatusModal
      :show="showConfirm"
      :type="confirmType"
      :count="pendingNpps.length"
      @close="showConfirm = false"
      @confirm="applyStatusChange"
    />

    <UserDetailModal
      :show="showDetail"
      :user="detailUser"
      @close="showDetail = false"
      @change-status="requestSingleStatusChange"
    />
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'
import ManagementUserView from './ManagementUserView.vue'
import AddUserModal from './Addusermodal.vue'
import ConfirmStatusModal from './Confirmstatusmodal.vue'
import UserDetailModal from './Userdetailmodal.vue'

// ---- master data pegawai (dipakai untuk auto-fill NIPP di modal tambah user) ----
// Di aplikasi nyata ini biasanya berasal dari API HR/master data pegawai.
const employeeDirectory = {
  12345678: { name: 'Yoga Pratama', company: 'PT KAI', area: 'DAOP 1 JAKARTA', position: 'Staff Anggaran' },
  23456781: { name: 'Siti Amalia', company: 'PT KAI', area: 'DAOP 2 BANDUNG', position: 'Supervisor Anggaran' },
  34567812: { name: 'Budi Santoso', company: 'PT KAI', area: 'DAOP 6 YOGYAKARTA', position: 'Manager Anggaran' },
  45678123: { name: 'Rina Wulandari', company: 'PT KAI', area: 'DIVRE 1 SUMATERA UTARA', position: 'Staff Sarana' },
  56781234: { name: 'Agus Setiawan', company: 'PT KAI', area: 'DAOP 8 SURABAYA', position: 'Staff Prasarana' },
  67812345: { name: 'Dewi Lestari', company: 'PT KAI', area: 'Balai Yasa Yogyakarta', position: 'Staff Balai Yasa' },
  78123456: { name: 'Fajar Nugroho', company: 'PT KAI', area: 'DAOP 4 SEMARANG', position: 'Manager Sarana' },
  81234567: { name: 'Nur Hidayah', company: 'PT KAI', area: 'DAOP 3 CIREBON', position: 'Staff Anggaran' },
  91234568: { name: 'Hendra Wijaya', company: 'PT KAI', area: 'DAOP 9 JEMBER', position: 'Staff Prasarana' },
  10293847: { name: 'Maya Puspita', company: 'PT KAI', area: 'DAOP 7 MADIUN', position: 'Supervisor Sarana' },
  99887766: { name: 'Andi Saputra', company: 'PT KAI', area: 'DAOP 5 PURWOKERTO', position: 'Staff Anggaran' },
  88776655: { name: 'Lina Marlina', company: 'PT KAI', area: 'DIVRE 1 SUMATERA UTARA', position: 'Staff Sarana' },
  77665544: { name: 'Rudi Hartono', company: 'PT KAI', area: 'Balai Yasa Manggarai', position: 'Staff Balai Yasa' },
}
// Catatan: key numerik di atas otomatis disimpan sebagai string oleh JS,
// jadi employeeDirectory['12345678'] tetap valid saat dicocokkan dengan input form.

// ---- data user (ganti dengan hasil fetch API pada implementasi nyata) ----
const users = ref(
  Object.entries(employeeDirectory)
    .slice(0, 10)
    .map(([npp, data], index) => ({
      npp: String(npp),
      name: data.name,
      company: data.company,
      area: data.area,
      position: data.position,
      status: index % 3 === 0 ? 'Non Active' : 'Active',
    }))
)

const existingNpps = computed(() => users.value.map((u) => u.npp))

// ---- statistik total data user berdasarkan status ----
const stats = computed(() => ({
  total: users.value.length,
  active: users.value.filter((u) => u.status === 'Active').length,
  nonActive: users.value.filter((u) => u.status === 'Non Active').length,
}))

// ---- pencarian ----
const searchQuery = ref('')
function onSearch(value) {
  searchQuery.value = value
  page.value = 1
}
const filteredUsers = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  if (!q) return users.value
  return users.value.filter(
    (u) => u.npp.toLowerCase().includes(q) || u.name.toLowerCase().includes(q)
  )
})

// ---- paginasi ----
const page = ref(1)
const pageSize = 8
const totalPages = computed(() => Math.max(1, Math.ceil(filteredUsers.value.length / pageSize)))
const pagedUsers = computed(() => {
  const start = (page.value - 1) * pageSize
  return filteredUsers.value.slice(start, start + pageSize)
})

// ---- seleksi multi (checkbox) ----
const selectedUsers = ref([])
const allSelectedOnPage = computed(
  () => pagedUsers.value.length > 0 && pagedUsers.value.every((u) => selectedUsers.value.includes(u.npp))
)
function toggleAllOnPage() {
  const pageNpps = pagedUsers.value.map((u) => u.npp)
  selectedUsers.value = allSelectedOnPage.value
    ? selectedUsers.value.filter((npp) => !pageNpps.includes(npp))
    : Array.from(new Set([...selectedUsers.value, ...pageNpps]))
}
const activeSelected = computed(() =>
  selectedUsers.value.some((npp) => users.value.find((u) => u.npp === npp)?.status === 'Active')
)
const inactiveSelected = computed(() =>
  selectedUsers.value.some((npp) => users.value.find((u) => u.npp === npp)?.status === 'Non Active')
)

// ---- detail modal ----
const showDetail = ref(false)
const detailUser = ref(null)
function openDetail(user) {
  detailUser.value = user
  showDetail.value = true
}

// ---- konfirmasi aktif/nonaktif (satuan & multi) ----
const showConfirm = ref(false)
const confirmType = ref('deactivate') // 'activate' | 'deactivate'
const pendingNpps = ref([])

function requestSingleStatusChange(user) {
  confirmType.value = user.status === 'Active' ? 'deactivate' : 'activate'
  pendingNpps.value = [user.npp]
  showDetail.value = false
  showConfirm.value = true
}
function requestBulkDeactivate() {
  const targets = selectedUsers.value.filter(
    (npp) => users.value.find((u) => u.npp === npp)?.status === 'Active'
  )
  if (targets.length === 0) return
  confirmType.value = 'deactivate'
  pendingNpps.value = targets
  showConfirm.value = true
}
function requestBulkActivate() {
  const targets = selectedUsers.value.filter(
    (npp) => users.value.find((u) => u.npp === npp)?.status === 'Non Active'
  )
  if (targets.length === 0) return
  confirmType.value = 'activate'
  pendingNpps.value = targets
  showConfirm.value = true
}
function applyStatusChange() {
  const newStatus = confirmType.value === 'activate' ? 'Active' : 'Non Active'
  users.value = users.value.map((u) =>
    pendingNpps.value.includes(u.npp) ? { ...u, status: newStatus } : u
  )
  selectedUsers.value = selectedUsers.value.filter((npp) => !pendingNpps.value.includes(npp))
  showConfirm.value = false
  showNotice(
    `${pendingNpps.value.length} user berhasil ${newStatus === 'Active' ? 'diaktifkan' : 'dinonaktifkan'}.`,
    'success'
  )
  pendingNpps.value = []
}

// ---- tambah user (multi, dengan auto-fill dari master data) ----
const showAddModal = ref(false)
function handleAddUsers(newUsers) {
  users.value = [...users.value, ...newUsers]
  showAddModal.value = false
  showNotice(`${newUsers.length} user berhasil ditambahkan.`, 'success')
}

// ---- notifikasi ----
const notice = ref(null)
let noticeTimer = null
function showNotice(msg, type = 'success') {
  notice.value = { msg, type }
  clearTimeout(noticeTimer)
  noticeTimer = setTimeout(() => (notice.value = null), 3000)
}
</script>

<style scoped>
.notice {
  margin: 12px 24px 0;
  padding: 10px 16px;
  border-radius: 8px;
  font-size: 12.5px;
  font-weight: 600;
}
.notice.success { background: #e7f7ee; color: #157347; }
.notice.error { background: #fbeceb; color: #b3261e; }
.fade-enter-active, .fade-leave-active { transition: opacity .2s ease; }
.fade-enter-from, .fade-leave-to { opacity: 0; }
</style>