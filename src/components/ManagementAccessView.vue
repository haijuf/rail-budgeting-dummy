<template>
  <section class="page-body inner-page role-page">
    <div class="page-heading">
      <div>
        <p class="eyebrow">ADMINISTRATION / ROLE MANAGEMENT</p>
        <h1>Role Management</h1>
        <p class="page-description">Buat role, atur access, dan tentukan user yang dapat menggunakannya.</p>
      </div>
      <button class="primary-button" @click="$emit('add-role')">＋ Tambah role</button>
    </div>

    <div class="user-summary">
      <div><strong>{{ roles.length }}</strong><span>Total role</span></div>
      <div><strong>{{ roles.reduce((total, role) => total + role.users, 0) }}</strong><span>User dengan role</span></div>
      <div><strong>{{ roleMenus.length }}</strong><span>Menu tersedia</span></div>
    </div>

    <section class="panel data-panel">
      <div class="panel-heading">
        <div><h2>Daftar role</h2><p>Empat role utama tidak dapat dihapus.</p></div>
        <small>{{ roles.length }} role</small>
      </div>
      <div class="role-list">
        <article v-for="(role, index) in roles" :key="role.name" class="role-row">
          <div class="role-identity">
            <span class="role-avatar">{{ role.name.slice(0, 2).toUpperCase() }}</span>
            <div><strong>{{ role.name }}</strong><small>{{ role.users }} user terdaftar</small></div>
          </div>
          <div class="role-menu-list"><span v-for="menu in role.menus" :key="menu">{{ menu }}</span></div>
          <div class="role-actions">
            <button @click="$emit('edit-role', role, index)">Edit</button>
            <button class="delete-role" :disabled="protectedRoles.includes(role.name)" :title="protectedRoles.includes(role.name) ? 'Role utama tidak dapat dihapus' : 'Hapus role'" @click="$emit('delete-role', role, index)">
              {{ protectedRoles.includes(role.name) ? 'Role utama' : 'Hapus' }}
            </button>
          </div>
        </article>
      </div>
    </section>
  </section>
</template>

<script setup>
defineProps({ roles: { type: Array, required: true }, roleMenus: { type: Array, required: true }, protectedRoles: { type: Array, default: () => [] } })
defineEmits(['add-role', 'edit-role', 'delete-role'])
</script>
