<template>
  <section class="page-body inner-page">
    <div class="page-heading"><div><p class="eyebrow">ADMINISTRATION / {{ page.eyebrow }}</p><h1>{{ page.title }}</h1><p class="page-description">{{ page.description }}</p></div><button class="primary-button">＋ {{ page.action }}</button></div>
    <div class="toolbar"><div class="search-box">⌕ <input placeholder="Cari data..." /></div><button>≡ Filter　<span>2</span></button><button>Export ↓</button></div>
    <section class="panel data-panel"><div class="panel-heading"><div><h2>Daftar {{ page.title }}</h2><p>Menampilkan data terkini dalam workspace ini</p></div><small>24 data</small></div><div class="table-wrap"><table><thead><tr><th v-for="column in page.columns" :key="column">{{ column }}</th><th></th></tr></thead><tbody><tr v-for="(row, index) in page.rows" :key="index"><td v-for="cell in row" :key="cell">{{ cell }}</td><td>•••</td></tr></tbody></table></div><div class="table-footer">Menampilkan 1-3 dari 24 data <span>←　<b>1</b>　2　3　→</span></div></section>
  </section>
</template>

<script setup>
defineProps({ page: { type: Object, required: true } })
</script>
