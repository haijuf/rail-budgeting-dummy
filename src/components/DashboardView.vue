<template>
  <div class="dash">
    <!-- Header -->
    <header class="topbar">
      <div class="topbar__left">
        <button class="icon-btn icon-btn--menu" aria-label="Menu">
          <svg width="20" height="20" viewBox="0 0 24 24" fill="none">
            <rect x="3" y="4" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.7"/>
            <rect x="14" y="4" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.7"/>
            <rect x="3" y="13" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.7"/>
            <rect x="14" y="13" width="7" height="7" rx="1.5" stroke="currentColor" stroke-width="1.7"/>
          </svg>
        </button>

        <div>
          <div class="topbar__title">Dashboard SU</div>
          <div class="topbar__subtitle">Monitoring RKAP & Prognosa</div>
        </div>
      </div>

      <div class="topbar__right">
        <div class="user__meta">
          <span class="user__name">{{ user.name }}</span>
          <span class="user__role">{{ user.role }}</span>
        </div>
        <div class="user__avatar">{{ userInitials }}</div>
      </div>
    </header>

    <main class="content">
      <!-- Page intro -->
      <section class="page-intro">
        <div>
          <div class="breadcrumb">
            <span>Dashboard</span>
            <span class="breadcrumb__dot">/</span>
            <strong>Super User</strong>
          </div>
          <h1>Pemantauan Anggaran</h1>
          <p>Pantau progres RKAP dan Prognosa yang sedang berjalan serta lakukan approval dengan lebih cepat.</p>
        </div>

        <div class="live-status">
          <span class="live-status__dot"></span>
          Data berjalan
          <span class="live-status__time">{{ currentTime }}</span>
        </div>
      </section>

      <!-- Statistics -->
      <section class="stats-wrap">
        <div v-for="group in statGroups" :key="group.key" class="stats-group">
          <div class="stats-group__title">
            <span class="stats-group__badge" :class="`stats-group__badge--${group.key}`">{{ group.label }}</span>
            <span>Status form {{ group.label }}</span>
          </div>

          <div class="stats-grid">
            <article
              v-for="card in group.cards"
              :key="card.key"
              class="stat-card"
              :class="`stat-card--${card.tone}`"
            >
              <div class="stat-card__label">{{ card.label }}</div>
              <div class="stat-card__value">{{ card.value }}</div>

              <div class="stat-card__icon">
                <!-- Total -->
                <svg v-if="card.icon === 'doc'" width="18" height="18" viewBox="0 0 24 24" fill="none">
                  <path d="M6 3h9l4 4v14H6a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2Z" stroke="currentColor" stroke-width="1.8"/>
                  <path d="M14 3v5h5" stroke="currentColor" stroke-width="1.8"/>
                </svg>
                <!-- Drafting -->
                <svg v-else-if="card.icon === 'edit'" width="18" height="18" viewBox="0 0 24 24" fill="none">
                  <path d="M6 3h9l4 4v6M6 3a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h6" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                  <path d="M14 21l1-4 6-6 3 3-6 6-4 1Z" stroke="currentColor" stroke-width="1.6" stroke-linejoin="round"/>
                </svg>
                <!-- Approval -->
                <svg v-else-if="card.icon === 'clock'" width="18" height="18" viewBox="0 0 24 24" fill="none">
                  <path d="M6 3h9l4 4v4M6 3a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h4" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                  <circle cx="16.5" cy="17" r="4.5" stroke="currentColor" stroke-width="1.8"/>
                  <path d="M16.5 14.8V17l1.5 1" stroke="currentColor" stroke-width="1.6" stroke-linecap="round"/>
                </svg>
                <!-- Terkirim -->
                <svg v-else width="18" height="18" viewBox="0 0 24 24" fill="none">
                  <path d="M6 3h9l4 4v14H6a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2Z" stroke="currentColor" stroke-width="1.8"/>
                  <path d="M8.5 13.5l2.5 2.5 4.5-5" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                </svg>
              </div>
            </article>
          </div>
        </div>
      </section>

            <!-- Progress -->
      <section class="panel progress-panel">
        <div class="panel__heading">
          <div>
            <div class="eyebrow">MONITORING</div>
            <h2 class="panel__title">Progress Pengisian per Form</h2>
            <p class="panel__description">Progres pengisian setiap form terhadap jumlah unit yang wajib mengisi.</p>
          </div>

          <div class="legend">
            <span><i class="legend__dot legend__dot--rkap"></i> RKAP</span>
            <span><i class="legend__dot legend__dot--prognosa"></i> Prognosa</span>
          </div>
        </div>

        <div class="progress-summary">
          <div v-for="group in progressGroups" :key="group.key" class="progress-card">
            <!-- Ringkasan keseluruhan -->
            <div class="progress-card__header">
              <div class="progress-card__title">
                <span class="progress-card__badge" :class="`progress-card__badge--${group.key}`">
                  {{ group.shortLabel }}
                </span>
                <div>
                  <strong>{{ group.label }}</strong>
                  <small>{{ group.filled }} dari {{ group.total }} isian masuk · {{ group.items.length }} form</small>
                </div>
              </div>
              <strong class="progress-card__percentage">{{ formatPct(group.pct) }}</strong>
            </div>

            <div class="progress-track">
              <div
                class="progress-fill"
                :class="`progress-fill--${group.key}`"
                :style="{ width: `${Math.min(group.pct, 100)}%` }"
              ></div>
            </div>

            <!-- Per form -->
            <ul class="form-progress-list">
              <li v-for="f in group.items" :key="f.key" class="form-progress">
                <div class="form-progress__top">
                  <span class="form-progress__name" :title="f.nama">{{ f.nama }}</span>
                  <span class="form-progress__meta">
                    {{ f.filled }}/{{ f.total }}
                    <strong :class="`tone--${f.tone}`">{{ formatPct(f.pct) }}</strong>
                  </span>
                </div>
                <div class="progress-track progress-track--sm">
                  <div
                    class="progress-fill"
                    :class="[`progress-fill--${group.key}`, { 'progress-fill--done': f.pct >= 100 }]"
                    :style="{ width: `${Math.min(f.pct, 100)}%` }"
                  ></div>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </section>

      <!-- Approval -->
      <section class="panel approval-panel">
        <div class="approval-heading">
          <div>
            <div class="eyebrow">ACTION CENTER</div>
            <h2 class="panel__title">Form Approval</h2>
            <p class="panel__description">
              Form yang sudah <strong>Approve by VP</strong> dan membutuhkan tindak lanjut Super User.
            </p>
          </div>

          <div class="approval-summary">
            <div class="approval-summary__item">
              <span class="approval-summary__number">{{ approvalForms.length }}</span>
              <span>Menunggu</span>
            </div>
            <div class="approval-summary__divider"></div>
            <div class="approval-summary__item approval-summary__item--urgent">
              <span class="approval-summary__number">{{ urgentCount }}</span>
              <span>≤ 1 hari</span>
            </div>
          </div>
        </div>

        <!-- RKAP table -->
        <div class="table-section">
          <div class="table-section__heading">
            <div class="table-title">
              <span class="table-title__icon table-title__icon--rkap">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                  <path d="M6 3h9l4 4v14H6a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2Z" stroke="currentColor" stroke-width="1.7"/>
                  <path d="M14 3v5h5" stroke="currentColor" stroke-width="1.7"/>
                </svg>
              </span>
              <div>
                <strong>RKAP</strong>
                <span>{{ rkapApprovalForms.length }} form siap diproses</span>
              </div>
            </div>

          <div class="table-actions">
            <span v-if="selectedRkap.size" class="selected-count">{{ selectedRkap.size }} dipilih</span>
            <button
              class="btn btn--reject"
              :disabled="selectedRkap.size === 0"
              @click="openReject('rkap')"
            >
              ✕ Reject Terpilih
            </button>
            <button
              class="btn btn--approve"
              :disabled="selectedRkap.size === 0"
              @click="bulkApprove('rkap')"
            >
              ✓ Approve Terpilih
            </button>
          </div>
        </div>

          <div class="table-scroll">
            <table class="table">
              <thead>
                <tr>
                  <th class="table__checkbox-col">
                    <input
                      type="checkbox"
                      :checked="allRkapSelected"
                      :disabled="rkapApprovalForms.length === 0"
                      @change="toggleSelectAll('rkap', $event)"
                    />
                  </th>
                  <th>No</th>
                  <th>Nama Form</th>
                  <th>Divisi / Area</th>
                  <th>Approve VP</th>
                  <th>Penutupan RKAP</th>
                  <th>Sisa Waktu</th>
                  <th class="table__action-col">Action</th>
                </tr>
              </thead>
              <tbody>
                <tr v-if="rkapApprovalForms.length === 0">
                  <td class="table__empty" colspan="8">
                    <div class="empty-state">
                      <span>✓</span>
                      <strong>Tidak ada form RKAP</strong>
                      <small>Semua form sudah diproses.</small>
                    </div>
                  </td>
                </tr>

                <tr
                  v-for="(form, idx) in rkapApprovalForms"
                  :key="form.id"
                  :class="{ 'row--selected': isSelected('rkap', form.id) }"
                >
                  <td class="table__checkbox-col">
                    <input type="checkbox" :checked="isSelected('rkap', form.id)" @change="toggleRow('rkap', form.id)" />
                  </td>
                  <td class="table__number">{{ idx + 1 }}</td>
                  <td>
                    <div class="form-name">
                      <strong>{{ form.namaForm }}</strong>
                      <span>{{ form.kode }}</span>
                    </div>
                  </td>
                  <td>
                    <div class="cell-stack">
                      <strong>{{ form.divisi }}</strong>
                      <span>{{ form.lokasi }}</span>
                    </div>
                  </td>
                  <td><span class="vp-badge">✓ Approved VP</span></td>
                  <td>
                    <div class="date-stack">
                      <strong>{{ form.closingDate }}</strong>
                      <span>{{ form.closingLabel }}</span>
                    </div>
                  </td>
                  <td>
                    <span class="deadline" :class="{ 'deadline--urgent': form.sisaWaktuHari <= 1 }">
                      {{ form.sisaWaktu }}
                    </span>
                  </td>
                  <td class="table__action">
                    <button class="action-btn action-btn--approve" @click="approveForm(form)" title="Approve">
                      ✓ <span>Approve</span>
                    </button>
                    <button class="action-btn action-btn--reject" @click="openReject(form.type, form)" title="Reject">
                      ✕ <span>Reject</span>
                    </button>
                    <button class="action-btn action-btn--view" @click="$emit('view', form)" title="Lihat detail">
                      Detail
                    </button>
                </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <!-- Prognosa table -->
        <div class="table-section table-section--last">
          <div class="table-section__heading">
            <div class="table-title">
              <span class="table-title__icon table-title__icon--prognosa">
                <svg width="16" height="16" viewBox="0 0 24 24" fill="none">
                  <path d="M4 17l5-5 4 3 7-8" stroke="currentColor" stroke-width="1.8" stroke-linecap="round" stroke-linejoin="round"/>
                  <path d="M16 7h4v4" stroke="currentColor" stroke-width="1.8" stroke-linecap="round"/>
                </svg>
              </span>
              <div>
                <strong>Prognosa</strong>
                <span>{{ prognosaApprovalForms.length }} form siap diproses</span>
              </div>
            </div>

            <div class="table-actions">
              <span v-if="selectedPrognosa.size" class="selected-count">{{ selectedPrognosa.size }} dipilih</span>
              <button
                class="btn btn--reject"
                :disabled="selectedPrognosa.size === 0"
                @click="openReject('prognosa')"
              >
                ✕ Reject Terpilih
              </button>
              <button
                class="btn btn--approve"
                :disabled="selectedPrognosa.size === 0"
                @click="bulkApprove('prognosa')"  
              >
                ✓ Approve Terpilih
              </button>
            </div>
          </div>

          <div class="table-scroll">
            <table class="table">
              <thead>
                <tr>
                  <th class="table__checkbox-col">
                    <input
                      type="checkbox"
                      :checked="allPrognosaSelected"
                      :disabled="prognosaApprovalForms.length === 0"
                      @change="toggleSelectAll('prognosa', $event)"
                    />
                  </th>
                  <th>No</th>
                  <th>Nama Form</th>
                  <th>Divisi / Area</th>
                  <th>Approve VP</th>
                  <th>Penutupan Prognosa</th>
                  <th>Sisa Waktu</th>
                  <th class="table__action-col">Action</th>
                </tr>
              </thead>
              <tbody>
                <tr v-if="prognosaApprovalForms.length === 0">
                  <td class="table__empty" colspan="8">
                    <div class="empty-state">
                      <span>✓</span>
                      <strong>Tidak ada form Prognosa</strong>
                      <small>Semua form sudah diproses.</small>
                    </div>
                  </td>
                </tr>

                <tr
                  v-for="(form, idx) in prognosaApprovalForms"
                  :key="form.id"
                  :class="{ 'row--selected': isSelected('prognosa', form.id) }"
                >
                  <td class="table__checkbox-col">
                    <input type="checkbox" :checked="isSelected('prognosa', form.id)" @change="toggleRow('prognosa', form.id)" />
                  </td>
                  <td class="table__number">{{ idx + 1 }}</td>
                  <td>
                    <div class="form-name">
                      <strong>{{ form.namaForm }}</strong>
                      <span>{{ form.kode }}</span>
                    </div>
                  </td>
                  <td>
                    <div class="cell-stack">
                      <strong>{{ form.divisi }}</strong>
                      <span>{{ form.lokasi }}</span>
                    </div>
                  </td>
                  <td><span class="vp-badge">✓ Approved VP</span></td>
                  <td>
                    <div class="date-stack">
                      <strong>{{ form.closingDate }}</strong>
                      <span>{{ form.closingLabel }}</span>
                    </div>
                  </td>
                  <td>
                    <span class="deadline" :class="{ 'deadline--urgent': form.sisaWaktuHari <= 1 }">
                      {{ form.sisaWaktu }}
                    </span>
                  </td>
                  <td class="table__action">
                    <button class="action-btn action-btn--approve" @click="approveForm(form)" title="Approve">
                      ✓ <span>Approve</span>
                    </button>
                    <button class="action-btn action-btn--reject" @click="openReject(form.type, form)" title="Reject">
                      ✕ <span>Reject</span>
                    </button>
                    <button class="action-btn action-btn--view" @click="$emit('view', form)" title="Lihat detail">
                      Detail
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>

        <footer class="table-footer">
          <span>Data diurutkan berdasarkan waktu penutupan terdekat</span>
          <div class="table-footer__pagination">
            <span>{{ totalApprovalCount }} form ditampilkan</span>
          </div>
        </footer>
      </section>
    </main>

    <transition name="fade">
      <div v-if="rejectModal.open" class="modal-overlay" @click.self="closeReject">
        <div class="modal" role="dialog" aria-modal="true" aria-labelledby="reject-title">
          <div class="modal__header">
            <div class="modal__icon">✕</div>
            <div>
              <h3 id="reject-title" class="modal__title">Reject Form</h3>
              <p class="modal__subtitle">
                {{ rejectModal.targets.length === 1
                  ? rejectModal.targets[0].namaForm
                  : `${rejectModal.targets.length} form ${rejectModal.type === 'rkap' ? 'RKAP' : 'Prognosa'} dipilih` }}
              </p>
            </div>
          </div>

          <ul v-if="rejectModal.targets.length > 1" class="modal__list">
            <li v-for="t in rejectModal.targets" :key="t.id">
              {{ t.namaForm }} <small>· {{ t.divisi }}</small>
            </li>
          </ul>

          <label class="modal__label" for="reject-note">
            Catatan <span>*wajib diisi</span>
          </label>
          <textarea
            id="reject-note"
            ref="noteInput"
            v-model="rejectModal.note"
            class="modal__textarea"
            :class="{ 'modal__textarea--error': rejectModal.error }"
            rows="4"
            placeholder="Tuliskan alasan reject / hal yang perlu diperbaiki..."
            @input="rejectModal.error = ''"
          ></textarea>
          <small v-if="rejectModal.error" class="modal__error">{{ rejectModal.error }}</small>

          <div class="modal__footer">
            <button class="btn btn--ghost" @click="closeReject">Batal</button>
            <button class="btn btn--reject-solid" @click="confirmReject">Kirim Reject</button>
          </div>
        </div>
      </div>
    </transition>
      </div>
    </template>bulkApprove

<script setup>
import { computed, ref, nextTick } from 'vue'

const emit = defineEmits(['view', 'approve', 'reject', 'approve-bulk'])

const user = {
  name: 'NANDA',
  role: 'Super User · Anggaran',
}

const userInitials = computed(() =>
  user.name.split(' ').map(w => w[0]).join('').slice(0, 2)
)

const currentTime = new Intl.DateTimeFormat('id-ID', {
  day: '2-digit',
  month: 'short',
  hour: '2-digit',
  minute: '2-digit',
}).format(new Date())

/*
 * Demo data.
 * Ganti array ini dengan response API pada implementasi production.
 *
 * status:
 * - open
 * - publish
 * - approval_su
 * - approved
 *
 * type:
 * - rkap
 * - prognosa
 */
const forms = ref([
  {
    id: 1,
    kode: 'RKAP-2027-001',
    namaForm: 'Usulan Produksi Sarana',
    divisi: 'Angkutan Barang',
    lokasi: 'DAOP 1 JAKARTA',
    type: 'rkap',
    status: 'approval_su',
    closingDate: '30 Sep 2026',
    closingLabel: 'Rabu',
    closingSort: '2026-09-30',
    sisaWaktu: '2 hari',
    sisaWaktuHari: 2,
  },
  {
    id: 2,
    kode: 'RKAP-2027-002',
    namaForm: 'Usulan Prasarana Jembatan KAI',
    divisi: 'Prasarana',
    lokasi: 'DAOP 4 SEMARANG',
    type: 'rkap',
    status: 'approval_su',
    closingDate: '29 Sep 2026',
    closingLabel: 'Selasa',
    closingSort: '2026-09-29',
    sisaWaktu: '1 hari',
    sisaWaktuHari: 1,
  },
  {
    id: 3,
    kode: 'RKAP-2027-003',
    namaForm: 'Usulan Produksi Penumpang',
    divisi: 'Angkutan Penumpang',
    lokasi: 'DAOP 6 YOGYAKARTA',
    type: 'rkap',
    status: 'publish',
    closingDate: '03 Okt 2026',
    closingLabel: 'Sabtu',
    closingSort: '2026-10-03',
    sisaWaktu: '6 hari',
    sisaWaktuHari: 6,
  },
  {
    id: 4,
    kode: 'PROG-2026-014',
    namaForm: 'Prognosa Produksi Sarana',
    divisi: 'Sarana',
    lokasi: 'DAOP 1 JAKARTA',
    type: 'prognosa',
    status: 'approval_su',
    closingDate: '28 Sep 2026',
    closingLabel: 'Hari ini',
    closingSort: '2026-09-28',
    sisaWaktu: 'Hari ini',
    sisaWaktuHari: 0,
  },
  {
    id: 5,
    kode: 'PROG-2026-015',
    namaForm: 'Prognosa Prasarana',
    divisi: 'Prasarana',
    lokasi: 'DAOP 3 CIREBON',
    type: 'prognosa',
    status: 'approval_su',
    closingDate: '01 Okt 2026',
    closingLabel: 'Kamis',
    closingSort: '2026-10-01',
    sisaWaktu: '3 hari',
    sisaWaktuHari: 3,
  },
  {
    id: 6,
    kode: 'PROG-2026-016',
    namaForm: 'Prognosa Angkutan Penumpang',
    divisi: 'Angkutan Penumpang',
    lokasi: 'DAOP 6 YOGYAKARTA',
    type: 'prognosa',
    status: 'open',
    closingDate: '05 Okt 2026',
    closingLabel: 'Senin',
    closingSort: '2026-10-05',
    sisaWaktu: '7 hari',
    sisaWaktuHari: 7,
  },
])

const statusSummary = ref({
  rkap:     { drafting: 92, approval_manager: 1, approval_vp: 4, terkirim: 12 },
  prognosa: { drafting: 57, approval_manager: 1, approval_vp: 3, terkirim: 10 },
})

const statGroups = computed(() =>
  [
    { key: 'rkap', label: 'RKAP' },
    { key: 'prognosa', label: 'Prognosa' },
  ].map(group => {
    const s = statusSummary.value[group.key]
    const total = s.drafting + s.approval_manager + s.approval_vp + s.terkirim

    return {
      ...group,
      cards: [
        { key: 'total',    label: 'Total Form',       value: total,              tone: 'navy',   icon: 'doc' },
        { key: 'drafting', label: 'Drafting',         value: s.drafting,         tone: 'blue',   icon: 'edit' },
        { key: 'manager',  label: 'Approval Manager', value: s.approval_manager, tone: 'orange', icon: 'clock' },
        { key: 'vp',       label: 'Approval VP',      value: s.approval_vp,      tone: 'orange', icon: 'clock' },
        { key: 'sent',     label: 'Terkirim',         value: s.terkirim,         tone: 'green',  icon: 'check' },
      ],
    }
  })
)

const formProgress = ref([
  { key: 'hk-balai-yasa',        nama: 'Usulan Hari Kerja Balai Yasa',                   rkap: { filled: 8,  total: 8  }, prognosa: { filled: 7,  total: 8  } },
  { key: 'prod-sarana',          nama: 'Usulan Produksi Sarana',                         rkap: { filled: 12, total: 14 }, prognosa: { filled: 10, total: 14 } },
  { key: 'prasarana-sintel-pem', nama: 'Usulan Produksi Prasarana Sintel Pemerintah',    rkap: { filled: 6,  total: 9  }, prognosa: { filled: 4,  total: 9  } },
  { key: 'prasarana-rel-kai',    nama: 'Usulan Produksi Prasarana Rel KAI',              rkap: { filled: 9,  total: 9  }, prognosa: { filled: 8,  total: 9  } },
  { key: 'prasarana-jbt-kai',    nama: 'Usulan Produksi Prasarana Jembatan KAI',         rkap: { filled: 7,  total: 9  }, prognosa: { filled: 5,  total: 9  } },
  { key: 'asset-sintel',         nama: 'Usulan Produksi Asset Prasarana Sintel',         rkap: { filled: 5,  total: 9  }, prognosa: { filled: 3,  total: 9  } },
  { key: 'prod-penumpang',       nama: 'Usulan Produksi Penumpang',                      rkap: { filled: 10, total: 12 }, prognosa: { filled: 9,  total: 12 } },
  { key: 'prod-pelayanan',       nama: 'Usulan Produksi Pelayanan',                      rkap: { filled: 11, total: 12 }, prognosa: { filled: 6,  total: 12 } },
  { key: 'perawatan-sarana',     nama: 'Usulan Produksi Perawatan Sarana',               rkap: { filled: 13, total: 14 }, prognosa: { filled: 12, total: 14 } },
  { key: 'tanggungan-by',        nama: 'Usulan Tanggungan Balai Yasa',                   rkap: { filled: 4,  total: 8  }, prognosa: { filled: 2,  total: 8  } },
  { key: 'prasarana-rel-pem',    nama: 'Usulan Produksi Prasarana Rel Pemerintah',       rkap: { filled: 6,  total: 9  }, prognosa: { filled: 5,  total: 9  } },
  { key: 'prasarana-jbt-pem',    nama: 'Usulan Produksi Prasarana Jembatan Pemerintah',  rkap: { filled: 3,  total: 9  }, prognosa: { filled: 1,  total: 9  } },
  { key: 'asset-jalan-jbt',      nama: 'Usulan Produksi Asset Prasarana Jalan Jembatan', rkap: { filled: 5,  total: 9  }, prognosa: { filled: 4,  total: 9  } },
  { key: 'prod-sdm',             nama: 'Usulan Produksi Sumber Daya Manusia',            rkap: { filled: 14, total: 14 }, prognosa: { filled: 13, total: 14 } },
  { key: 'prod-barang',          nama: 'Usulan Produksi Barang',                         rkap: { filled: 9,  total: 12 }, prognosa: { filled: 7,  total: 12 } },
  { key: 'norma-kecelakaan',     nama: 'Usulan Norma Kendali Kecelakaan Kereta Api',     rkap: { filled: 2,  total: 6  }, prognosa: { filled: 1,  total: 6  } },
])

const pctOf = (filled, total) => (total ? (filled / total) * 100 : 0)

const toneOf = pct => (pct >= 100 ? 'done' : pct >= 60 ? 'good' : pct >= 30 ? 'warn' : 'low')

const progressGroups = computed(() =>
  [
    { key: 'rkap', label: 'RKAP', shortLabel: 'R' },
    { key: 'prognosa', label: 'Prognosa', shortLabel: 'P' },
  ].map(group => {
    const items = formProgress.value.map(f => {
      const { filled, total } = f[group.key]
      const pct = pctOf(filled, total)
      return { key: f.key, nama: f.nama, filled, total, pct, tone: toneOf(pct) }
    })
    const filled = items.reduce((sum, i) => sum + i.filled, 0)
    const total = items.reduce((sum, i) => sum + i.total, 0)
    return { ...group, items, filled, total, pct: pctOf(filled, total) }
  })
)

const approvalForms = computed(() =>
  forms.value
    .filter(f => f.status === 'approval_su')
    .sort((a, b) => new Date(a.closingSort) - new Date(b.closingSort))
)

const rkapApprovalForms = computed(() =>
  approvalForms.value.filter(f => f.type === 'rkap')
)

const prognosaApprovalForms = computed(() =>
  approvalForms.value.filter(f => f.type === 'prognosa')
)

const urgentCount = computed(() =>
  approvalForms.value.filter(f => f.sisaWaktuHari <= 1).length
)

const totalApprovalCount = computed(() => approvalForms.value.length)

const selectedRkap = ref(new Set())
const selectedPrognosa = ref(new Set())

function getSelection(type) {
  return type === 'rkap' ? selectedRkap : selectedPrognosa
}

function setSelection(type, value) {
  if (type === 'rkap') selectedRkap.value = value
  else selectedPrognosa.value = value
}

function isSelected(type, id) {
  return getSelection(type).value.has(id)
}

function toggleRow(type, id) {
  const next = new Set(getSelection(type).value)
  next.has(id) ? next.delete(id) : next.add(id)
  setSelection(type, next)
}

function toggleSelectAll(type, event) {
  const source = type === 'rkap' ? rkapApprovalForms.value : prognosaApprovalForms.value
  setSelection(type, event.target.checked ? new Set(source.map(f => f.id)) : new Set())
}

const allRkapSelected = computed(() =>
  rkapApprovalForms.value.length > 0 &&
  rkapApprovalForms.value.every(f => selectedRkap.value.has(f.id))
)

const allPrognosaSelected = computed(() =>
  prognosaApprovalForms.value.length > 0 &&
  prognosaApprovalForms.value.every(f => selectedPrognosa.value.has(f.id))
)

function deselect(type, id) {
  const next = new Set(getSelection(type).value)
  next.delete(id)
  setSelection(type, next)
}

function approveForm(form) {
  form.status = 'approved'
  deselect(form.type, form.id)
  showNotice(`"${form.namaForm}" berhasil di-approve oleh Super User.`, 'success')
  emit('approve', [form.id])
}

function bulkApprove(type) {
  const selection = getSelection(type).value
  if (!selection.size) return

  const targets = approvalForms.value.filter(
    f => f.type === type && selection.has(f.id)
  )

  targets.forEach(f => { f.status = 'approved' })
  setSelection(type, new Set())

  showNotice(`${targets.length} form ${type === 'rkap' ? 'RKAP' : 'Prognosa'} berhasil di-approve.`, 'success')
  emit('approve-bulk', targets.map(f => f.id))
}

/* ---------- Reject ---------- */
const rejectModal = ref({
  open: false,
  type: null,     // 'rkap' | 'prognosa'
  targets: [],    // form yang akan di-reject
  note: '',
  error: '',
})
const noteInput = ref(null)

// form = satu baris (opsional). Tanpa form = reject terpilih (bulk).
function openReject(type, form = null) {
  const targets = form
    ? [form]
    : approvalForms.value.filter(f => f.type === type && getSelection(type).value.has(f.id))

  if (!targets.length) return

  rejectModal.value = { open: true, type, targets, note: '', error: '' }
  nextTick(() => noteInput.value?.focus())
}

function closeReject() {
  rejectModal.value.open = false
}

function confirmReject() {
  const note = rejectModal.value.note.trim()
  if (!note) {
    rejectModal.value.error = 'Catatan wajib diisi sebelum reject.'
    return
  }

  const { type, targets } = rejectModal.value
  targets.forEach(f => {
    f.status = 'rejected'
    f.rejectNote = note
    deselect(type, f.id)
  })

  showNotice(
    targets.length === 1
      ? `"${targets[0].namaForm}" di-reject dengan catatan.`
      : `${targets.length} form ${type === 'rkap' ? 'RKAP' : 'Prognosa'} di-reject dengan catatan.`,
    'reject'
  )
  emit('reject', { ids: targets.map(f => f.id), note })
  closeReject()
}

const notice = ref(null)
let noticeTimer = null

function showNotice(msg, type = 'success') {
  notice.value = { msg, type }
  clearTimeout(noticeTimer)
  noticeTimer = setTimeout(() => {
    notice.value = null
  }, 3200)
}

function formatPct(value) {
  return `${Number(value.toFixed(0))}%`
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.dash {
  --navy: #123c4a;
  --navy-dark: #0d303b;
  --teal: #168b86;
  --teal-soft: #e9f7f5;
  --blue: #3267a8;
  --blue-soft: #edf4fc;
  --orange: #e69a42;
  --orange-soft: #fff5e8;
  --green: #168451;
  --green-soft: #eaf8f0;
  --red: #d94b4b;
  --red-soft: #fff0f0;
  --text: #20343b;
  --muted: #829197;
  --line: #e7edef;
  --surface: #ffffff;
  --background: #f5f8f9;

  min-height: 100vh;
  background:
    radial-gradient(circle at 90% 0%, rgba(22, 139, 134, .06), transparent 28%),
    var(--background);
  color: var(--text);
  font-family: Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
}

/* Header */
.topbar {
  min-height: 68px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 10px 36px;
  background: rgba(255, 255, 255, .94);
  border-bottom: 1px solid var(--line);
  position: sticky;
  top: 0;
  z-index: 20;
  backdrop-filter: blur(12px);
}

.topbar__left,
.topbar__right {
  display: flex;
  align-items: center;
}

.topbar__left {
  gap: 13px;
}

.topbar__title {
  color: var(--navy);
  font-size: 17px;
  font-weight: 800;
  line-height: 1.2;
}

.topbar__subtitle {
  margin-top: 2px;
  color: var(--muted);
  font-size: 10px;
}

.user__meta {
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  margin-right: 10px;
}

.user__name {
  color: var(--navy);
  font-size: 12px;
  font-weight: 800;
}

.user__role {
  color: var(--muted);
  font-size: 10px;
  margin-top: 2px;
}

.user__avatar {
  width: 37px;
  height: 37px;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: linear-gradient(145deg, #f0b35e, #e39432);
  color: #18363e;
  font-size: 11px;
  font-weight: 900;
  box-shadow: 0 5px 12px rgba(230, 154, 66, .22);
}

.icon-btn {
  border: 0;
  background: transparent;
  color: #60777e;
  cursor: pointer;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
}

.icon-btn--menu {
  width: 38px;
  height: 38px;
  background: #f1f5f6;
}

.icon-btn:hover {
  background: #eaf2f2;
}

/* Main */
.content {
  width: min(1480px, calc(100% - 64px));
  margin: 0 auto;
  padding: 28px 0 48px;
}

.page-intro {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: 20px;
  margin-bottom: 22px;
}

.breadcrumb {
  display: flex;
  gap: 8px;
  align-items: center;
  color: #96a3a8;
  font-size: 11px;
  margin-bottom: 8px;
}

.breadcrumb strong {
  color: var(--navy);
}

.breadcrumb__dot {
  color: #c2cbce;
}

.page-intro h1 {
  margin: 0;
  font-size: 25px;
  letter-spacing: -.5px;
  color: var(--navy-dark);
}

.page-intro p {
  margin: 6px 0 0;
  color: var(--muted);
  font-size: 12px;
}

.live-status {
  display: flex;
  align-items: center;
  gap: 7px;
  padding: 8px 11px;
  border: 1px solid #dcebe8;
  border-radius: 10px;
  background: #f8fcfb;
  color: #527078;
  font-size: 10px;
  white-space: nowrap;
}

.live-status__dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  background: #22a064;
  box-shadow: 0 0 0 4px rgba(34, 160, 100, .1);
}

.live-status__time {
  padding-left: 5px;
  color: #9aa7ab;
  border-left: 1px solid #dfe8e8;
}

/* Statistics */
.stats-wrap {
  display: flex;
  flex-direction: column;
  gap: 18px;
  margin-bottom: 18px;
}

.stats-group__title {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 9px;
  color: var(--muted);
  font-size: 10.5px;
}

.stats-group__badge {
  padding: 4px 9px;
  border-radius: 6px;
  font-size: 10px;
  font-weight: 900;
  letter-spacing: .5px;
}

.stats-group__badge--rkap {
  color: var(--blue);
  background: var(--blue-soft);
}

.stats-group__badge--prognosa {
  color: var(--teal);
  background: var(--teal-soft);
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(5, 1fr);
  gap: 13px;
}

.stat-card {
  position: relative;
  padding: 14px 16px 16px;
  border-radius: 12px;
  color: #fff;
  box-shadow: 0 6px 16px rgba(24, 59, 68, .14);
  overflow: hidden;
}

.stat-card--navy   { background: #241f6b; }
.stat-card--blue   { background: #0088d1; }
.stat-card--orange { background: #f06a24; }
.stat-card--green  { background: #10b81a; }

.stat-card__label {
  font-size: 11px;
  font-weight: 600;
  opacity: .95;
}

.stat-card__value {
  margin-top: 4px;
  font-size: 38px;
  line-height: 1.1;
  font-weight: 850;
}

.stat-card__icon {
  position: absolute;
  top: 12px;
  right: 12px;
  width: 34px;
  height: 34px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  background: rgba(255, 255, 255, .22);
}

/* Panels */
.panel {
  background: var(--surface);
  border: 1px solid var(--line);
  border-radius: 15px;
  box-shadow: 0 8px 28px rgba(24, 59, 68, .045);
}

.progress-panel {
  padding: 20px 22px 21px;
  margin-bottom: 18px;
}

.panel__heading,
.approval-heading {
  display: flex;
  align-items: flex-start;
  justify-content: space-between;
  gap: 20px;
}

.eyebrow {
  color: var(--teal);
  font-size: 8px;
  font-weight: 900;
  letter-spacing: 1.2px;
  margin-bottom: 5px;
}

.panel__title {
  margin: 0;
  color: var(--navy-dark);
  font-size: 16px;
  font-weight: 800;
}

.panel__description {
  margin: 5px 0 0;
  color: var(--muted);
  font-size: 10px;
  line-height: 1.5;
}

.legend {
  display: flex;
  align-items: center;
  gap: 15px;
  color: #75878d;
  font-size: 10px;
  padding-top: 2px;
}

.legend span {
  display: flex;
  align-items: center;
  gap: 6px;
}

.legend__dot {
  width: 8px;
  height: 8px;
  border-radius: 3px;
}

.legend__dot--rkap {
  background: var(--blue);
}

.legend__dot--prognosa {
  background: var(--teal);
}

.progress-summary {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 14px;
  margin-top: 18px;
}

.progress-card {
  padding: 16px;
  border: 1px solid #edf1f2;
  border-radius: 12px;
  background: linear-gradient(180deg, #fff, #fbfcfc);
}

.progress-card__header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 15px;
}

.progress-card__title {
  display: flex;
  align-items: center;
  gap: 10px;
}

.progress-card__badge {
  width: 34px;
  height: 34px;
  flex: 0 0 34px;
  border-radius: 9px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 900;
  font-size: 11px;
}

.progress-card__badge--rkap {
  color: var(--blue);
  background: var(--blue-soft);
}

.progress-card__badge--prognosa {
  color: var(--teal);
  background: var(--teal-soft);
}

.progress-card__title strong {
  display: block;
  color: var(--navy);
  font-size: 12px;
}

.progress-card__title small {
  display: block;
  color: #9aa7ab;
  margin-top: 3px;
  font-size: 9px;
}

.progress-card__percentage {
  color: var(--navy);
  font-size: 20px;
}

.progress-track {
  height: 10px;
  margin-top: 16px;
  padding: 2px;
  border-radius: 8px;
  background: #edf2f2;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  border-radius: 6px;
  position: relative;
  transition: width .35s ease;
}

.progress-fill::after {
  content: "";
  position: absolute;
  inset: 0;
  opacity: .18;
  background: repeating-linear-gradient(
    -45deg,
    transparent,
    transparent 7px,
    rgba(255,255,255,.7) 7px,
    rgba(255,255,255,.7) 11px
  );
}

.progress-fill--rkap {
  background: linear-gradient(90deg, #285b98, #4d80bd);
}

.progress-fill--prognosa {
  background: linear-gradient(90deg, #087c77, #32a49d);
}

.form-progress-list {
  list-style: none;
  margin: 16px 0 0;
  padding: 14px 0 0;
  border-top: 1px dashed var(--line);
  display: flex;
  flex-direction: column;
  gap: 11px;
}

.form-progress__top {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  gap: 12px;
  margin-bottom: 5px;
}

.form-progress__name {
  color: #42565d;
  font-size: 10.5px;
  font-weight: 650;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.form-progress__meta {
  flex: 0 0 auto;
  display: flex;
  align-items: baseline;
  gap: 8px;
  color: #9aa7ab;
  font-size: 9px;
}

.form-progress__meta strong {
  min-width: 30px;
  text-align: right;
  font-size: 10.5px;
}

.tone--done { color: var(--green); }
.tone--good { color: var(--navy); }
.tone--warn { color: #b9741d; }
.tone--low  { color: var(--red); }

.progress-track--sm {
  height: 7px;
  margin-top: 0;
  padding: 1.5px;
  border-radius: 6px;
}

.progress-track--sm .progress-fill {
  border-radius: 4px;
}

.progress-track--sm .progress-fill::after {
  display: none;
}

/* Form yang sudah 100% */
.progress-fill--done {
  background: linear-gradient(90deg, #128a52, #2fb57a);
}

/* Approval */
.approval-panel {
  overflow: hidden;
}

.approval-heading {
  padding: 21px 22px 17px;
  border-bottom: 1px solid var(--line);
}

.approval-summary {
  display: flex;
  align-items: center;
  padding: 7px 10px;
  border: 1px solid #edf1f2;
  border-radius: 10px;
  background: #fafcfc;
}

.approval-summary__item {
  display: flex;
  align-items: center;
  gap: 7px;
  color: #849398;
  font-size: 9px;
  padding: 0 7px;
}

.approval-summary__number {
  color: var(--navy);
  font-size: 16px;
  font-weight: 850;
}

.approval-summary__item--urgent .approval-summary__number {
  color: var(--red);
}

.approval-summary__divider {
  width: 1px;
  height: 25px;
  background: var(--line);
}

.table-section {
  border-bottom: 1px solid var(--line);
}

.table-section--last {
  border-bottom: 0;
}

.table-section__heading {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 15px;
  padding: 15px 22px 12px;
  background: #fcfdfd;
}

.table-title {
  display: flex;
  align-items: center;
  gap: 10px;
}

.table-title__icon {
  width: 31px;
  height: 31px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
}

.table-title__icon--rkap {
  color: var(--blue);
  background: var(--blue-soft);
}

.table-title__icon--prognosa {
  color: var(--teal);
  background: var(--teal-soft);
}

.table-title strong {
  display: block;
  color: var(--navy);
  font-size: 12px;
}

.table-title span:not(.table-title__icon) {
  display: block;
  margin-top: 2px;
  color: #9aa7ab;
  font-size: 9px;
}

.table-actions {
  display: flex;
  align-items: center;
  gap: 9px;
}

.selected-count {
  color: var(--teal);
  font-size: 9px;
  font-weight: 800;
}

.btn {
  border: 0;
  border-radius: 8px;
  padding: 8px 11px;
  font-size: 10px;
  font-weight: 800;
  cursor: pointer;
  transition: .18s ease;
}

.btn--approve {
  color: #fff;
  background: var(--green);
  box-shadow: 0 4px 10px rgba(22, 132, 81, .15);
}

.btn--approve:hover:not(:disabled) {
  transform: translateY(-1px);
  background: #117044;
}

.btn:disabled {
  opacity: .35;
  cursor: default;
  box-shadow: none;
}

/* Tables */
.table-scroll {
  overflow-x: auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
  min-width: 1050px;
  font-size: 10.5px;
}

.table thead th {
  padding: 10px 13px;
  background: #f7f9f9;
  border-top: 1px solid #f0f2f2;
  border-bottom: 1px solid var(--line);
  color: #73858b;
  text-align: left;
  font-size: 9px;
  font-weight: 800;
  white-space: nowrap;
}

.table tbody td {
  padding: 12px 13px;
  border-bottom: 1px solid #f0f3f3;
  color: #42565d;
  vertical-align: middle;
}

.table tbody tr {
  transition: background .15s ease;
}

.table tbody tr:hover {
  background: #fafcfc;
}

.table tbody tr.row--selected {
  background: #f0faf7;
}

.table input[type="checkbox"] {
  width: 14px;
  height: 14px;
  accent-color: var(--teal);
  cursor: pointer;
}

.table__checkbox-col {
  width: 44px;
  text-align: center !important;
}

.table__number {
  color: #9aa7ab !important;
  width: 45px;
}

.table__action-col {
  text-align: right !important;
}

.table__action {
  text-align: right;
  white-space: nowrap;
}

.form-name strong {
  display: block;
  max-width: 280px;
  color: var(--navy);
  font-size: 10.5px;
}

.form-name span {
  display: block;
  margin-top: 3px;
  color: #a1adb1;
  font-size: 8.5px;
}

.cell-stack,
.date-stack {
  display: flex;
  flex-direction: column;
  gap: 3px;
}

.cell-stack strong,
.date-stack strong {
  color: #455960;
  font-size: 9.5px;
}

.cell-stack span,
.date-stack span {
  color: #a0adb1;
  font-size: 8.5px;
}

.vp-badge {
  display: inline-flex;
  align-items: center;
  padding: 5px 7px;
  color: #17784d;
  background: var(--green-soft);
  border: 1px solid #d4efdf;
  border-radius: 6px;
  font-size: 8px;
  font-weight: 800;
  white-space: nowrap;
}

.deadline {
  display: inline-flex;
  align-items: center;
  padding: 5px 7px;
  border-radius: 6px;
  color: #63767b;
  background: #f1f4f4;
  font-size: 9px;
  font-weight: 750;
  white-space: nowrap;
}

.deadline--urgent {
  color: var(--red);
  background: var(--red-soft);
}

.action-btn {
  border: 1px solid transparent;
  border-radius: 7px;
  padding: 6px 8px;
  font-size: 9px;
  font-weight: 800;
  cursor: pointer;
  margin-left: 4px;
}

.action-btn--approve {
  color: #fff;
  background: var(--green);
}

.action-btn--approve:hover {
  background: #117044;
}

.action-btn--view {
  color: #567078;
  border-color: #dfe7e8;
  background: #fff;
}

.action-btn--view:hover {
  background: #f5f8f8;
}

.table__empty {
  text-align: center;
  padding: 35px 15px !important;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 5px;
}

.empty-state > span {
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  color: var(--green);
  background: var(--green-soft);
  font-weight: 900;
}

.empty-state strong {
  color: #53676d;
  font-size: 10px;
}

.empty-state small {
  color: #a1adb1;
  font-size: 8.5px;
}

.table-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 11px 22px;
  color: #9aa7ab;
  font-size: 9px;
  background: #fcfdfd;
}

/* Notice */
.notice {
  position: fixed;
  right: 24px;
  bottom: 24px;
  z-index: 50;
  display: flex;
  align-items: center;
  gap: 9px;
  max-width: 420px;
  padding: 12px 15px;
  border-radius: 10px;
  font-size: 10.5px;
  font-weight: 750;
  box-shadow: 0 12px 35px rgba(20, 50, 58, .15);
}

.notice span {
  width: 21px;
  height: 21px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.notice.success {
  color: #176e47;
  background: #eefaf4;
  border: 1px solid #d5f0e1;
}

.notice.success span {
  background: #d9f3e5;
}

.notice.reject {
  color: #a62f2f;
  background: #fff2f2;
  border: 1px solid #f5dada;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity .2s ease, transform .2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
  transform: translateY(7px);
}

/* Reject buttons */
.btn--reject {
  color: var(--red);
  background: var(--red-soft);
  border: 1px solid #f5d5d5;
}

.btn--reject:hover:not(:disabled) {
  background: #ffe3e3;
}

.btn--reject-solid {
  color: #fff;
  background: var(--red);
  box-shadow: 0 4px 10px rgba(217, 75, 75, .18);
}

.btn--reject-solid:hover {
  background: #bd3a3a;
}

.btn--ghost {
  color: #567078;
  background: #fff;
  border: 1px solid #dfe7e8;
}

.btn--ghost:hover {
  background: #f5f8f8;
}

.action-btn--reject {
  color: var(--red);
  background: var(--red-soft);
  border-color: #f5d5d5;
}

.action-btn--reject:hover {
  background: #ffe3e3;
}

/* Modal */
.modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 60;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 20px;
  background: rgba(13, 48, 59, .45);
  backdrop-filter: blur(3px);
}

.modal {
  width: min(460px, 100%);
  padding: 22px;
  background: var(--surface);
  border-radius: 15px;
  box-shadow: 0 24px 60px rgba(13, 48, 59, .28);
}

.modal__header {
  display: flex;
  align-items: center;
  gap: 12px;
  margin-bottom: 16px;
}

.modal__icon {
  width: 38px;
  height: 38px;
  flex: 0 0 38px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 11px;
  color: var(--red);
  background: var(--red-soft);
  font-weight: 900;
}

.modal__title {
  margin: 0;
  color: var(--navy-dark);
  font-size: 15px;
  font-weight: 800;
}

.modal__subtitle {
  margin: 3px 0 0;
  color: var(--muted);
  font-size: 10.5px;
}

.modal__list {
  margin: 0 0 14px;
  padding: 9px 12px 9px 26px;
  max-height: 110px;
  overflow-y: auto;
  border: 1px solid var(--line);
  border-radius: 9px;
  background: #fafcfc;
  color: #42565d;
  font-size: 10.5px;
  line-height: 1.7;
}

.modal__list small {
  color: #a0adb1;
}

.modal__label {
  display: block;
  margin-bottom: 6px;
  color: var(--navy);
  font-size: 11px;
  font-weight: 800;
}

.modal__label span {
  margin-left: 4px;
  color: var(--red);
  font-size: 9px;
  font-weight: 700;
}

.modal__textarea {
  width: 100%;
  padding: 10px 12px;
  border: 1px solid #dfe7e8;
  border-radius: 10px;
  resize: vertical;
  color: var(--text);
  font: inherit;
  font-size: 11.5px;
  line-height: 1.5;
  outline: none;
  transition: border-color .15s ease, box-shadow .15s ease;
}

.modal__textarea:focus {
  border-color: var(--teal);
  box-shadow: 0 0 0 3px rgba(22, 139, 134, .12);
}

.modal__textarea--error {
  border-color: var(--red);
  box-shadow: 0 0 0 3px rgba(217, 75, 75, .1);
}

.modal__error {
  display: block;
  margin-top: 5px;
  color: var(--red);
  font-size: 10px;
}

.modal__footer {
  display: flex;
  justify-content: flex-end;
  gap: 9px;
  margin-top: 18px;
}

.modal__footer .btn {
  padding: 9px 14px;
  font-size: 11px;
}

/* Responsive */
@media (max-width: 1100px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }

  .content {
    width: min(100% - 36px, 1480px);
  }
}

@media (max-width: 760px) {
  .topbar {
    padding: 10px 18px;
  }

  .user__role {
    display: none;
  }

  .content {
    width: calc(100% - 24px);
    padding-top: 20px;
  }

  .page-intro,
  .panel__heading,
  .approval-heading,
  .table-section__heading {
    align-items: flex-start;
    flex-direction: column;
  }

  .live-status {
    align-self: stretch;
    justify-content: center;
  }

  .stats-grid,
  .progress-summary {
    grid-template-columns: 1fr;
  }

  .legend {
    padding-top: 0;
  }

  .approval-summary {
    width: 100%;
    justify-content: center;
  }

  .table-actions {
    width: 100%;
    justify-content: flex-end;
  }

  .table-footer {
    gap: 10px;
    align-items: flex-start;
    flex-direction: column;
  }
}

@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }

  .page-intro h1 {
    font-size: 21px;
  }

  .progress-panel,
  .approval-heading {
    padding-left: 15px;
    padding-right: 15px;
  }

  .table-section__heading {
    padding-left: 15px;
    padding-right: 15px;
  }

  .action-btn span {
    display: none;
  }
}
</style>
