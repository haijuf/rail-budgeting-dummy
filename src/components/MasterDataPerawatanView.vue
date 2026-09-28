<template>
  <DataMenuView :page="page" />
</template>

<script setup>
import DataMenuView from './DataMenuView.vue'

const page = {
  eyebrow: 'MASTER DATA / PERAWATAN',
  title: 'Master Data Perawatan',
  description: 'Kelola data referensi perawatan aset dan sarana operasional perusahaan.',
  action: 'Tambah Data Perawatan',
  columns: ['Kode perawatan', 'Nama perawatan', 'Kategori aset', 'Satuan', 'Status'],
  rows: [
    ['MNT-001', 'Perawatan berkala lokomotif', 'Lokomotif', 'Kegiatan', 'Active'],
    ['MNT-002', 'Pemeriksaan jalur rel', 'Infrastruktur', 'Kilometer', 'Active'],
    ['MNT-003', 'Perawatan fasilitas stasiun', 'Stasiun', 'Lokasi', 'Draft'],
  ],
}
</script>
