<template>
  <div v-if="show && user" class="modal-backdrop" @click.self="$emit('close')">
    <section class="user-modal detail-modal" role="dialog" aria-modal="true">
      <h2>Detail user</h2>

      <dl class="detail-list">
        <div class="detail-row">
          <dt>NIPP</dt>
          <dd>{{ user.npp }}</dd>
        </div>
        <div class="detail-row">
          <dt>Nama</dt>
          <dd>{{ user.name }}</dd>
        </div>
        <div class="detail-row">
          <dt>Perusahaan</dt>
          <dd>{{ user.company }}</dd>
        </div>
        <div class="detail-row">
          <dt>Sub Area</dt>
          <dd>{{ user.area }}</dd>
        </div>
        <div class="detail-row">
          <dt>Jabatan</dt>
          <dd>{{ user.position }}</dd>
        </div>
        <div class="detail-row" v-if="user.email">
          <dt>Email</dt>
          <dd>{{ user.email }}</dd>
        </div>
        <div class="detail-row">
          <dt>Status</dt>
          <dd>
            <span
              class="status-label"
              :class="user.status === 'Active' ? 'active-status' : 'inactive-status'"
            >
              {{ user.status }}
            </span>
          </dd>
        </div>
      </dl>

      <div class="modal-actions">
        <button class="cancel-button" @click="$emit('close')">Tutup</button>
        <button class="deactivate-button" @click="$emit('change-status', user)">
          {{ user.status === 'Active' ? 'Nonaktifkan user ini' : 'Aktifkan user ini' }}
        </button>
      </div>
    </section>
  </div>
</template>

<script setup>
defineProps({
  show: Boolean,
  user: { type: Object, default: null },
})
defineEmits(['close', 'change-status'])
</script>

<style scoped>
.detail-modal { width: min(420px, 92vw); }
.detail-list { margin: 12px 0 18px; }
.detail-row {
  display: grid;
  grid-template-columns: 110px 1fr;
  padding: 6px 0;
  border-bottom: 1px solid #eef0f4;
  font-size: 13px;
}
.detail-row dt { color: #8a999d; }
.detail-row dd { margin: 0; color: #1f2430; font-weight: 600; }
</style>