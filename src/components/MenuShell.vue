<template>
  <div class="app-shell">
    <aside class="sidebar">
      <div class="brand-lockup"><div class="brand-mark">RB</div><div><strong>Rail<span>Budgeting</span></strong><small>Planning workspace</small></div></div>
      <div class="workspace-switcher"><div class="workspace-avatar">R</div><div class="workspace-text"><small>WORKSPACE</small><strong>PT Rail Indonesia</strong></div><span>⌄</span></div>
      <nav class="main-nav">
        <div v-for="group in menuGroups" :key="group.label" class="nav-group">
          <p class="nav-label">{{ group.label }}</p>
          <template v-for="item in group.items" :key="item.label">
            <button class="nav-item" :class="{ active: activeMenu === (item.value || item.label), expandable: item.sections }" @click="item.sections ? toggleMenu(item.label) : $emit('change-menu', item.value || item.label)">
              <span class="nav-icon">{{ item.icon }}</span><span>{{ item.label }}</span><b v-if="item.badge">{{ item.badge }}</b><i v-if="item.sections" class="nav-chevron" :class="{ open: isOpen(item.label) }">⌄</i>
            </button>
            <div v-if="item.sections && isOpen(item.label)" class="nav-submenu">
              <div v-for="section in item.sections" :key="section.label" class="nav-subsection">
                <button v-if="!section.items" class="nav-child nav-section-link" :class="{ active: activeMenu === section.value }" @click="$emit('change-menu', section.value)">{{ section.label }}</button>
                <template v-else>
                  <p class="nav-subsection-label">{{ section.label }}</p>
                  <button v-for="child in section.items" :key="child.value" class="nav-child" :class="{ active: activeMenu === child.value }" @click="$emit('change-menu', child.value)">{{ child.label }}</button>
                </template>
              </div>
            </div>
          </template>
        </div>
      </nav>
      <div class="sidebar-bottom"><div class="help-card"><span>?</span><div><strong>Butuh bantuan?</strong><small>Hubungi tim support</small></div></div><div class="user-profile"><div class="profile-avatar">AD</div><div><strong>Adi Nugraha</strong><small>{{ activeRole }}</small></div><button>•••</button></div></div>
    </aside>
    <main class="main-content"><header class="topbar"><div class="breadcrumbs"><span>Workspace</span><b>/</b><strong>{{ activeMenu }}</strong></div><div class="topbar-actions"><label>View as <select :value="activeRole" @change="$emit('update:activeRole', $event.target.value)"><option>Super User</option><option>Vice manager</option><option>Manager</option><option>Specialist</option></select></label><button>♧</button><button>⚙</button></div></header><slot /></main>
  </div>
</template>

<script setup>
import { ref } from 'vue'

defineProps({ activeMenu: { type: String, required: true }, activeRole: { type: String, required: true }, menuGroups: { type: Array, required: true } })
defineEmits(['change-menu', 'update:activeRole'])

const openMenus = ref({})
function isOpen(label) { return Boolean(openMenus.value[label]) }
function toggleMenu(label) { openMenus.value[label] = !openMenus.value[label] }
</script>

<style scoped>
.nav-item.expandable { position: relative; }
.nav-chevron { margin-left: auto; color: #78969a; font-size: 13px; font-style: normal; transform: rotate(0deg); transition: transform .15s ease; }
.nav-chevron.open { transform: rotate(180deg); }
.nav-submenu { margin: 2px 0 8px 18px; border-left: 1px solid #31545b; padding-left: 10px; }
.nav-subsection { margin: 8px 0 12px; }
.nav-subsection-label { margin: 0 0 4px; color: #d0dddd; font-size: 10px; font-weight: 700; }
.nav-child { display: block; width: 100%; border: 0; border-radius: 4px; padding: 6px 7px; background: transparent; color: #8eabad; font-size: 10px; text-align: left; }
.nav-child:hover, .nav-child.active { background: #214954; color: #eef7f4; }
.nav-section-link { color: #d0dddd; font-weight: 700; }
</style>
