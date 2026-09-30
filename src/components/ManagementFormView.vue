<template>
  <section class="fm">
    <!-- ============ Header ============ -->
    <header class="fm-head">
      <div>
        <div class="fm-crumb">Administration / Management form</div>
        <h1>Management Form</h1>
        <p class="fm-desc">
          Atur layout formulir, publish RKAP per kategori, lalu pantau approval RKAP dan Prognosa dari satu tempat.
        </p>
      </div>
      <div class="fm-year">
        <span>Periode RKAP</span>
        <strong>{{ targetYear }}</strong>
      </div>
    </header>

    <!-- Alur kerja -->
    <ol class="fm-flow" aria-label="Alur kerja">
      <li><b>1</b><span>Pilih formulir</span></li>
      <li><b>2</b><span>Atur layout <small>(bila perlu)</small></span></li>
      <li><b>3</b><span>Publish RKAP per kategori</span></li>
      <li><b>4</b><span>Pantau approval &amp; atur Prognosa</span></li>
    </ol>

    <!-- Ringkasan -->
    <div class="fm-overview">
      <div class="fm-ov">
        <strong>{{ forms.length }}</strong>
        <span>Formulir tetap</span>
      </div>
      <div class="fm-ov">
        <strong>{{ overview.published }}<small>/{{ overview.total }}</small></strong>
        <span>Kategori RKAP sudah dipublish</span>
      </div>
      <div class="fm-ov fm-ov--warn">
        <strong>{{ overview.need }}</strong>
        <span>Menunggu approval (RKAP &amp; Prognosa)</span>
      </div>
      <div class="fm-ov">
        <strong>{{ overview.progOpen }}<small>/{{ overview.total }}</small></strong>
        <span>Kategori Prognosa sedang open</span>
      </div>
    </div>

    <!-- ============ Daftar formulir ============ -->
    <section class="fm-panel">
      <div class="fm-panel__head">
        <div>
          <h2>Daftar formulir</h2>
          <p>16 formulir bersifat tetap dan tidak dapat ditambah. Klik satu baris untuk mengelola publish, prognosa, dan layout-nya.</p>
        </div>
        <span class="fm-count">{{ formRows.length }} dari {{ forms.length }} formulir</span>
      </div>

      <div class="fm-toolbar">
        <label class="fm-search">
          <svg width="15" height="15" viewBox="0 0 24 24" fill="none" aria-hidden="true">
            <circle cx="11" cy="11" r="6.5" stroke="currentColor" stroke-width="1.8" />
            <path d="M16 16l4.5 4.5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" />
          </svg>
          <input v-model="searchQuery" placeholder="Cari nama formulir" />
        </label>

        <select v-model="direktoratFilter" class="fm-select" aria-label="Filter direktorat">
          <option value="semua">Semua direktorat</option>
          <option v-for="d in direktoratOptions" :key="d" :value="d">{{ d }}</option>
        </select>

        <div class="fm-chips">
          <button
            v-for="c in listChips"
            :key="c.key"
            class="fm-chip"
            :class="{ 'is-active': listFilter === c.key }"
            @click="listFilter = c.key"
          >
            {{ c.label }}
          </button>
        </div>
      </div>

      <div class="fm-table-scroll">
        <table class="fm-table fm-table--list">
          <thead>
            <tr>
              <th>Nama formulir</th>
              <th>Direktorat</th>
              <th>RKAP dipublish</th>
              <th>Prognosa open</th>
              <th>Diperbarui</th>
              <th class="fm-right">Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="formRows.length === 0">
              <td colspan="7" class="fm-empty">Tidak ada formulir yang cocok dengan filter.</td>
            </tr>
            <tr v-for="row in formRows" :key="row.form.id" class="fm-row" @click="openForm(row.form)">
              <td>
                <div class="fm-name">{{ row.form.name }}</div>
                <div class="fm-muted">Per {{ CATEGORY_TYPES[row.form.cat].label }} ({{ row.form.units.length }})</div>
              </td>
              <td>{{ row.form.direktorat }}</td>
              <td>
                <div class="fm-meter"><span :style="{ width: pct(row.s.published, row.s.total) + '%' }"></span></div>
                <div class="fm-muted">{{ row.s.published }} dari {{ row.s.total }}</div>
              </td>
              <td>
                <div class="fm-meter fm-meter--teal"><span :style="{ width: pct(row.s.progOpen, row.s.total) + '%' }"></span></div>
                <div class="fm-muted">{{ row.s.progOpen }} dari {{ row.s.total }}</div>
              </td>
              <td class="fm-muted">{{ fmtDate(row.form.lastUpdated) }}</td>
              <td class="fm-right">
                <button class="fm-btn fm-btn--primary fm-btn--sm" @click.stop="openForm(row.form)">Kelola</button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </section>

    <!-- ============ Drawer detail formulir ============ -->
    <transition name="fm-drawer-fx">
      <div v-if="selectedForm" class="fm-overlay" @click.self="requestCloseDrawer">
        <aside class="fm-drawer" role="dialog" aria-modal="true" :aria-label="selectedForm.name">
          <header class="fm-drawer__head">
            <div>
              <div class="fm-crumb">
                {{ selectedForm.direktorat }} · Per {{ catLabel }} · Diperbarui {{ fmtDate(selectedForm.lastUpdated) }}
              </div>
              <h2>{{ selectedForm.name }}</h2>
            </div>
            <button class="fm-btn fm-btn--ghost fm-btn--sm" @click="requestCloseDrawer">Tutup</button>
          </header>

          <!-- KPI -->
          <div class="fm-kpis">
            <button class="fm-kpi" :disabled="layoutEditing" @click="focusTab('rkap', 'semua')">
              <span>RKAP dipublish</span>
              <strong>{{ stats.published }}<small>/{{ stats.total }}</small></strong>
              <em>{{ stats.open }} sedang open</em>
            </button>
            <button class="fm-kpi fm-kpi--warn" :disabled="layoutEditing" @click="focusTab('rkap', 'perlu')">
              <span>RKAP perlu approval</span>
              <strong>{{ stats.rkapNeed }}</strong>
              <em>{{ stats.rkapApproved }} sudah approved</em>
            </button>
            <button class="fm-kpi" :disabled="layoutEditing" @click="focusTab('prognosa', 'semua')">
              <span>Prognosa open</span>
              <strong>{{ stats.progOpen }}<small>/{{ stats.total }}</small></strong>
              <em>{{ stats.total - stats.progOpen }} closed</em>
            </button>
            <button class="fm-kpi fm-kpi--warn" :disabled="layoutEditing" @click="focusTab('prognosa', 'perlu')">
              <span>Prognosa perlu approval</span>
              <strong>{{ stats.progNeed }}</strong>
              <em>{{ stats.progApproved }} sudah approved</em>
            </button>
          </div>

          <!-- Banner publish ulang setelah edit layout -->
          <div v-if="pendingRepublish.length" class="fm-alert fm-alert--info">
            <div>
              <strong>{{ pendingRepublish.length }} {{ catLabel }} perlu dipublish ulang.</strong>
              <span v-if="layoutEditing"> Simpan layout dulu, lalu publish ulang dari sini.</span>
              <span v-else> Publish dibatalkan untuk mengedit layout. Publish ulang dengan jadwal sebelumnya.</span>
            </div>
            <div class="fm-alert__actions">
              <button
                class="fm-btn fm-btn--primary fm-btn--sm"
                :disabled="layoutEditing || !isPublishWindowOpen"
                @click="republishAll"
              >
                Publish ulang semua
              </button>
              <button class="fm-btn fm-btn--ghost fm-btn--sm" :disabled="layoutEditing" @click="dismissRepublish">
                Abaikan
              </button>
            </div>
          </div>

          <!-- Tabs -->
          <nav class="fm-tabs" role="tablist">
            <button
              v-for="t in TABS"
              :key="t.key"
              class="fm-tab"
              role="tab"
              :aria-selected="activeTab === t.key"
              :class="{ 'is-active': activeTab === t.key }"
              :disabled="layoutEditing && t.key !== 'layout'"
              :title="layoutEditing && t.key !== 'layout' ? 'Selesaikan edit layout terlebih dahulu' : ''"
              @click="activeTab = t.key"
            >
              {{ t.label }}
              <i v-if="t.key === 'rkap' && stats.rkapNeed" class="fm-dot">{{ stats.rkapNeed }}</i>
              <i v-if="t.key === 'prognosa' && stats.progNeed" class="fm-dot">{{ stats.progNeed }}</i>
            </button>
          </nav>

          <div class="fm-drawer__body">
            <!-- ================= TAB RKAP ================= -->
            <div v-if="activeTab === 'rkap'">
              <div v-if="!isPublishWindowOpen" class="fm-alert fm-alert--warn">
                Publish RKAP {{ targetYear }} baru dibuka setiap 1 Agustus. Tenggat masih bisa diatur pada kategori yang sudah dipublish.
              </div>

              <p class="fm-intro">
                Publish RKAP per {{ catLabel }}. Setelah dipublish, RKAP terbuka sampai tenggat. Tenggat dapat diatur ulang kapan saja,
                dan publish dapat dibatalkan.
              </p>

              <div class="fm-toolbar fm-toolbar--inner">
                <label class="fm-search">
                  <svg width="15" height="15" viewBox="0 0 24 24" fill="none" aria-hidden="true">
                    <circle cx="11" cy="11" r="6.5" stroke="currentColor" stroke-width="1.8" />
                    <path d="M16 16l4.5 4.5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" />
                  </svg>
                  <input v-model="rkapSearch" :placeholder="`Cari ${catLabel.toLowerCase()}`" />
                </label>
                <div class="fm-chips">
                  <button
                    v-for="c in rkapChips"
                    :key="c.key"
                    class="fm-chip"
                    :class="{ 'is-active': rkapStatusFilter === c.key }"
                    @click="rkapStatusFilter = c.key"
                  >
                    {{ c.label }} <span>{{ rkapStatusCounts[c.key] }}</span>
                  </button>
                </div>
                <select v-model="rkapApproval" class="fm-select" aria-label="Filter approval RKAP">
                  <option value="semua">Semua approval</option>
                  <option value="perlu">Perlu approval</option>
                  <option value="approved">Approved</option>
                  <option value="kosong">Belum diisi</option>
                </select>
              </div>

              <div v-if="sel.rkap.size" class="fm-bulk">
                <strong>{{ sel.rkap.size }} dipilih</strong>
                <div class="fm-bulk__actions">
                  <button
                    class="fm-btn fm-btn--primary fm-btn--sm"
                    :disabled="!rkapBulk.publish.length || !isPublishWindowOpen"
                    @click="openPublishDialog('publish', rkapBulk.publish)"
                  >
                    Publish ({{ rkapBulk.publish.length }})
                  </button>
                  <button
                    class="fm-btn fm-btn--outline fm-btn--sm"
                    :disabled="!rkapBulk.published.length"
                    @click="openPublishDialog('deadline', rkapBulk.published)"
                  >
                    Atur tenggat ({{ rkapBulk.published.length }})
                  </button>
                  <button
                    class="fm-btn fm-btn--danger-outline fm-btn--sm"
                    :disabled="!rkapBulk.published.length"
                    @click="confirmUnpublish(rkapBulk.published)"
                  >
                    Batalkan publish ({{ rkapBulk.published.length }})
                  </button>
                  <button class="fm-btn fm-btn--ghost fm-btn--sm" @click="sel.rkap = new Set()">Batal pilih</button>
                </div>
              </div>

              <div class="fm-table-scroll">
                <table class="fm-table">
                  <thead>
                    <tr>
                      <th class="fm-check">
                        <input
                          type="checkbox"
                          :checked="allRkapChecked"
                          :disabled="!filteredRkap.length"
                          aria-label="Pilih semua"
                          @change="toggleAll('rkap', filteredRkap)"
                        />
                      </th>
                      <th>{{ catLabel }}</th>
                      <th>Status</th>
                      <th>Jadwal &amp; tenggat</th>
                      <th>Approval</th>
                      <th class="fm-right">Aksi</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-if="!filteredRkap.length">
                      <td colspan="6" class="fm-empty">Tidak ada {{ catLabel.toLowerCase() }} yang cocok dengan filter.</td>
                    </tr>
                    <tr v-for="u in filteredRkap" :key="u.name" :class="{ 'is-selected': sel.rkap.has(u.name) }">
                      <td class="fm-check">
                        <input
                          type="checkbox"
                          :checked="sel.rkap.has(u.name)"
                          :aria-label="`Pilih ${u.name}`"
                          @change="toggleSel('rkap', u.name)"
                        />
                      </td>
                      <td class="fm-name">{{ u.name }}</td>
                      <td>
                        <span class="fm-badge" :class="'fm-badge--' + RKAP_STATUS[rkapStatus(u)].cls">
                          {{ RKAP_STATUS[rkapStatus(u)].label }}
                        </span>
                      </td>
                      <td>
                        <template v-if="rkapStatus(u) === 'belum'"><span class="fm-muted">—</span></template>
                        <template v-else-if="rkapStatus(u) === 'scheduled'">
                          <div>Buka {{ fmtDateTime(u.rkap.start) }}</div>
                          <div class="fm-muted">Tenggat {{ fmtDateTime(u.rkap.deadline) }}</div>
                        </template>
                        <template v-else-if="rkapStatus(u) === 'open'">
                          <div>Tenggat {{ fmtDateTime(u.rkap.deadline) }}</div>
                          <div class="fm-remain" :class="{ 'is-urgent': isUrgent(u.rkap.deadline) }">
                            {{ remainingLabel(u.rkap.deadline) }}
                          </div>
                        </template>
                        <template v-else>
                          <div>Berakhir {{ fmtDateTime(u.rkap.deadline) }}</div>
                        </template>
                      </td>
                      <td>
                        <span v-if="rkapStatus(u) === 'belum'" class="fm-muted">—</span>
                        <span v-else class="fm-badge" :class="'fm-badge--' + APPROVAL[u.rkap.approval].cls">
                          {{ APPROVAL[u.rkap.approval].label }}
                        </span>
                      </td>
                      <td class="fm-right fm-actions">
                        <button
                          v-if="rkapStatus(u) === 'belum'"
                          class="fm-btn fm-btn--primary fm-btn--sm"
                          :disabled="!isPublishWindowOpen"
                          @click="openPublishDialog('publish', [u])"
                        >
                          Publish
                        </button>
                        <template v-else>
                          <button class="fm-btn fm-btn--outline fm-btn--sm" @click="openPublishDialog('deadline', [u])">
                            {{ rkapStatus(u) === 'closed' ? 'Buka kembali' : 'Atur tenggat' }}
                          </button>
                          <button class="fm-btn fm-btn--danger-outline fm-btn--sm" @click="confirmUnpublish([u])">
                            Batalkan publish
                          </button>
                        </template>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>

            <!-- ================= TAB PROGNOSA ================= -->
            <div v-else-if="activeTab === 'prognosa'">
              <p class="fm-intro">
                Prognosa otomatis open saat RKAP kategori terkait dipublish. Anda juga dapat membuka atau menutupnya secara manual per {{ catLabel }}.
              </p>

              <div class="fm-toolbar fm-toolbar--inner">
                <label class="fm-search">
                  <svg width="15" height="15" viewBox="0 0 24 24" fill="none" aria-hidden="true">
                    <circle cx="11" cy="11" r="6.5" stroke="currentColor" stroke-width="1.8" />
                    <path d="M16 16l4.5 4.5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" />
                  </svg>
                  <input v-model="progSearch" :placeholder="`Cari ${catLabel.toLowerCase()}`" />
                </label>
                <div class="fm-chips">
                  <button
                    v-for="c in progChips"
                    :key="c.key"
                    class="fm-chip"
                    :class="{ 'is-active': progStatusFilter === c.key }"
                    @click="progStatusFilter = c.key"
                  >
                    {{ c.label }} <span>{{ progStatusCounts[c.key] }}</span>
                  </button>
                </div>
                <select v-model="progApproval" class="fm-select" aria-label="Filter approval Prognosa">
                  <option value="semua">Semua approval</option>
                  <option value="perlu">Perlu approval</option>
                  <option value="approved">Approved</option>
                  <option value="kosong">Belum diisi</option>
                </select>
              </div>

              <div v-if="sel.prognosa.size" class="fm-bulk">
                <strong>{{ sel.prognosa.size }} dipilih</strong>
                <div class="fm-bulk__actions">
                  <button
                    class="fm-btn fm-btn--primary fm-btn--sm"
                    :disabled="!progBulk.toOpen.length"
                    @click="setPrognosa(progBulk.toOpen, true)"
                  >
                    Open ({{ progBulk.toOpen.length }})
                  </button>
                  <button
                    class="fm-btn fm-btn--danger-outline fm-btn--sm"
                    :disabled="!progBulk.toClose.length"
                    @click="setPrognosa(progBulk.toClose, false)"
                  >
                    Tutup ({{ progBulk.toClose.length }})
                  </button>
                  <button class="fm-btn fm-btn--ghost fm-btn--sm" @click="sel.prognosa = new Set()">Batal pilih</button>
                </div>
              </div>

              <div class="fm-table-scroll">
                <table class="fm-table">
                  <thead>
                    <tr>
                      <th class="fm-check">
                        <input
                          type="checkbox"
                          :checked="allProgChecked"
                          :disabled="!filteredProg.length"
                          aria-label="Pilih semua"
                          @change="toggleAll('prognosa', filteredProg)"
                        />
                      </th>
                      <th>{{ catLabel }}</th>
                      <th>Status Prognosa</th>
                      <th>Approval</th>
                      <th>Status RKAP</th>
                      <th class="fm-right">Open / Closed</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-if="!filteredProg.length">
                      <td colspan="6" class="fm-empty">Tidak ada {{ catLabel.toLowerCase() }} yang cocok dengan filter.</td>
                    </tr>
                    <tr v-for="u in filteredProg" :key="u.name" :class="{ 'is-selected': sel.prognosa.has(u.name) }">
                      <td class="fm-check">
                        <input
                          type="checkbox"
                          :checked="sel.prognosa.has(u.name)"
                          :aria-label="`Pilih ${u.name}`"
                          @change="toggleSel('prognosa', u.name)"
                        />
                      </td>
                      <td class="fm-name">{{ u.name }}</td>
                      <td>
                        <span class="fm-badge" :class="u.prognosa.open ? 'fm-badge--ok' : 'fm-badge--danger'">
                          {{ u.prognosa.open ? 'Open' : 'Closed' }}
                        </span>
                      </td>
                      <td>
                        <span class="fm-badge" :class="'fm-badge--' + APPROVAL[u.prognosa.approval].cls">
                          {{ APPROVAL[u.prognosa.approval].label }}
                        </span>
                      </td>
                      <td>
                        <span class="fm-badge fm-badge--soft" :class="'fm-badge--' + RKAP_STATUS[rkapStatus(u)].cls">
                          RKAP {{ RKAP_STATUS[rkapStatus(u)].label.toLowerCase() }}
                        </span>
                      </td>
                      <td class="fm-right">
                        <button
                          class="fm-switch"
                          :class="{ 'is-on': u.prognosa.open }"
                          role="switch"
                          :aria-checked="u.prognosa.open"
                          :aria-label="`${u.prognosa.open ? 'Tutup' : 'Buka'} prognosa ${u.name}`"
                          @click="setPrognosa([u], !u.prognosa.open)"
                        >
                          <span></span>
                        </button>
                      </td>
                    </tr>
                  </tbody>
                </table>
              </div>
            </div>

            <!-- ================= TAB LAYOUT ================= -->
            <div v-else>
              <div class="fm-layout-head">
                <div>
                  <h3>Layout formulir RKAP {{ targetYear }}</h3>
                  <p>
                    Struktur tabel terdiri dari label, sub label (opsional), dan uraian. Header kolom tabel bersifat tetap dan tidak ikut berubah.
                  </p>
                </div>
                <div v-if="!layoutEditing" class="fm-layout-actions">
                  <button
                    v-if="!layoutLocked"
                    class="fm-btn fm-btn--primary"
                    @click="startEdit"
                  >
                    Edit layout
                  </button>
                </div>
              </div>

              <!-- Terkunci -->
              <div v-if="layoutLocked && !layoutEditing" class="fm-lock">
                <svg width="20" height="20" viewBox="0 0 24 24" fill="none" aria-hidden="true">
                  <rect x="5" y="11" width="14" height="9" rx="2" stroke="currentColor" stroke-width="1.8" />
                  <path d="M8 11V8a4 4 0 0 1 8 0v3" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" />
                </svg>
                <div>
                  <strong>Layout terkunci</strong>
                  <p>
                    RKAP sudah dipublish di {{ stats.published }} {{ catLabel.toLowerCase() }}. Layout hanya bisa diedit saat RKAP belum dipublish.
                    Batalkan publish, edit, lalu publish ulang dengan sekali klik.
                  </p>
                </div>
                <button class="fm-btn fm-btn--danger-outline" @click="startEditFlow">Batalkan publish &amp; edit</button>
              </div>

              <!-- Pratinjau -->
              <div v-if="!layoutEditing" class="fm-table-scroll">
                <table class="fm-table fm-table--layout">
                  <thead>
                    <tr>
                      <th class="fm-no">No</th>
                      <th>Uraian</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="r in layoutRows" :key="r.key" :class="'fm-lr fm-lr--' + r.type">
                      <td class="fm-no">{{ r.no || '' }}</td>
                      <td :class="{ 'is-indent': r.indent }">{{ r.text }}</td>
                    </tr>
                  </tbody>
                </table>
              </div>

              <!-- Editor -->
              <div v-else class="fm-editor">
                <div v-if="showLayoutErrors && layoutErrors.length" class="fm-alert fm-alert--danger">
                  <div>
                    <strong>Layout belum bisa disimpan:</strong>
                    <ul>
                      <li v-for="(e, i) in layoutErrors.slice(0, 5)" :key="i">{{ e }}</li>
                      <li v-if="layoutErrors.length > 5">dan {{ layoutErrors.length - 5 }} masalah lainnya</li>
                    </ul>
                  </div>
                </div>

                <div v-for="(lb, li) in draft" :key="lb.id" class="fm-lb">
                  <div class="fm-lb__head">
                    <span class="fm-tag fm-tag--label">Label</span>
                    <input
                      v-model="lb.label"
                      class="fm-input"
                      :class="{ 'is-invalid': showLayoutErrors && !lb.label.trim() }"
                      placeholder="Nama label"
                    />
                    <div class="fm-mini">
                      <button title="Naikkan" :disabled="li === 0" @click="move(draft, li, -1)">▲</button>
                      <button title="Turunkan" :disabled="li === draft.length - 1" @click="move(draft, li, 1)">▼</button>
                      <button title="Hapus label" :disabled="draft.length === 1" @click="draft.splice(li, 1)">✕</button>
                    </div>
                  </div>

                  <div v-for="(sb, si) in lb.subs" :key="sb.id" class="fm-sb">
                    <div class="fm-sb__head">
                      <span class="fm-tag fm-tag--sub">Sub label</span>
                      <input v-model="sb.name" class="fm-input" placeholder="Nama sub label (opsional)" />
                      <div class="fm-mini">
                        <button title="Naikkan" :disabled="si === 0" @click="move(lb.subs, si, -1)">▲</button>
                        <button title="Turunkan" :disabled="si === lb.subs.length - 1" @click="move(lb.subs, si, 1)">▼</button>
                        <button title="Hapus sub label" :disabled="lb.subs.length === 1" @click="lb.subs.splice(si, 1)">✕</button>
                      </div>
                    </div>

                    <div v-for="(ur, ui) in sb.uraians" :key="ur.id" class="fm-ur">
                      <span class="fm-ur__no">{{ ui + 1 }}</span>
                      <input
                        v-model="ur.text"
                        class="fm-input"
                        :class="{ 'is-invalid': showLayoutErrors && !ur.text.trim() }"
                        placeholder="Uraian"
                      />
                      <div class="fm-mini">
                        <button title="Naikkan" :disabled="ui === 0" @click="move(sb.uraians, ui, -1)">▲</button>
                        <button title="Turunkan" :disabled="ui === sb.uraians.length - 1" @click="move(sb.uraians, ui, 1)">▼</button>
                        <button title="Hapus uraian" :disabled="sb.uraians.length === 1" @click="sb.uraians.splice(ui, 1)">✕</button>
                      </div>
                    </div>
                    <button class="fm-link" @click="sb.uraians.push(makeUraian(''))">+ Tambah uraian</button>
                  </div>

                  <button class="fm-link" @click="lb.subs.push(makeSub('', ['']))">+ Tambah sub label</button>
                </div>

                <button class="fm-btn fm-btn--outline" @click="draft.push(makeLabel())">+ Tambah label</button>

                <div class="fm-editor__foot">
                  <span class="fm-muted">{{ layoutDirty ? 'Ada perubahan yang belum disimpan' : 'Belum ada perubahan' }}</span>
                  <div>
                    <button class="fm-btn fm-btn--ghost" @click="cancelEdit">Batal</button>
                    <button class="fm-btn fm-btn--primary" @click="saveLayout">Simpan layout</button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </aside>
      </div>
    </transition>

    <!-- ============ Dialog publish / atur tenggat ============ -->
    <transition name="fm-fade">
      <div v-if="dlg.open" class="fm-modal-overlay" @click.self="dlg.open = false">
        <div class="fm-modal" role="dialog" aria-modal="true">
          <h3>{{ dlg.mode === 'publish' ? 'Publish RKAP' : 'Atur tenggat RKAP' }}</h3>
          <p class="fm-muted">
            {{ dlgUnits.length === 1 ? dlgUnits[0].name : `${dlgUnits.length} ${catLabel.toLowerCase()} dipilih` }}
          </p>

          <ul v-if="dlgUnits.length > 1" class="fm-list">
            <li v-for="u in dlgUnits" :key="u.name">{{ u.name }}</li>
          </ul>

          <div v-if="dlg.mode === 'publish'" class="fm-field">
            <span class="fm-label">Waktu buka</span>
            <div class="fm-seg">
              <button :class="{ 'is-active': dlg.startMode === 'now' }" @click="dlg.startMode = 'now'">Sekarang</button>
              <button :class="{ 'is-active': dlg.startMode === 'schedule' }" @click="dlg.startMode = 'schedule'">Jadwalkan</button>
            </div>
            <div v-if="dlg.startMode === 'schedule'" class="fm-row2">
              <input v-model="dlg.startDate" type="date" class="fm-input" aria-label="Tanggal buka" />
              <input v-model="dlg.startTime" type="time" class="fm-input" aria-label="Jam buka" />
            </div>
          </div>

          <div class="fm-field">
            <span class="fm-label">Tenggat pengisian</span>
            <div class="fm-row2">
              <input v-model="dlg.deadlineDate" type="date" class="fm-input" aria-label="Tanggal tenggat" />
              <input v-model="dlg.deadlineTime" type="time" class="fm-input" aria-label="Jam tenggat" />
            </div>
            <div class="fm-chips fm-chips--tight">
              <button class="fm-chip" @click="setDeadlineIn(7)">+7 hari</button>
              <button class="fm-chip" @click="setDeadlineIn(14)">+14 hari</button>
              <button class="fm-chip" @click="setDeadlineIn(30)">+30 hari</button>
            </div>
          </div>

          <p v-if="dlg.error" class="fm-error">{{ dlg.error }}</p>

          <div class="fm-modal__foot">
            <button class="fm-btn fm-btn--ghost" @click="dlg.open = false">Batal</button>
            <button class="fm-btn fm-btn--primary" @click="submitDialog">
              {{ dlg.mode === 'publish' ? 'Publish' : 'Simpan tenggat' }}
            </button>
          </div>
        </div>
      </div>
    </transition>

    <!-- ============ Dialog konfirmasi ============ -->
    <transition name="fm-fade">
      <div v-if="confirmDlg.open" class="fm-modal-overlay fm-modal-overlay--top" @click.self="confirmDlg.open = false">
        <div class="fm-modal fm-modal--sm" role="alertdialog" aria-modal="true">
          <h3>{{ confirmDlg.title }}</h3>
          <p class="fm-confirm-msg">{{ confirmDlg.message }}</p>
          <ul v-if="confirmDlg.items.length" class="fm-list">
            <li v-for="it in confirmDlg.items" :key="it">{{ it }}</li>
          </ul>
          <div class="fm-modal__foot">
            <button class="fm-btn fm-btn--ghost" @click="confirmDlg.open = false">Kembali</button>
            <button class="fm-btn" :class="confirmDlg.danger ? 'fm-btn--danger' : 'fm-btn--primary'" @click="runConfirm">
              {{ confirmDlg.confirmLabel }}
            </button>
          </div>
        </div>
      </div>
    </transition>

    <!-- Toast -->
    <transition name="fm-fade">
      <div v-if="toast" class="fm-toast" :class="'fm-toast--' + toast.type" role="status">{{ toast.msg }}</div>
    </transition>
  </section>
</template>

<script setup>
import { computed, onBeforeUnmount, onMounted, reactive, ref, watch } from 'vue'

/* ==========================================================
 * Master data (DEMO) — ganti dengan response API di production
 * ========================================================== */
const businessAreas = [
  'Daop 1 Jakarta',
  'Daop 2 Bandung',
  'Daop 3 Cirebon',
  'Daop 4 Semarang',
  'Daop 5 Purwokerto',
  'Daop 6 Yogyakarta',
  'Daop 7 Madiun',
  'Daop 8 Surabaya',
  'Daop 9 Jember',
  'Divre I Sumatera Utara',
  'Divre II Sumatera Barat',
  'Divre III Palembang',
  'Divre IV Tanjungkarang',
]

// Jenis kategori yang dipublish per formulir
const CATEGORY_TYPES = {
  area: { label: 'Area Bisnis', items: businessAreas },
  sarana: {
    label: 'Jenis Sarana',
    items: ['Lokomotif', 'KRL', 'KRD / DMU', 'Kereta Penumpang', 'Gerbong', 'Peralatan Khusus'],
  },
  balai: {
    label: 'Balai Yasa',
    items: [
      'Balai Yasa Manggarai',
      'Balai Yasa Yogyakarta',
      'Balai Yasa Surabaya Gubeng',
      'Balai Yasa Tegal',
      'Balai Yasa Lahat',
      'Balai Yasa Bandung',
    ],
  },
}

// 16 formulir tetap
const FORM_BLUEPRINTS = [
  { name: 'Usulan Hari Kerja Balai Yasa', direktorat: 'Balai Yasa', cat: 'balai', lastUpdated: '2026-09-18' },
  { name: 'Usulan Produksi Sarana', direktorat: 'Sarana', cat: 'sarana', lastUpdated: '2026-09-20' },
  { name: 'Usulan Produksi Prasarana Sintel Pemerintah', direktorat: 'Prasarana', cat: 'area', lastUpdated: '2026-09-15' },
  { name: 'Usulan Produksi Prasarana Rel KAI', direktorat: 'Prasarana', cat: 'area', lastUpdated: '2026-09-12' },
  { name: 'Usulan Produksi Prasarana Jembatan KAI', direktorat: 'Prasarana', cat: 'area', lastUpdated: '2026-09-12' },
  { name: 'Usulan Produksi Asset Prasarana Sintel', direktorat: 'Prasarana', cat: 'area', lastUpdated: '2026-09-10' },
  { name: 'Usulan Produksi Penumpang', direktorat: 'Komersial', cat: 'area', lastUpdated: '2026-09-22' },
  { name: 'Usulan Produksi Pelayanan', direktorat: 'Komersial', cat: 'area', lastUpdated: '2026-09-21' },
  { name: 'Usulan Produksi Perawatan Sarana', direktorat: 'Sarana', cat: 'sarana', lastUpdated: '2026-09-19' },
  { name: 'Usulan Tanggungan Balai Yasa', direktorat: 'Balai Yasa', cat: 'balai', lastUpdated: '2026-09-08' },
  { name: 'Usulan Produksi Prasarana Rel Pemerintah', direktorat: 'Prasarana', cat: 'area', lastUpdated: '2026-09-09' },
  { name: 'Usulan Produksi Prasarana Jembatan Pemerintah', direktorat: 'Prasarana', cat: 'area', lastUpdated: '2026-09-09' },
  { name: 'Usulan Produksi Asset Prasarana Jalan Jembatan', direktorat: 'Prasarana', cat: 'area', lastUpdated: '2026-09-07' },
  { name: 'Usulan Produksi Sumber Daya Manusia', direktorat: 'SDM', cat: 'area', lastUpdated: '2026-09-17' },
  { name: 'Usulan Produksi Barang', direktorat: 'Logistik', cat: 'area', lastUpdated: '2026-09-14' },
  { name: 'Usulan Norma Kendali Kecelakaan Kereta Api', direktorat: 'Keselamatan & Operasi', cat: 'area', lastUpdated: '2026-09-23' },
]

const TABS = [
  { key: 'rkap', label: 'Publish RKAP' },
  { key: 'prognosa', label: 'Prognosa' },
  { key: 'layout', label: 'Layout formulir' },
]

const RKAP_STATUS = {
  belum: { label: 'Belum publish', cls: 'muted' },
  scheduled: { label: 'Terjadwal', cls: 'info' },
  open: { label: 'Open', cls: 'ok' },
  closed: { label: 'Closed', cls: 'danger' },
}

const APPROVAL = {
  kosong: { label: 'Belum diisi', cls: 'muted' },
  perlu: { label: 'Perlu approval', cls: 'warn' },
  approved: { label: 'Approved', cls: 'ok' },
}

/* ==========================================================
 * Helper tanggal
 * ========================================================== */
const pad = n => String(n).padStart(2, '0')
const toDateInput = d => `${d.getFullYear()}-${pad(d.getMonth() + 1)}-${pad(d.getDate())}`
const toTimeInput = d => `${pad(d.getHours())}:${pad(d.getMinutes())}`
const addDays = (d, n) => {
  const x = new Date(d)
  x.setDate(x.getDate() + n)
  return x
}
const fmtDate = d => new Date(d).toLocaleDateString('id-ID', { day: 'numeric', month: 'short', year: 'numeric' })
const fmtDateTime = d =>
  `${fmtDate(d)}, ${new Date(d).toLocaleTimeString('id-ID', { hour: '2-digit', minute: '2-digit' })}`
const parseDT = (date, time) => {
  if (!date || !time) return null
  const x = new Date(`${date}T${time}`)
  return isNaN(x) ? null : x
}
const pct = (a, b) => (b ? Math.round((a / b) * 100) : 0)

const now = ref(new Date())
const targetYear = computed(() => now.value.getFullYear() + 1)
// RKAP tahun berikutnya hanya bisa dipublish mulai 1 Agustus
const isPublishWindowOpen = computed(() => now.value.getMonth() >= 7)

let clock = null

function remainingLabel(d) {
  const ms = new Date(d) - now.value
  if (ms <= 0) return ''
  const days = Math.floor(ms / 86400000)
  if (days >= 1) return `Sisa ${days} hari`
  return `Sisa ${Math.max(1, Math.floor(ms / 3600000))} jam`
}
const isUrgent = d => new Date(d) - now.value < 2 * 86400000

/* ==========================================================
 * Seed data demo
 * ========================================================== */
let uidCounter = 0
const uid = () => ++uidCounter

const rand = seed => {
  const x = Math.sin(seed * 9973) * 10000
  return x - Math.floor(x)
}
const daysFromNow = n => {
  const t = addDays(new Date(), n)
  t.setHours(23, 59, 0, 0)
  return t
}
const pickApproval = r => (r > 0.66 ? 'approved' : r > 0.33 ? 'perlu' : 'kosong')

function buildUnits(formIndex, catKey) {
  const noPublish = formIndex % 5 === 4 // beberapa formulir belum pernah dipublish
  return CATEGORY_TYPES[catKey].items.map((name, i) => {
    const r = rand(formIndex * 31 + i + 1)
    const r2 = rand(formIndex * 17 + i * 7 + 3)
    const rkap = { published: false, start: null, deadline: null, approval: 'kosong' }

    if (!noPublish && r > 0.3) {
      rkap.published = true
      if (r > 0.9) {
        rkap.start = daysFromNow(3)
        rkap.deadline = daysFromNow(17)
      } else if (r > 0.8) {
        rkap.start = daysFromNow(-20)
        rkap.deadline = daysFromNow(-3)
        rkap.approval = pickApproval(r2)
      } else {
        rkap.deadline = daysFromNow(1 + Math.floor(r2 * 12))
        rkap.approval = pickApproval(r2)
      }
    }

    const prognosa = {
      open: rand(formIndex * 13 + i * 5 + 9) > 0.45,
      approval: pickApproval(rand(formIndex * 11 + i * 3 + 5)),
    }
    return { name, rkap, prognosa }
  })
}

const makeUraian = text => ({ id: uid(), text })
const makeSub = (name, list) => ({ id: uid(), name, uraians: list.map(makeUraian) })
const makeLabel = () => ({ id: uid(), label: '', subs: [makeSub('', [''])] })

function buildLayout() {
  return [
    {
      id: uid(),
      label: 'Volume produksi',
      subs: [
        makeSub('Realisasi', ['Realisasi tahun berjalan', 'Estimasi sisa tahun']),
        makeSub('Target', ['Target RKAP', 'Target semester I']),
      ],
    },
    {
      id: uid(),
      label: 'Biaya dan sumber daya',
      subs: [makeSub('', ['Kebutuhan tenaga kerja', 'Kebutuhan material'])],
    },
  ]
}

const forms = reactive(
  FORM_BLUEPRINTS.map((f, i) => ({
    id: i + 1,
    name: f.name,
    direktorat: f.direktorat,
    cat: f.cat,
    lastUpdated: f.lastUpdated,
    units: buildUnits(i, f.cat),
    layout: buildLayout(),
  }))
)

/* ==========================================================
 * Status & statistik
 * ========================================================== */
function rkapStatus(u) {
  const r = u.rkap
  if (!r.published) return 'belum'
  if (r.start && new Date(r.start) > now.value) return 'scheduled'
  if (r.deadline && new Date(r.deadline) <= now.value) return 'closed'
  return 'open'
}

function formStats(f) {
  const s = {
    total: f.units.length,
    published: 0,
    open: 0,
    rkapNeed: 0,
    rkapApproved: 0,
    progOpen: 0,
    progNeed: 0,
    progApproved: 0,
  }
  f.units.forEach(u => {
    const st = rkapStatus(u)
    if (st !== 'belum') {
      s.published++
      if (u.rkap.approval === 'perlu') s.rkapNeed++
      if (u.rkap.approval === 'approved') s.rkapApproved++
    }
    if (st === 'open') s.open++
    if (u.prognosa.open) s.progOpen++
    if (u.prognosa.approval === 'perlu') s.progNeed++
    if (u.prognosa.approval === 'approved') s.progApproved++
  })
  return s
}

const overview = computed(() => {
  const o = { total: 0, published: 0, need: 0, progOpen: 0 }
  forms.forEach(f => {
    const s = formStats(f)
    o.total += s.total
    o.published += s.published
    o.need += s.rkapNeed + s.progNeed
    o.progOpen += s.progOpen
  })
  return o
})

/* ==========================================================
 * Daftar formulir
 * ========================================================== */
const searchQuery = ref('')
const direktoratFilter = ref('semua')
const listFilter = ref('semua')

const direktoratOptions = computed(() => [...new Set(forms.map(f => f.direktorat))])
const listChips = [
  { key: 'semua', label: 'Semua' },
  { key: 'approval', label: 'Menunggu approval' },
  { key: 'belum', label: 'Belum dipublish' },
]

const formRows = computed(() => {
  const q = searchQuery.value.trim().toLowerCase()
  return forms
    .filter(f => !q || f.name.toLowerCase().includes(q))
    .filter(f => direktoratFilter.value === 'semua' || f.direktorat === direktoratFilter.value)
    .map(f => ({ form: f, s: formStats(f) }))
    .filter(r => {
      if (listFilter.value === 'approval') return r.s.rkapNeed + r.s.progNeed > 0
      if (listFilter.value === 'belum') return r.s.published === 0
      return true
    })
})

/* ==========================================================
 * Drawer detail
 * ========================================================== */
const selectedFormId = ref(null)
const activeTab = ref('rkap')
const selectedForm = computed(() => forms.find(f => f.id === selectedFormId.value) || null)
const catLabel = computed(() => (selectedForm.value ? CATEGORY_TYPES[selectedForm.value.cat].label : ''))
const stats = computed(() => (selectedForm.value ? formStats(selectedForm.value) : formStats({ units: [] })))

// Seleksi baris (bulk)
const sel = reactive({ rkap: new Set(), prognosa: new Set() })

function toggleSel(type, key) {
  const next = new Set(sel[type])
  next.has(key) ? next.delete(key) : next.add(key)
  sel[type] = next
}
function toggleAll(type, list) {
  const all = list.length > 0 && list.every(u => sel[type].has(u.name))
  sel[type] = all ? new Set() : new Set(list.map(u => u.name))
}

// Filter RKAP
const rkapSearch = ref('')
const rkapStatusFilter = ref('semua')
const rkapApproval = ref('semua')
const rkapChips = [
  { key: 'semua', label: 'Semua' },
  { key: 'belum', label: 'Belum publish' },
  { key: 'scheduled', label: 'Terjadwal' },
  { key: 'open', label: 'Open' },
  { key: 'closed', label: 'Closed' },
]
const rkapStatusCounts = computed(() => {
  const c = { semua: 0, belum: 0, scheduled: 0, open: 0, closed: 0 }
  ;(selectedForm.value?.units || []).forEach(u => {
    c.semua++
    c[rkapStatus(u)]++
  })
  return c
})
const filteredRkap = computed(() => {
  const q = rkapSearch.value.trim().toLowerCase()
  return (selectedForm.value?.units || []).filter(u => {
    if (q && !u.name.toLowerCase().includes(q)) return false
    if (rkapStatusFilter.value !== 'semua' && rkapStatus(u) !== rkapStatusFilter.value) return false
    if (rkapApproval.value !== 'semua') {
      if (rkapStatus(u) === 'belum' || u.rkap.approval !== rkapApproval.value) return false
    }
    return true
  })
})
const allRkapChecked = computed(
  () => filteredRkap.value.length > 0 && filteredRkap.value.every(u => sel.rkap.has(u.name))
)
const rkapSelectedUnits = computed(() => (selectedForm.value?.units || []).filter(u => sel.rkap.has(u.name)))
const rkapBulk = computed(() => ({
  publish: rkapSelectedUnits.value.filter(u => rkapStatus(u) === 'belum'),
  published: rkapSelectedUnits.value.filter(u => rkapStatus(u) !== 'belum'),
}))

// Filter Prognosa
const progSearch = ref('')
const progStatusFilter = ref('semua')
const progApproval = ref('semua')
const progChips = [
  { key: 'semua', label: 'Semua' },
  { key: 'open', label: 'Open' },
  { key: 'closed', label: 'Closed' },
]
const progStatusCounts = computed(() => {
  const units = selectedForm.value?.units || []
  return {
    semua: units.length,
    open: units.filter(u => u.prognosa.open).length,
    closed: units.filter(u => !u.prognosa.open).length,
  }
})
const filteredProg = computed(() => {
  const q = progSearch.value.trim().toLowerCase()
  return (selectedForm.value?.units || []).filter(u => {
    if (q && !u.name.toLowerCase().includes(q)) return false
    if (progStatusFilter.value === 'open' && !u.prognosa.open) return false
    if (progStatusFilter.value === 'closed' && u.prognosa.open) return false
    if (progApproval.value !== 'semua' && u.prognosa.approval !== progApproval.value) return false
    return true
  })
})
const allProgChecked = computed(
  () => filteredProg.value.length > 0 && filteredProg.value.every(u => sel.prognosa.has(u.name))
)
const progBulk = computed(() => {
  const chosen = (selectedForm.value?.units || []).filter(u => sel.prognosa.has(u.name))
  return { toOpen: chosen.filter(u => !u.prognosa.open), toClose: chosen.filter(u => u.prognosa.open) }
})

// Ubah filter → bersihkan seleksi supaya tidak ada pilihan tersembunyi
watch([rkapSearch, rkapStatusFilter, rkapApproval], () => (sel.rkap = new Set()))
watch([progSearch, progStatusFilter, progApproval], () => (sel.prognosa = new Set()))

function resetDrawerState() {
  activeTab.value = 'rkap'
  sel.rkap = new Set()
  sel.prognosa = new Set()
  rkapSearch.value = ''
  rkapStatusFilter.value = 'semua'
  rkapApproval.value = 'semua'
  progSearch.value = ''
  progStatusFilter.value = 'semua'
  progApproval.value = 'semua'
  layoutEditing.value = false
  draft.value = null
}

function openForm(form) {
  resetDrawerState()
  selectedFormId.value = form.id
}

function focusTab(tab, approval) {
  activeTab.value = tab
  if (tab === 'rkap') {
    rkapStatusFilter.value = 'semua'
    rkapApproval.value = approval
  } else {
    progStatusFilter.value = 'semua'
    progApproval.value = approval
  }
}

function closeDrawer() {
  selectedFormId.value = null
  resetDrawerState()
}

function requestCloseDrawer() {
  if (layoutEditing.value && layoutDirty.value) {
    askConfirm({
      title: 'Buang perubahan layout?',
      message: 'Perubahan layout yang belum disimpan akan hilang.',
      confirmLabel: 'Buang & tutup',
      danger: true,
      action: closeDrawer,
    })
    return
  }
  closeDrawer()
}

/* ==========================================================
 * Aksi RKAP: publish, atur tenggat, batalkan publish
 * ========================================================== */
const dlg = reactive({
  open: false,
  mode: 'publish', // 'publish' | 'deadline'
  names: [],
  startMode: 'now',
  startDate: '',
  startTime: '08:00',
  deadlineDate: '',
  deadlineTime: '23:59',
  error: '',
})

const dlgUnits = computed(() => (selectedForm.value?.units || []).filter(u => dlg.names.includes(u.name)))

function openPublishDialog(mode, units) {
  if (!units.length) return
  const base = new Date()
  const single = units.length === 1 ? units[0] : null
  const existing = mode === 'deadline' && single?.rkap.deadline ? new Date(single.rkap.deadline) : null
  const deadline = existing && existing > base ? existing : addDays(base, 14)
  Object.assign(dlg, {
    open: true,
    mode,
    names: units.map(u => u.name),
    startMode: 'now',
    startDate: toDateInput(addDays(base, 1)),
    startTime: '08:00',
    deadlineDate: toDateInput(deadline),
    deadlineTime: existing && existing > base ? toTimeInput(existing) : '23:59',
    error: '',
  })
}

function setDeadlineIn(days) {
  const base =
    dlg.mode === 'publish' && dlg.startMode === 'schedule' && dlg.startDate
      ? new Date(`${dlg.startDate}T00:00`)
      : new Date()
  dlg.deadlineDate = toDateInput(addDays(base, days))
  dlg.error = ''
}

function submitDialog() {
  const deadline = parseDT(dlg.deadlineDate, dlg.deadlineTime)
  if (!deadline) {
    dlg.error = 'Tenggat wajib diisi.'
    return
  }

  let start = null
  if (dlg.mode === 'publish' && dlg.startMode === 'schedule') {
    start = parseDT(dlg.startDate, dlg.startTime)
    if (!start) {
      dlg.error = 'Tanggal dan jam buka wajib diisi.'
      return
    }
    if (start <= new Date()) {
      dlg.error = 'Waktu buka harus di masa depan. Pilih "Sekarang" untuk membuka langsung.'
      return
    }
  }

  const units = dlgUnits.value
  for (const u of units) {
    const from =
      dlg.mode === 'publish'
        ? start || new Date()
        : u.rkap.start && new Date(u.rkap.start) > new Date()
          ? new Date(u.rkap.start)
          : new Date()
    if (deadline <= from) {
      dlg.error = 'Tenggat harus setelah waktu buka (atau setelah waktu sekarang).'
      return
    }
  }

  if (dlg.mode === 'publish') {
    units.forEach(u => {
      Object.assign(u.rkap, { published: true, start, deadline: new Date(deadline), approval: 'kosong' })
      u.prognosa.open = true // aturan: Prognosa otomatis open saat RKAP dipublish
    })
    showToast(`RKAP dipublish untuk ${units.length} ${catLabel.value.toLowerCase()}.`)
  } else {
    units.forEach(u => {
      u.rkap.deadline = new Date(deadline)
    })
    showToast(`Tenggat diperbarui untuk ${units.length} ${catLabel.value.toLowerCase()}.`)
  }
  sel.rkap = new Set()
  dlg.open = false
}

function unpublish(units) {
  units.forEach(u => Object.assign(u.rkap, { published: false, start: null, deadline: null, approval: 'kosong' }))
}

function confirmUnpublish(units) {
  if (!units.length) return
  askConfirm({
    title: 'Batalkan publish RKAP?',
    message: `${units.length} ${catLabel.value.toLowerCase()} ditarik dari publikasi. Pengisian ditutup, dan isian serta status approval yang sudah ada direset.`,
    items: units.map(u => u.name),
    confirmLabel: 'Batalkan publish',
    danger: true,
    action: () => {
      unpublish(units)
      sel.rkap = new Set()
      showToast(`Publish dibatalkan untuk ${units.length} ${catLabel.value.toLowerCase()}.`, 'warn')
    },
  })
}

/* ==========================================================
 * Aksi Prognosa
 * ========================================================== */
function setPrognosa(units, open) {
  if (!units.length) return
  units.forEach(u => (u.prognosa.open = open))
  sel.prognosa = new Set()
  showToast(
    units.length === 1
      ? `Prognosa ${units[0].name} ${open ? 'dibuka' : 'ditutup'}.`
      : `Prognosa ${open ? 'dibuka' : 'ditutup'} untuk ${units.length} ${catLabel.value.toLowerCase()}.`,
    open ? 'success' : 'warn'
  )
}

/* ==========================================================
 * Layout formulir (label → sub label → uraian)
 * ========================================================== */
const layoutEditing = ref(false)
const draft = ref(null)
const showLayoutErrors = ref(false)
const republishQueue = reactive({}) // { [formId]: [{ name, start, deadline }] }

const clone = v => JSON.parse(JSON.stringify(v))

// Layout terkunci selama masih ada kategori yang dipublish
const layoutLocked = computed(() => stats.value.published > 0)

const layoutDirty = computed(
  () => !!draft.value && !!selectedForm.value && JSON.stringify(draft.value) !== JSON.stringify(selectedForm.value.layout)
)

function flattenLayout(layout) {
  let n = 0
  const rows = []
  layout.forEach(lb => {
    rows.push({ key: `l${lb.id}`, type: 'label', text: lb.label })
    lb.subs.forEach(sb => {
      const hasSub = !!sb.name.trim()
      if (hasSub) rows.push({ key: `s${sb.id}`, type: 'sub', text: sb.name })
      sb.uraians.forEach(ur => rows.push({ key: `u${ur.id}`, type: 'uraian', no: ++n, text: ur.text, indent: hasSub }))
    })
  })
  return rows
}
const layoutRows = computed(() => (selectedForm.value ? flattenLayout(selectedForm.value.layout) : []))

function validateLayout(layout) {
  const errs = []
  if (!layout.length) errs.push('Minimal harus ada 1 label.')
  layout.forEach((lb, i) => {
    const lname = lb.label.trim() || `Label ${i + 1}`
    if (!lb.label.trim()) errs.push(`Label ${i + 1} belum diberi nama.`)
    lb.subs.forEach(sb => {
      sb.uraians.forEach((ur, k) => {
        if (!ur.text.trim()) {
          errs.push(`Uraian ${k + 1} pada "${lname}"${sb.name.trim() ? ` › ${sb.name.trim()}` : ''} masih kosong.`)
        }
      })
    })
  })
  return errs
}
const layoutErrors = computed(() => (draft.value ? validateLayout(draft.value) : []))

function move(arr, i, dir) {
  const j = i + dir
  if (j < 0 || j >= arr.length) return
  const [x] = arr.splice(i, 1)
  arr.splice(j, 0, x)
}

function startEdit() {
  if (!selectedForm.value) return
  draft.value = clone(selectedForm.value.layout)
  showLayoutErrors.value = false
  layoutEditing.value = true
}

// Terkunci → batalkan publish dulu, simpan antrean untuk publish ulang
function startEditFlow() {
  const form = selectedForm.value
  const published = form.units.filter(u => rkapStatus(u) !== 'belum')
  if (!published.length) return startEdit()

  askConfirm({
    title: 'Batalkan publish untuk mengedit layout?',
    message: `Layout hanya bisa diedit saat RKAP belum dipublish. ${published.length} ${catLabel.value.toLowerCase()} akan ditarik dari publikasi dan isian serta approval-nya direset. Setelah layout disimpan, Anda bisa publish ulang dengan jadwal sebelumnya.`,
    items: published.map(u => u.name),
    confirmLabel: 'Batalkan publish & edit',
    danger: true,
    action: () => {
      const prev = republishQueue[form.id] || []
      const names = new Set(published.map(u => u.name))
      republishQueue[form.id] = [
        ...prev.filter(p => !names.has(p.name)),
        ...published.map(u => ({ name: u.name, start: u.rkap.start, deadline: u.rkap.deadline })),
      ]
      unpublish(published)
      startEdit()
    },
  })
}

function cancelEdit() {
  if (layoutDirty.value) {
    askConfirm({
      title: 'Buang perubahan layout?',
      message: 'Perubahan yang belum disimpan akan hilang.',
      confirmLabel: 'Buang perubahan',
      danger: true,
      action: () => {
        layoutEditing.value = false
        draft.value = null
      },
    })
    return
  }
  layoutEditing.value = false
  draft.value = null
}

function saveLayout() {
  if (layoutErrors.value.length) {
    showLayoutErrors.value = true
    return
  }
  const form = selectedForm.value
  form.layout = clone(draft.value)
  form.lastUpdated = toDateInput(new Date())
  layoutEditing.value = false
  draft.value = null
  const pending = (republishQueue[form.id] || []).length
  showToast(pending ? 'Layout disimpan. Publish ulang kategori dari banner di atas.' : 'Layout disimpan.')
}

// Publish ulang setelah edit layout
const pendingRepublish = computed(() => (selectedForm.value ? republishQueue[selectedForm.value.id] || [] : []))

function republishAll() {
  const form = selectedForm.value
  const queue = republishQueue[form.id] || []
  let count = 0
  queue.forEach(q => {
    const u = form.units.find(x => x.name === q.name)
    if (!u || rkapStatus(u) !== 'belum') return
    const start = q.start && new Date(q.start) > new Date() ? new Date(q.start) : null
    const from = start || new Date()
    const deadline = q.deadline && new Date(q.deadline) > from ? new Date(q.deadline) : addDays(from, 14)
    Object.assign(u.rkap, { published: true, start, deadline, approval: 'kosong' })
    u.prognosa.open = true
    count++
  })
  delete republishQueue[form.id]
  showToast(`RKAP dipublish ulang untuk ${count} ${catLabel.value.toLowerCase()}.`)
}

function dismissRepublish() {
  delete republishQueue[selectedForm.value.id]
}

/* ==========================================================
 * Dialog konfirmasi, toast, keyboard
 * ========================================================== */
const confirmDlg = reactive({
  open: false,
  title: '',
  message: '',
  items: [],
  confirmLabel: 'Ya, lanjutkan',
  danger: false,
  action: null,
})

function askConfirm(opts) {
  Object.assign(confirmDlg, {
    open: true,
    title: '',
    message: '',
    items: [],
    confirmLabel: 'Ya, lanjutkan',
    danger: false,
    action: null,
    ...opts,
  })
}
function runConfirm() {
  const fn = confirmDlg.action
  confirmDlg.open = false
  if (fn) fn()
}

const toast = ref(null)
let toastTimer = null
function showToast(msg, type = 'success') {
  toast.value = { msg, type }
  clearTimeout(toastTimer)
  toastTimer = setTimeout(() => (toast.value = null), 3400)
}

function onKey(e) {
  if (e.key !== 'Escape') return
  if (confirmDlg.open) confirmDlg.open = false
  else if (dlg.open) dlg.open = false
  else if (selectedForm.value) requestCloseDrawer()
}

onMounted(() => {
  clock = setInterval(() => (now.value = new Date()), 60000)
  window.addEventListener('keydown', onKey)
})
onBeforeUnmount(() => {
  clearInterval(clock)
  clearTimeout(toastTimer)
  window.removeEventListener('keydown', onKey)
})
</script>

<style scoped>
.fm {
  --navy: #123c4a;
  --navy-dark: #0d303b;
  --teal: #168b86;
  --teal-soft: #e9f7f5;
  --blue: #3267a8;
  --blue-soft: #edf4fc;
  --orange: #b9741d;
  --orange-soft: #fff5e8;
  --green: #168451;
  --green-soft: #eaf8f0;
  --red: #d94b4b;
  --red-soft: #fff0f0;
  --text: #20343b;
  --muted: #7b8c92;
  --line: #e7edef;
  --surface: #fff;
  --bg: #f5f8f9;

  box-sizing: border-box;
  min-height: 100vh;
  padding: 28px clamp(16px, 3vw, 40px) 56px;
  background: var(--bg);
  color: var(--text);
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, 'Segoe UI', sans-serif;
  font-size: 13px;
}
.fm *,
.fm *::before,
.fm *::after {
  box-sizing: border-box;
}
.fm h1,
.fm h2,
.fm h3,
.fm p {
  margin: 0;
}
.fm button {
  font-family: inherit;
}
.fm button:focus-visible,
.fm input:focus-visible,
.fm select:focus-visible {
  outline: 2px solid var(--teal);
  outline-offset: 2px;
}

/* ---------- Header ---------- */
.fm-head {
  display: flex;
  justify-content: space-between;
  align-items: flex-end;
  gap: 20px;
  margin-bottom: 18px;
}
.fm-crumb {
  color: var(--muted);
  font-size: 11.5px;
  margin-bottom: 6px;
}
.fm-head h1 {
  color: var(--navy-dark);
  font-size: 26px;
  letter-spacing: -0.5px;
}
.fm-desc {
  margin-top: 6px;
  max-width: 620px;
  color: var(--muted);
  line-height: 1.55;
}
.fm-year {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  padding: 8px 14px;
  border: 1px solid #dcebe8;
  border-radius: 10px;
  background: #f8fcfb;
  color: #527078;
  font-size: 11px;
}
.fm-year strong {
  color: var(--navy);
  font-size: 20px;
  line-height: 1.2;
}

/* ---------- Alur ---------- */
.fm-flow {
  display: flex;
  flex-wrap: wrap;
  gap: 8px 22px;
  margin: 0 0 18px;
  padding: 0;
  list-style: none;
  color: #527078;
  font-size: 12px;
}
.fm-flow li {
  display: flex;
  align-items: center;
  gap: 8px;
}
.fm-flow b {
  display: inline-flex;
  width: 20px;
  height: 20px;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: var(--navy);
  color: #fff;
  font-size: 11px;
}
.fm-flow small {
  color: var(--muted);
}

/* ---------- Overview ---------- */
.fm-overview {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 12px;
  margin-bottom: 18px;
}
.fm-ov {
  padding: 14px 16px;
  border: 1px solid var(--line);
  border-radius: 12px;
  background: var(--surface);
}
.fm-ov strong {
  display: block;
  color: var(--navy);
  font-size: 26px;
  line-height: 1.1;
  font-weight: 800;
}
.fm-ov strong small {
  color: var(--muted);
  font-size: 14px;
  font-weight: 600;
}
.fm-ov span {
  display: block;
  margin-top: 5px;
  color: var(--muted);
  font-size: 12px;
}
.fm-ov--warn {
  border-color: #f3dcb9;
  background: var(--orange-soft);
}
.fm-ov--warn strong {
  color: var(--orange);
}

/* ---------- Panel & toolbar ---------- */
.fm-panel {
  border: 1px solid var(--line);
  border-radius: 14px;
  background: var(--surface);
  overflow: hidden;
}
.fm-panel__head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  padding: 18px 20px 4px;
}
.fm-panel__head h2 {
  color: var(--navy-dark);
  font-size: 16px;
}
.fm-panel__head p {
  margin-top: 4px;
  color: var(--muted);
  font-size: 12px;
  line-height: 1.5;
}
.fm-count {
  color: var(--muted);
  font-size: 12px;
  white-space: nowrap;
}

.fm-toolbar {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  gap: 10px;
  padding: 14px 20px;
}
.fm-toolbar--inner {
  padding: 0 0 12px;
}
.fm-search {
  display: flex;
  align-items: center;
  gap: 8px;
  min-width: 230px;
  padding: 0 11px;
  height: 34px;
  border: 1px solid #dfe7e8;
  border-radius: 9px;
  background: #fff;
  color: var(--muted);
}
.fm-search input {
  flex: 1;
  border: 0;
  outline: 0;
  background: transparent;
  color: var(--text);
  font: inherit;
}
.fm-select {
  height: 34px;
  padding: 0 10px;
  border: 1px solid #dfe7e8;
  border-radius: 9px;
  background: #fff;
  color: var(--text);
  font: inherit;
}
.fm-chips {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
}
.fm-chips--tight {
  margin-top: 8px;
}
.fm-chip {
  height: 30px;
  padding: 0 11px;
  border: 1px solid #dfe7e8;
  border-radius: 999px;
  background: #fff;
  color: #567078;
  font-size: 12px;
  font-weight: 600;
  cursor: pointer;
}
.fm-chip span {
  margin-left: 3px;
  color: var(--muted);
  font-weight: 700;
}
.fm-chip:hover {
  background: #f5f8f8;
}
.fm-chip.is-active {
  border-color: var(--navy);
  background: var(--navy);
  color: #fff;
}
.fm-chip.is-active span {
  color: #b9d3da;
}

/* ---------- Table ---------- */
.fm-table-scroll {
  overflow-x: auto;
}
.fm-table {
  width: 100%;
  border-collapse: collapse;
  min-width: 760px;
}
.fm-table--list {
  min-width: 940px;
}
.fm-table thead th {
  padding: 10px 14px;
  border-top: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
  background: #f7f9f9;
  color: #6b7f85;
  font-size: 11.5px;
  font-weight: 700;
  text-align: left;
  white-space: nowrap;
}
.fm-table tbody td {
  padding: 12px 14px;
  border-bottom: 1px solid #f0f3f3;
  vertical-align: middle;
}
.fm-table tbody tr:last-child td {
  border-bottom: 0;
}
.fm-row {
  cursor: pointer;
  transition: background 0.15s ease;
}
.fm-row:hover {
  background: #f8fbfb;
}
.fm-table tr.is-selected {
  background: #f0faf7;
}
.fm-name {
  color: var(--navy);
  font-weight: 700;
}
.fm-muted {
  color: var(--muted);
  font-size: 12px;
}
.fm-right {
  text-align: right !important;
}
.fm-check {
  width: 42px;
  text-align: center !important;
}
.fm-check input,
.fm-table input[type='checkbox'] {
  width: 15px;
  height: 15px;
  accent-color: var(--teal);
  cursor: pointer;
}
.fm-empty {
  padding: 34px 14px !important;
  text-align: center;
  color: var(--muted);
}
.fm-actions {
  white-space: nowrap;
}
.fm-actions .fm-btn + .fm-btn {
  margin-left: 6px;
}
.fm-badges {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}

.fm-meter {
  width: 130px;
  height: 7px;
  margin-bottom: 5px;
  border-radius: 6px;
  background: #edf2f2;
  overflow: hidden;
}
.fm-meter span {
  display: block;
  height: 100%;
  border-radius: 6px;
  background: var(--blue);
  transition: width 0.3s ease;
}
.fm-meter--teal span {
  background: var(--teal);
}

/* ---------- Badge ---------- */
.fm-badge {
  display: inline-flex;
  align-items: center;
  padding: 4px 9px;
  border-radius: 999px;
  font-size: 11.5px;
  font-weight: 700;
  white-space: nowrap;
}
.fm-badge--muted {
  background: #eef1f2;
  color: #5b6b6d;
}
.fm-badge--ok {
  background: var(--green-soft);
  color: var(--green);
}
.fm-badge--warn {
  background: var(--orange-soft);
  color: var(--orange);
}
.fm-badge--info {
  background: var(--blue-soft);
  color: var(--blue);
}
.fm-badge--danger {
  background: var(--red-soft);
  color: #c1512e;
}
.fm-badge--soft {
  font-weight: 600;
}

.fm-remain {
  margin-top: 2px;
  color: var(--muted);
  font-size: 12px;
}
.fm-remain.is-urgent {
  color: var(--red);
  font-weight: 700;
}

/* ---------- Button ---------- */
.fm-btn {
  height: 34px;
  padding: 0 14px;
  border: 1px solid transparent;
  border-radius: 9px;
  font-size: 12.5px;
  font-weight: 700;
  cursor: pointer;
  transition: background 0.15s ease, border-color 0.15s ease;
}
.fm-btn--sm {
  height: 30px;
  padding: 0 10px;
  font-size: 12px;
  border-radius: 8px;
}
.fm-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
.fm-btn--primary {
  background: var(--teal);
  color: #fff;
}
.fm-btn--primary:hover:not(:disabled) {
  background: #117873;
}
.fm-btn--outline {
  border-color: #bcd6d4;
  background: #fff;
  color: #0f6f6b;
}
.fm-btn--outline:hover:not(:disabled) {
  background: var(--teal-soft);
}
.fm-btn--ghost {
  border-color: #dfe7e8;
  background: #fff;
  color: #567078;
}
.fm-btn--ghost:hover:not(:disabled) {
  background: #f5f8f8;
}
.fm-btn--danger {
  background: var(--red);
  color: #fff;
}
.fm-btn--danger:hover:not(:disabled) {
  background: #bd3a3a;
}
.fm-btn--danger-outline {
  border-color: #f2cfcf;
  background: #fff;
  color: #c23f3f;
}
.fm-btn--danger-outline:hover:not(:disabled) {
  background: var(--red-soft);
}
.fm-link {
  align-self: flex-start;
  padding: 4px 0;
  border: 0;
  background: none;
  color: var(--teal);
  font-size: 12px;
  font-weight: 700;
  cursor: pointer;
}
.fm-link:hover {
  text-decoration: underline;
}

/* ---------- Drawer ---------- */
.fm-overlay {
  position: fixed;
  inset: 0;
  z-index: 40;
  display: flex;
  justify-content: flex-end;
  background: rgba(13, 48, 59, 0.42);
}
.fm-drawer {
  display: flex;
  flex-direction: column;
  width: min(1060px, 100%);
  height: 100%;
  background: var(--bg);
  box-shadow: -18px 0 50px rgba(13, 48, 59, 0.22);
}
.fm-drawer__head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  padding: 20px 26px 14px;
  background: var(--surface);
}
.fm-drawer__head h2 {
  color: var(--navy-dark);
  font-size: 20px;
  letter-spacing: -0.3px;
}

.fm-kpis {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
  padding: 0 26px 16px;
  background: var(--surface);
}
.fm-kpi {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 11px 13px;
  border: 1px solid var(--line);
  border-radius: 11px;
  background: #fbfcfc;
  text-align: left;
  cursor: pointer;
  transition: border-color 0.15s ease, background 0.15s ease;
}
.fm-kpi:hover:not(:disabled) {
  border-color: #b7d3d1;
  background: #fff;
}
.fm-kpi:disabled {
  cursor: default;
  opacity: 0.7;
}
.fm-kpi span {
  color: var(--muted);
  font-size: 11.5px;
}
.fm-kpi strong {
  color: var(--navy);
  font-size: 22px;
  line-height: 1.2;
  font-weight: 800;
}
.fm-kpi strong small {
  color: var(--muted);
  font-size: 13px;
  font-weight: 600;
}
.fm-kpi em {
  color: var(--muted);
  font-size: 11.5px;
  font-style: normal;
}
.fm-kpi--warn strong {
  color: var(--orange);
}

.fm-alert {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 14px;
  margin: 0 0 14px;
  padding: 11px 14px;
  border-radius: 10px;
  font-size: 12.5px;
  line-height: 1.5;
}
.fm-alert ul {
  margin: 4px 0 0;
  padding-left: 18px;
}
.fm-alert__actions {
  display: flex;
  gap: 8px;
  flex-shrink: 0;
}
.fm-alert--warn {
  background: #fdf3e7;
  color: #8f5f1a;
}
.fm-alert--info {
  margin: 0;
  border-radius: 0;
  border-top: 1px solid #cfe1f3;
  background: var(--blue-soft);
  color: #2b5688;
  padding: 11px 26px;
}
.fm-alert--danger {
  background: var(--red-soft);
  color: #a62f2f;
}

.fm-tabs {
  display: flex;
  gap: 4px;
  padding: 0 26px;
  border-bottom: 1px solid var(--line);
  background: var(--surface);
}
.fm-tab {
  position: relative;
  padding: 12px 14px;
  border: 0;
  border-bottom: 2px solid transparent;
  background: none;
  color: #6b7f85;
  font-size: 13px;
  font-weight: 700;
  cursor: pointer;
}
.fm-tab:hover:not(:disabled) {
  color: var(--navy);
}
.fm-tab.is-active {
  border-bottom-color: var(--teal);
  color: var(--navy);
}
.fm-tab:disabled {
  opacity: 0.4;
  cursor: not-allowed;
}
.fm-dot {
  display: inline-block;
  min-width: 18px;
  margin-left: 6px;
  padding: 1px 6px;
  border-radius: 999px;
  background: var(--orange-soft);
  color: var(--orange);
  font-size: 11px;
  font-style: normal;
  text-align: center;
}

.fm-drawer__body {
  flex: 1;
  overflow-y: auto;
  padding: 20px 26px 40px;
}
.fm-drawer__body .fm-table-scroll {
  border: 1px solid var(--line);
  border-radius: 12px;
  background: var(--surface);
}
.fm-drawer__body .fm-table thead th:first-child {
  border-top-left-radius: 12px;
}
.fm-intro {
  margin-bottom: 14px;
  max-width: 760px;
  color: #567078;
  line-height: 1.6;
}

.fm-bulk {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;
  gap: 10px;
  margin-bottom: 12px;
  padding: 9px 12px;
  border: 1px solid #cfe6e3;
  border-radius: 10px;
  background: var(--teal-soft);
  color: var(--navy);
}
.fm-bulk__actions {
  display: flex;
  flex-wrap: wrap;
  gap: 7px;
}

/* ---------- Switch prognosa ---------- */
.fm-switch {
  position: relative;
  width: 42px;
  height: 24px;
  padding: 0;
  border: 0;
  border-radius: 999px;
  background: #cfd8da;
  cursor: pointer;
  transition: background 0.2s ease;
}
.fm-switch span {
  position: absolute;
  top: 3px;
  left: 3px;
  width: 18px;
  height: 18px;
  border-radius: 50%;
  background: #fff;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.25);
  transition: transform 0.2s ease;
}
.fm-switch.is-on {
  background: var(--green);
}
.fm-switch.is-on span {
  transform: translateX(18px);
}

/* ---------- Layout tab ---------- */
.fm-layout-head {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 16px;
  margin-bottom: 14px;
}
.fm-layout-head h3 {
  color: var(--navy);
  font-size: 15px;
}
.fm-layout-head p {
  margin-top: 4px;
  max-width: 640px;
  color: var(--muted);
  line-height: 1.55;
}
.fm-lock {
  display: flex;
  align-items: center;
  gap: 14px;
  margin-bottom: 14px;
  padding: 14px 16px;
  border: 1px solid #f3dcb9;
  border-radius: 12px;
  background: var(--orange-soft);
  color: #8f5f1a;
}
.fm-lock svg {
  flex-shrink: 0;
}
.fm-lock div {
  flex: 1;
}
.fm-lock strong {
  display: block;
  margin-bottom: 2px;
}
.fm-lock p {
  font-size: 12.5px;
  line-height: 1.55;
}

.fm-table--layout {
  min-width: 0;
}
.fm-no {
  width: 56px;
  color: var(--muted);
  text-align: center !important;
}
.fm-lr--label td {
  background: #eef4f4;
  color: var(--navy-dark);
  font-weight: 800;
}
.fm-lr--sub td {
  background: #f8fbfb;
  color: #3d5860;
  font-weight: 700;
  padding-left: 26px !important;
}
.fm-lr--uraian td.is-indent {
  padding-left: 42px !important;
}

/* ---------- Editor ---------- */
.fm-editor {
  display: flex;
  flex-direction: column;
  gap: 14px;
}
.fm-lb {
  display: flex;
  flex-direction: column;
  gap: 10px;
  padding: 14px;
  border: 1px solid var(--line);
  border-radius: 12px;
  background: var(--surface);
}
.fm-lb__head,
.fm-sb__head,
.fm-ur {
  display: flex;
  align-items: center;
  gap: 8px;
}
.fm-sb {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-left: 18px;
  padding: 10px 12px;
  border-left: 3px solid #cfe6e3;
  border-radius: 0 10px 10px 0;
  background: #f8fbfb;
}
.fm-ur {
  margin-left: 18px;
}
.fm-ur__no {
  width: 22px;
  color: var(--muted);
  font-size: 12px;
  text-align: center;
}
.fm-tag {
  flex: 0 0 68px;
  padding: 4px 0;
  border-radius: 6px;
  font-size: 11px;
  font-weight: 800;
  text-align: center;
}
.fm-tag--label {
  background: var(--blue-soft);
  color: var(--blue);
}
.fm-tag--sub {
  background: var(--teal-soft);
  color: var(--teal);
}
.fm-input {
  flex: 1;
  min-width: 0;
  height: 34px;
  padding: 0 11px;
  border: 1px solid #dfe7e8;
  border-radius: 8px;
  background: #fff;
  color: var(--text);
  font: inherit;
}
.fm-input:focus {
  border-color: var(--teal);
  outline: 0;
  box-shadow: 0 0 0 3px rgba(22, 139, 134, 0.12);
}
.fm-input.is-invalid {
  border-color: var(--red);
  box-shadow: 0 0 0 3px rgba(217, 75, 75, 0.1);
}
.fm-mini {
  display: flex;
  gap: 3px;
}
.fm-mini button {
  width: 28px;
  height: 28px;
  border: 1px solid #e3eaeb;
  border-radius: 7px;
  background: #fff;
  color: #6b7f85;
  font-size: 10px;
  cursor: pointer;
}
.fm-mini button:hover:not(:disabled) {
  background: #f1f5f6;
}
.fm-mini button:last-child:hover:not(:disabled) {
  border-color: #f2cfcf;
  background: var(--red-soft);
  color: var(--red);
}
.fm-mini button:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}
.fm-editor__foot {
  position: sticky;
  bottom: -40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  margin-top: 6px;
  padding: 12px 14px;
  border: 1px solid var(--line);
  border-radius: 12px;
  background: var(--surface);
  box-shadow: 0 -8px 22px rgba(24, 59, 68, 0.06);
}
.fm-editor__foot > div {
  display: flex;
  gap: 8px;
}

/* ---------- Modal ---------- */
.fm-modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 60;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: rgba(13, 48, 59, 0.5);
}
.fm-modal-overlay--top {
  z-index: 70;
}
.fm-modal {
  width: min(460px, 100%);
  max-height: 88vh;
  overflow-y: auto;
  padding: 22px;
  border-radius: 14px;
  background: var(--surface);
  box-shadow: 0 24px 60px rgba(13, 48, 59, 0.3);
}
.fm-modal h3 {
  color: var(--navy-dark);
  font-size: 16px;
}
.fm-modal > .fm-muted {
  margin-top: 3px;
}
.fm-confirm-msg {
  margin-top: 8px;
  color: #4a6067;
  line-height: 1.6;
}
.fm-list {
  max-height: 130px;
  margin: 12px 0 0;
  padding: 9px 12px 9px 28px;
  overflow-y: auto;
  border: 1px solid var(--line);
  border-radius: 9px;
  background: #fafcfc;
  color: #42565d;
  line-height: 1.7;
}
.fm-field {
  margin-top: 16px;
}
.fm-label {
  display: block;
  margin-bottom: 7px;
  color: var(--navy);
  font-weight: 700;
}
.fm-row2 {
  display: grid;
  grid-template-columns: 1fr 130px;
  gap: 8px;
  margin-top: 8px;
}
.fm-row2:first-of-type {
  margin-top: 0;
}
.fm-seg {
  display: inline-flex;
  padding: 3px;
  border-radius: 10px;
  background: #eef2f3;
}
.fm-seg button {
  height: 30px;
  padding: 0 14px;
  border: 0;
  border-radius: 8px;
  background: transparent;
  color: #567078;
  font-size: 12.5px;
  font-weight: 700;
  cursor: pointer;
}
.fm-seg button.is-active {
  background: #fff;
  color: var(--navy);
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.12);
}
.fm-error {
  margin-top: 12px;
  color: var(--red);
  font-size: 12.5px;
}
.fm-modal__foot {
  display: flex;
  justify-content: flex-end;
  gap: 9px;
  margin-top: 20px;
}

/* ---------- Toast ---------- */
.fm-toast {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 80;
  max-width: 420px;
  padding: 12px 16px;
  border-radius: 10px;
  font-size: 12.5px;
  font-weight: 700;
  box-shadow: 0 12px 35px rgba(20, 50, 58, 0.18);
}
.fm-toast--success {
  border: 1px solid #d5f0e1;
  background: #eefaf4;
  color: #176e47;
}
.fm-toast--warn {
  border: 1px solid #f3dcb9;
  background: #fff8ee;
  color: #8f5f1a;
}

/* ---------- Transisi ---------- */
.fm-fade-enter-active,
.fm-fade-leave-active {
  transition: opacity 0.18s ease;
}
.fm-fade-enter-from,
.fm-fade-leave-to {
  opacity: 0;
}
.fm-drawer-fx-enter-active,
.fm-drawer-fx-leave-active {
  transition: background 0.22s ease;
}
.fm-drawer-fx-enter-active .fm-drawer,
.fm-drawer-fx-leave-active .fm-drawer {
  transition: transform 0.22s ease;
}
.fm-drawer-fx-enter-from,
.fm-drawer-fx-leave-to {
  background: rgba(13, 48, 59, 0);
}
.fm-drawer-fx-enter-from .fm-drawer,
.fm-drawer-fx-leave-to .fm-drawer {
  transform: translateX(40px);
}

@media (prefers-reduced-motion: reduce) {
  .fm *,
  .fm-drawer-fx-enter-active .fm-drawer,
  .fm-drawer-fx-leave-active .fm-drawer {
    transition: none !important;
  }
}

/* ---------- Responsive ---------- */
@media (max-width: 960px) {
  .fm-overview,
  .fm-kpis {
    grid-template-columns: repeat(2, 1fr);
  }
}
@media (max-width: 640px) {
  .fm-head,
  .fm-panel__head,
  .fm-layout-head,
  .fm-alert,
  .fm-lock {
    flex-direction: column;
    align-items: flex-start;
  }
  .fm-year {
    align-items: flex-start;
  }
  .fm-drawer__head,
  .fm-kpis,
  .fm-drawer__body,
  .fm-tabs {
    padding-left: 16px;
    padding-right: 16px;
  }
  .fm-tab {
    padding: 12px 9px;
    font-size: 12px;
  }
  .fm-search {
    flex: 1 1 100%;
  }
  .fm-tag {
    flex-basis: 54px;
  }
}
</style>