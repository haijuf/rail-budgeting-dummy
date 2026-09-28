<template>
  <div v-if="show" class="modal-backdrop" @click.self="$emit('close')">
    <section class="user-modal confirm-modal" role="dialog" aria-modal="true">
      <div class="confirm-icon" :class="{ 'activate-icon': type === 'activate' }">
        {{ type === 'activate' ? '✓' : '!' }}
      </div>
      <h2>{{ type === 'activate' ? 'Aktifkan kembali user?' : 'Nonaktifkan user?' }}</h2>
      <p>
        Anda akan {{ type === 'activate' ? 'mengaktifkan kembali' : 'menonaktifkan' }}
        <strong>{{ count }} user</strong> yang dipilih.
      </p>
      <div class="modal-actions">
        <button class="cancel-button" @click="$emit('close')">Batal</button>
        <button
          :class="type === 'activate' ? 'confirm-activate-button' : 'confirm-danger-button'"
          @click="$emit('confirm')"
        >
          Ya, {{ type === 'activate' ? 'aktifkan' : 'nonaktifkan' }}
        </button>
      </div>
    </section>
  </div>
</template>

<script setup>
defineProps({
  show: Boolean,
  type: { type: String, default: 'deactivate' },
  count: { type: Number, default: 0 },
})
defineEmits(['close', 'confirm'])
</script>
