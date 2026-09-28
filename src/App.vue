<script setup>
import { computed, ref } from 'vue'
import DashboardView from './components/DashboardView.vue'
import DataMenuView from './components/DataMenuView.vue'
import ManagementAccessView from './components/ManagementAccessView.vue'
import ManagementArchiveView from './components/ManagementArchiveView.vue'
import ManagementFormView from './components/ManagementFormView.vue'
import Usermanagamentpage from './components/Usermanagamentpage.vue'
import MasterDataPerawatanView from './components/MasterDataPerawatanView.vue'
import MenuShell from './components/MenuShell.vue'

const activeMenu = ref('Dashboard')
const activeRole = ref('Super User')
const showUserModal = ref(false)
const showRoleModal = ref(false)
const showDeactivateModal = ref(false)
const showActivateModal = ref(false)
const showDeleteRoleModal = ref(false)
const editingRoleIndex = ref(null)
const roleToDelete = ref(null)
const newUser = ref({ npp: '', name: '' })
const roleForm = ref({ name: '', menus: [], npps: [''] })
const selectedUsers = ref([])
const userRows = ref([
  { npp: '12001', name: 'Dimas Pratama', company: 'PT Rail Indonesia', area: 'Sarana', position: 'Manager', status: 'Active' },
  { npp: '12002', name: 'Nadia Kurnia', company: 'PT Rail Indonesia', area: 'Keuangan', position: 'Specialist', status: 'Active' },
  { npp: '12003', name: 'Rizky Maulana', company: 'PT Rail Indonesia', area: 'Operasi', position: 'Vice President', status: 'Non Active' },
  { npp: '12004', name: 'Sari Wulandari', company: 'PT Rail Indonesia', area: 'Sarana', position: 'Specialist', status: 'Active' },
])
const roleMenus = ['Dashboard', 'Management User', 'Management Role', 'Management Form', 'Management Archive', 'Master Data Perawatan', 'Prognosa', 'RKAP']
const protectedRoleNames = ['Super User', 'Vice manager', 'Manager', 'Specialist']
const roles = ref([
  { name: 'Super User', npps: ['12001'], users: 1, menus: [...roleMenus] },
  { name: 'Vice manager', npps: ['12002'], users: 1, menus: ['Dashboard', 'Management User', 'Management Role', 'Prognosa', 'RKAP'] },
  { name: 'Manager', npps: ['12003'], users: 1, menus: ['Dashboard', 'Management User', 'Management Form', 'Management Archive', 'Prognosa', 'RKAP'] },
  { name: 'Specialist', npps: ['12004'], users: 1, menus: ['Dashboard', 'Management Form', 'Management Archive'] },
])
const planningHierarchy = [
  { label: 'Sarana', items: ['Hari Kerja Balai Yasa', 'Produksi Perawatan Sarana', 'Produksi Sarana', 'Tanggungan Balai Yasa'] },
  { label: 'Prasarana Jalan Jembatan', items: ['Produksi Prasarana Rel Pemerintah', 'Produksi Prasarana Rel KAI', 'Produksi Prasarana Jembatan Pemerintah', 'Produksi Prasarana Jembatan KAI', 'Produksi Asset Prasarana Jalan Jembatan'] },
  { label: 'Prasarana Sintel', items: ['Produksi Prasarana Sintel Pemerintah', 'Produksi Asset Prasarana Sintel'] },
  { label: 'Produksi Sumber Daya Manusia' },
  { label: 'Produksi Penumpang' },
  { label: 'Produksi Barang' },
  { label: 'Produksi Pelayanan' },
  { label: 'Norma Kendali Kecelakaan Kereta Api' },
]
function createPlanningMenu(label, icon) {
  return {
    label,
    icon,
    sections: planningHierarchy.map((section) => ({
      label: section.label,
      value: `${label} / ${section.label}`,
      items: section.items?.map((child) => ({ label: child, value: `${label} / ${section.label} / ${child}` })),
    })),
  }
}
const menuGroups = [
  { label: 'OVERVIEW', items: [{ label: 'Dashboard', icon: 'DB' }] },
  { label: 'ADMINISTRATION', items: [{ label: 'Management User', icon: 'US', badge: '12' }, { label: 'Management Role', icon: 'AC' }, { label: 'Management Form', icon: 'FM' }, { label: 'Management Archive', icon: 'AR' }] },
  { label: 'MASTER DATA', items: [{ label: 'Master Data Perawatan', icon: 'MT' }] },
  { label: 'PLANNING', items: [createPlanningMenu('Prognosa', 'PR'), createPlanningMenu('RKAP', 'RK')] },
]
const pageCopy = {
  'Management Form': { eyebrow: 'FORM BUILDER', title: 'Management Form', description: 'Siapkan template form untuk pengumpulan data anggaran.', action: 'Buat Form Baru', columns: ['Nama form', 'Periode', 'Pengisi', 'Status'], rows: [['Form RKA 2025', 'FY 2025', '28 unit kerja', 'Published'], ['Realisasi Q3', 'Q3 2024', '18 unit kerja', 'Draft'], ['Revisi Prognosa', 'FY 2024', '12 unit kerja', 'Published']] },
  'Management Archive': { eyebrow: 'DOCUMENT ARCHIVE', title: 'Management Archive', description: 'Temukan dokumen dan histori pengajuan yang telah diarsipkan.', action: 'Upload Dokumen', columns: ['Nama dokumen', 'Kategori', 'Periode', 'Diarsipkan'], rows: [['RKAP Konsolidasi 2023', 'RKAP', 'FY 2023', '18 Jan 2024'], ['Laporan Prognosa Q4', 'Prognosa', 'Q4 2023', '12 Jan 2024'], ['Berita Acara RKA', 'Administrasi', 'FY 2023', '04 Jan 2024']] },
  Prognosa: { eyebrow: 'FORECASTING', title: 'Prognosa', description: 'Pantau proyeksi penyerapan dan tren anggaran tahun berjalan.', action: 'Input Prognosa', columns: ['Unit kerja', 'Pagu tahun berjalan', 'Prognosa serapan', 'Varians'], rows: [['Divisi Infrastruktur', 'Rp 1.240.000.000.000', '92,4%', '+2,4%'], ['Divisi Sarana', 'Rp 860.000.000.000', '88,1%', '-1,9%'], ['Divisi Operasi', 'Rp 690.000.000.000', '95,8%', '+5,8%']] },
  RKAP: { eyebrow: 'ANNUAL BUDGET', title: 'RKAP', description: 'Review dan monitor rencana kerja dan anggaran perusahaan.', action: 'Buat RKAP', columns: ['Program kerja', 'Pemilik', 'Nilai usulan', 'Progress'], rows: [['Modernisasi Jalur Utara', 'Infrastructure', 'Rp 2,4 T', '78%'], ['Revitalisasi Stasiun', 'Property', 'Rp 840 M', '54%'], ['Digitalisasi Operasi', 'Technology', 'Rp 312 M', '91%']] },
}
function pageForMenu(menu) {
  if (pageCopy[menu]) return pageCopy[menu]
  const title = menu.split(' / ').pop()
  return { eyebrow: 'PLANNING', title, description: `Kelola data ${title} untuk workspace anggaran.`, action: 'Tambah data', columns: ['Nama item', 'Area bisnis', 'Periode', 'Status'], rows: [] }
}
const allUsersSelected = computed(() => userRows.value.length > 0 && selectedUsers.value.length === userRows.value.length)
const selectedActiveUsers = computed(() => userRows.value.filter((user) => selectedUsers.value.includes(user.npp) && user.status === 'Active'))
const selectedInactiveUsers = computed(() => userRows.value.filter((user) => selectedUsers.value.includes(user.npp) && user.status === 'Non Active'))

function openUserModal() { newUser.value = { npp: '', name: '' }; showUserModal.value = true }
function openStatusModal(user) {
  selectedUsers.value = [user.npp]
  if (user.status === 'Active') showDeactivateModal.value = true
  else showActivateModal.value = true
}
function saveUser() {
  if (!/^\d{5}$/.test(newUser.value.npp) || !newUser.value.name.trim()) return
  if (!userRows.value.some((user) => user.npp === newUser.value.npp)) userRows.value.unshift({ ...newUser.value, company: 'PT Rail Indonesia', area: 'Sarana', position: 'Specialist', status: 'Active' })
  showUserModal.value = false
}
function openRoleModal(role = null, index = null) {
  editingRoleIndex.value = index
  roleForm.value = role
    ? { name: role.name, menus: [...role.menus], npps: [...(role.npps || [])] }
    : { name: '', menus: [], npps: [''] }
  showRoleModal.value = true
}
function addRoleMember() { roleForm.value.npps.push('') }
function removeRoleMember(index) {
  if (roleForm.value.npps.length === 1) return
  roleForm.value.npps.splice(index, 1)
}
function roleMemberName(npp) {
  return userRows.value.find((user) => user.npp === npp)?.name || ''
}
function saveRole() {
  const npps = roleForm.value.npps.map((npp) => npp.trim())
  if (!roleForm.value.name.trim() || !roleForm.value.menus.length || !npps.length || npps.some((npp) => !roleMemberName(npp))) return
  if (new Set(npps).size !== npps.length) return
  const role = { name: roleForm.value.name.trim(), npps, users: npps.length, menus: [...roleForm.value.menus] }
  if (editingRoleIndex.value === null) roles.value.push(role); else roles.value[editingRoleIndex.value] = role
  showRoleModal.value = false
}
function requestDeleteRole(role, index) {
  if (protectedRoleNames.includes(role.name)) return
  roleToDelete.value = { name: role.name, index }
  showDeleteRoleModal.value = true
}
function confirmDeleteRole() {
  if (roleToDelete.value) roles.value.splice(roleToDelete.value.index, 1)
  roleToDelete.value = null
  showDeleteRoleModal.value = false
}
function confirmStatus(type) {
  const targetStatus = type === 'activate' ? 'Non Active' : 'Active'
  const nextStatus = type === 'activate' ? 'Active' : 'Non Active'
  userRows.value.forEach((user) => {
    if (selectedUsers.value.includes(user.npp) && user.status === targetStatus) user.status = nextStatus
  })
  selectedUsers.value = []
  showDeactivateModal.value = false
  showActivateModal.value = false
}
</script>

<template>
  <MenuShell :active-menu="activeMenu" :active-role="activeRole" :menu-groups="menuGroups" @change-menu="activeMenu = $event" @update:active-role="activeRole = $event">
    <DashboardView v-if="activeMenu === 'Dashboard'" />
    <Usermanagamentpage v-else-if="activeMenu === 'Management User'" />
    <ManagementAccessView v-else-if="activeMenu === 'Management Role'" :roles="roles" :role-menus="roleMenus" :protected-roles="protectedRoleNames" @add-role="openRoleModal()" @edit-role="openRoleModal" @delete-role="requestDeleteRole" />
    <ManagementFormView v-else-if="activeMenu === 'Management Form'" />
    <ManagementArchiveView v-else-if="activeMenu === 'Management Archive'" />
    <MasterDataPerawatanView v-else-if="activeMenu === 'Master Data Perawatan'" />
    <DataMenuView v-else :page="pageForMenu(activeMenu)" />

    <div v-if="showUserModal" class="modal-backdrop" @click.self="showUserModal = false"><section class="user-modal" role="dialog" aria-modal="true"><div class="modal-header"><div><p class="eyebrow">USER DIRECTORY / NEW ACCESS</p><h2>Tambah user baru</h2><p>Masukkan NPP dan nama karyawan untuk membuka akses Rail Budgeting.</p></div><button class="modal-close" @click="showUserModal = false">×</button></div><label class="modal-label">NPP karyawan</label><input v-model="newUser.npp" class="role-name-input" placeholder="Contoh: 12005" /><label class="modal-label menu-label">Nama lengkap</label><input v-model="newUser.name" class="role-name-input" placeholder="Nama lengkap" /><div class="modal-actions"><button class="cancel-button" @click="showUserModal = false">Batal</button><button class="primary-button" :disabled="!/^\d{5}$/.test(newUser.npp) || !newUser.name" @click="saveUser">Simpan user</button></div></section></div>
    <div v-if="showRoleModal" class="modal-backdrop" @click.self="showRoleModal = false"><section class="user-modal role-modal" role="dialog" aria-modal="true"><div class="modal-header"><div><p class="eyebrow">ROLE MANAGEMENT / ACCESS MENU</p><h2>{{ editingRoleIndex === null ? 'Tambah role baru' : 'Edit role' }}</h2></div><button class="modal-close" @click="showRoleModal = false">×</button></div><label class="modal-label">Nama role</label><input v-model="roleForm.name" class="role-name-input" placeholder="Contoh: Budget Reviewer" /><label class="modal-label menu-label">NIPP user</label><div class="role-member-list"><div v-for="(npp, index) in roleForm.npps" :key="index" class="role-member-row"><div><input v-model="roleForm.npps[index]" class="role-name-input" inputmode="numeric" list="role-user-directory" placeholder="Contoh: 12001" /><small :class="roleMemberName(npp) ? 'field-valid' : 'field-error'">{{ roleMemberName(npp) || 'NIPP tidak ditemukan' }}</small></div><button type="button" class="remove-row-button" :disabled="roleForm.npps.length === 1" @click="removeRoleMember(index)">×</button></div></div><datalist id="role-user-directory"><option v-for="user in userRows" :key="user.npp" :value="user.npp">{{ user.name }}</option></datalist><button type="button" class="add-row-button" @click="addRoleMember">+ Tambah NIPP</button><label class="modal-label menu-label">Access menu</label><div class="menu-choice-grid"><button v-for="menu in roleMenus" :key="menu" type="button" class="menu-choice" :class="{ selected: roleForm.menus.includes(menu) }" @click="roleForm.menus.includes(menu) ? roleForm.menus = roleForm.menus.filter((item) => item !== menu) : roleForm.menus.push(menu)"><span>{{ menu.slice(0, 2).toUpperCase() }}</span><strong>{{ menu }}</strong><i>{{ roleForm.menus.includes(menu) ? '✓' : '+' }}</i></button></div><div class="modal-actions"><button class="cancel-button" @click="showRoleModal = false">Batal</button><button class="primary-button" :disabled="!roleForm.name.trim() || !roleForm.menus.length || !roleForm.npps.length || roleForm.npps.some((npp) => !roleMemberName(npp))" @click="saveRole">{{ editingRoleIndex === null ? 'Tambahkan role' : 'Simpan perubahan' }}</button></div></section></div>
    <div v-if="showDeleteRoleModal" class="modal-backdrop" @click.self="showDeleteRoleModal = false"><section class="user-modal confirm-modal" role="dialog" aria-modal="true"><h2>Hapus role?</h2><p>Role <strong>{{ roleToDelete?.name }}</strong> akan dihapus dan tidak dapat digunakan lagi.</p><div class="modal-actions"><button class="cancel-button" @click="showDeleteRoleModal = false">Batal</button><button class="confirm-danger-button" @click="confirmDeleteRole">Ya, hapus role</button></div></section></div>
  </MenuShell>
</template>
