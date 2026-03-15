<template>
  <div
    class="spread-member-navbar"
    :class="{
      'spread-member-navbar--sidebar-open': sidebarOpen,
      'spread-member-navbar--mobile-open':  mobileDrawerOpen,
    }"
    v-if="content.isAuthenticated"
  >
    <!-- ── SERVICE ALERT BANNER ───────────────────────────────────────── -->
    <div
      v-if="content.showAlertBanner && alertBanners.length"
      class="spread-member-navbar__alert-banner"
    >
      <div class="spread-member-navbar__alert-banner-inner">
        <div class="spread-member-navbar__alert-banner-item" v-for="alert in visibleAlerts" :key="alert.id">
          <span class="spread-member-navbar__alert-banner-dot"></span>
          <span class="spread-member-navbar__alert-banner-text">{{ alert.message || alert.headline }}</span>
          <button
            class="spread-member-navbar__alert-banner-dismiss"
            @click="dismissAlert(alert.id)"
            aria-label="Dismiss alert"
          >
            <svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
            </svg>
          </button>
        </div>
      </div>
    </div>

    <!-- ── TOP BAR ────────────────────────────────────────────────────── -->
    <header class="spread-member-navbar__topbar">
      <div class="spread-member-navbar__topbar-left">
        <button
          class="spread-member-navbar__hamburger"
          @click="toggleNav"
          :aria-expanded="mobileDrawerOpen || sidebarOpen"
          aria-label="Open navigation"
        >
          <span class="spread-member-navbar__hamburger-line"></span>
          <span class="spread-member-navbar__hamburger-line"></span>
          <span class="spread-member-navbar__hamburger-line"></span>
        </button>
      </div>

      <div class="spread-member-navbar__brand" @click="navigate('/', 'Home')" role="button" tabindex="0" aria-label="Go to home" title="Home">
        <img class="spread-member-navbar__logo spread-member-navbar__logo--full" :src="content.logoUrl" alt="Spread.co" />
        <img class="spread-member-navbar__logo spread-member-navbar__logo--mark" :src="content.logoMarkUrl" alt="Spread.co" />
      </div>

      <div class="spread-member-navbar__topbar-right">
        <template v-if="!content.hideBuiltinTopbarIcons">
          <!-- Notification bell -->
          <button
            v-if="content.showNotificationBell"
            class="spread-member-navbar__icon-btn"
            @click="onNotificationBell"
            title="Notifications"
          >
            <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M18 8A6 6 0 0 0 6 8c0 7-3 9-3 9h18s-3-2-3-9"/>
              <path d="M13.73 21a2 2 0 0 1-3.46 0"/>
            </svg>
            <span v-if="alertBanners.length" class="spread-member-navbar__icon-badge">{{ alertBanners.length > 9 ? '9+' : alertBanners.length }}</span>
          </button>

          <!-- Cart -->
          <button class="spread-member-navbar__icon-btn" @click="handleCart" title="Cart">
            <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z"/>
              <line x1="3" y1="6" x2="21" y2="6"/>
              <path d="M16 10a4 4 0 0 1-8 0"/>
            </svg>
            <span class="spread-member-navbar__icon-badge" v-if="cartCount > 0">{{ cartCount > 99 ? '99+' : cartCount }}</span>
          </button>

          <!-- Theme toggle -->
          <button
            v-if="content.showThemeToggle !== false"
            class="spread-member-navbar__icon-btn spread-member-navbar__theme-toggle"
            :class="`spread-member-navbar__theme-toggle--${themeMode}`"
            @click="cycleTheme"
            :title="themeToggleLabel"
            :data-tooltip="themeToggleLabel"
            aria-label="Toggle colour theme"
          >
            <!-- Moon: dark mode active -->
            <svg v-if="themeMode === 'dark'" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M21 12.79A9 9 0 1 1 11.21 3 7 7 0 0 0 21 12.79z"/>
            </svg>
            <!-- Sun: light mode active -->
            <svg v-else-if="themeMode === 'light'" viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <circle cx="12" cy="12" r="5"/>
              <line x1="12" y1="1" x2="12" y2="3"/><line x1="12" y1="21" x2="12" y2="23"/>
              <line x1="4.22" y1="4.22" x2="5.64" y2="5.64"/><line x1="18.36" y1="18.36" x2="19.78" y2="19.78"/>
              <line x1="1" y1="12" x2="3" y2="12"/><line x1="21" y1="12" x2="23" y2="12"/>
              <line x1="4.22" y1="19.78" x2="5.64" y2="18.36"/><line x1="18.36" y1="5.64" x2="19.78" y2="4.22"/>
            </svg>
            <!-- Monitor: system preference -->
            <svg v-else viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <rect x="2" y="3" width="20" height="14" rx="2" ry="2"/>
              <line x1="8" y1="21" x2="16" y2="21"/>
              <line x1="12" y1="17" x2="12" y2="21"/>
            </svg>
          </button>

          <!-- User avatar -->
          <div class="spread-member-navbar__avatar-wrap" @click.stop="toggleUserMenu">
            <button class="spread-member-navbar__avatar-btn" :title="content.userName">
              <img v-if="content.userAvatarUrl" :src="content.userAvatarUrl" class="spread-member-navbar__avatar-img" alt="Avatar" />
              <span v-else class="spread-member-navbar__avatar-initials">{{ initials }}</span>
            </button>
            <!-- User popover -->
            <div class="spread-member-navbar__user-popover" v-if="userMenuOpen" v-click-outside="closeUserMenu">
              <div class="spread-member-navbar__user-popover-meta">
                <div class="spread-member-navbar__user-popover-name">{{ content.userName }}</div>
                <div class="spread-member-navbar__user-popover-email">{{ content.userEmail }}</div>
              </div>
              <hr class="spread-member-navbar__user-popover-divider" />
              <button class="spread-member-navbar__user-popover-item" @click="navigate('/account', 'My Account')">My Account</button>
              <hr class="spread-member-navbar__user-popover-divider" />
              <button class="spread-member-navbar__user-popover-signout" @click="handleLogout">Sign out</button>
            </div>
          </div>
        </template>
      </div>
    </header>

    <!-- ── SIDEBAR ─────────────────────────────────────────────────────── -->
    <nav class="spread-member-navbar__sidebar" aria-label="Member navigation">
      <!-- Have your say sticky -->
      <div class="spread-member-navbar__have-your-say-wrap">
        <button class="spread-member-navbar__have-your-say" @click="onHaveYourSay" title="Have your say">
          <svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
          </svg>
          <span class="spread-member-navbar__sidebar-label">Have your say</span>
        </button>
      </div>

      <ul class="spread-member-navbar__sidebar-list">
        <li v-for="link in NAV_LINKS" :key="link.path" class="spread-member-navbar__sidebar-item">
          <button
            class="spread-member-navbar__sidebar-link"
            :class="{ 'spread-member-navbar__sidebar-link--active': isActive(link.path) }"
            @click="navigate(link.path, link.label)"
            :title="link.label"
          >
            <span class="spread-member-navbar__sidebar-icon" v-html="link.icon" aria-hidden="true"></span>
            <span class="spread-member-navbar__sidebar-label">{{ link.label }}</span>
          </button>
        </li>
      </ul>

      <!-- Sign out bottom -->
      <div class="spread-member-navbar__sidebar-footer">
        <button class="spread-member-navbar__sidebar-signout" @click="handleLogout" title="Sign out">
          <svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
            <path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/>
            <polyline points="16 17 21 12 16 7"/>
            <line x1="21" y1="12" x2="9" y2="12"/>
          </svg>
          <span class="spread-member-navbar__sidebar-label">Sign out</span>
        </button>
      </div>

      <!-- ── CERANOVANAV BADGE ─────────────────────────────────────────── -->
      <a
        href="https://ceranovaai.com"
        target="_blank"
        rel="noopener noreferrer"
        title="Built by CeraNova"
        class="spread-member-navbar__ceranovanav"
      >
        <img :src="ceraNovaIcon" class="spread-member-navbar__ceranovanav-icon" width="32" height="32" alt="CeraNova" />
        <span class="spread-member-navbar__ceranovanav-content">
          <span class="spread-member-navbar__ceranovanav-built-by">Built by</span>
          <span class="spread-member-navbar__ceranovanav-brand">
            <span class="spread-member-navbar__ceranovanav-cera">Cera</span><span class="spread-member-navbar__ceranovanav-nova">Nova</span>
          </span>
        </span>
      </a>
    </nav>

    <!-- ── MOBILE DRAWER ──────────────────────────────────────────────── -->
    <div class="spread-member-navbar__drawer" v-if="mobileDrawerOpen">
      <div class="spread-member-navbar__drawer-backdrop" @click="closeMobileDrawer"></div>
      <nav class="spread-member-navbar__drawer-panel" aria-label="Mobile member navigation">
        <div class="spread-member-navbar__drawer-header">
          <div class="spread-member-navbar__drawer-member">
            <div v-if="content.userAvatarUrl">
              <img :src="content.userAvatarUrl" class="spread-member-navbar__drawer-avatar" alt="Avatar" />
            </div>
            <div v-else class="spread-member-navbar__drawer-avatar-initials">{{ initials }}</div>
            <div>
              <div class="spread-member-navbar__drawer-name">{{ content.userName }}</div>
              <div class="spread-member-navbar__drawer-email">{{ content.userEmail }}</div>
            </div>
          </div>
          <button class="spread-member-navbar__drawer-close" @click="closeMobileDrawer" aria-label="Close menu">
            <svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="2" aria-hidden="true">
              <line x1="18" y1="6" x2="6" y2="18"/><line x1="6" y1="6" x2="18" y2="18"/>
            </svg>
          </button>
        </div>

        <div class="spread-member-navbar__drawer-have-your-say">
          <button class="spread-member-navbar__drawer-hys-btn" @click="onHaveYourSay; closeMobileDrawer();">
            <svg viewBox="0 0 24 24" width="15" height="15" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
              <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
            </svg>
            Have your say
          </button>
        </div>

        <ul class="spread-member-navbar__drawer-list">
          <li v-for="link in NAV_LINKS" :key="link.path">
            <button
              class="spread-member-navbar__drawer-link"
              :class="{ 'spread-member-navbar__drawer-link--active': isActive(link.path) }"
              @click="navigate(link.path, link.label); closeMobileDrawer();"
            >
              <span class="spread-member-navbar__drawer-icon" v-html="link.icon" aria-hidden="true"></span>
              {{ link.label }}
            </button>
          </li>
        </ul>

        <div class="spread-member-navbar__drawer-footer">
          <button class="spread-member-navbar__drawer-signout" @click="handleLogout; closeMobileDrawer();">Sign out</button>
        </div>
        <a
          href="https://ceranovaai.com"
          target="_blank"
          rel="noopener noreferrer"
          title="Built by CeraNova"
          class="spread-member-navbar__drawer-ceranovanav"
        >
          <img :src="ceraNovaIcon" class="spread-member-navbar__ceranovanav-icon" width="32" height="32" alt="CeraNova" />
          <span class="spread-member-navbar__drawer-ceranovanav-content">
            <span class="spread-member-navbar__ceranovanav-built-by">Built by</span>
            <span class="spread-member-navbar__ceranovanav-brand">
              <span class="spread-member-navbar__ceranovanav-cera">Cera</span><span class="spread-member-navbar__ceranovanav-nova">Nova</span>
            </span>
          </span>
        </a>
      </nav>
    </div>
  </div>
</template>

<script>
/* eslint-disable no-unused-vars */
import ceraNovaIcon from '../MainIcon (64x64).png';

const ICONS = {
  home:     '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M3 9l9-7 9 7v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2z"/><polyline points="9 22 9 12 15 12 15 22"/></svg>',
  shop:     '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z"/><line x1="3" y1="6" x2="21" y2="6"/><path d="M16 10a4 4 0 0 1-8 0"/></svg>',
  account:  '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/><circle cx="12" cy="7" r="4"/></svg>',
  carts:    '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="9" cy="21" r="1"/><circle cx="20" cy="21" r="1"/><path d="M1 1h4l2.68 13.39a2 2 0 0 0 2 1.61h9.72a2 2 0 0 0 2-1.61L23 6H6"/></svg>',
  favorites:'<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M20.84 4.61a5.5 5.5 0 0 0-7.78 0L12 5.67l-1.06-1.06a5.5 5.5 0 0 0-7.78 7.78l1.06 1.06L12 21.23l7.78-7.78 1.06-1.06a5.5 5.5 0 0 0 0-7.78z"/></svg>',
  orders:   '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/><line x1="16" y1="13" x2="8" y2="13"/><line x1="16" y1="17" x2="8" y2="17"/><polyline points="10 9 9 9 8 9"/></svg>',
  faq:      '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><path d="M9.09 9a3 3 0 0 1 5.83 1c0 2-3 3-3 3"/><line x1="12" y1="17" x2="12.01" y2="17"/></svg>',
  about:    '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><line x1="12" y1="16" x2="12.01" y2="16"/></svg>',
  policies: '<svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M14 2H6a2 2 0 0 0-2 2v16a2 2 0 0 0 2 2h12a2 2 0 0 0 2-2V8z"/><polyline points="14 2 14 8 20 8"/></svg>',
};

const NAV_LINKS = [
  { path: '/',          label: 'Home',        icon: ICONS.home },
  { path: '/shop',      label: 'Shop',        icon: ICONS.shop },
  { path: '/account',   label: 'My Account',  icon: ICONS.account },
  { path: '/carts',     label: 'Saved Carts', icon: ICONS.carts },
  { path: '/favorites', label: 'Favourites',  icon: ICONS.favorites },
  { path: '/orders',    label: 'Orders',      icon: ICONS.orders },
  { path: '/faq',       label: 'FAQ',         icon: ICONS.faq },
  { path: '/about',     label: 'About',       icon: ICONS.about },
  { path: '/policies',  label: 'Policies',    icon: ICONS.policies },
];

function createSpreadClient(url, anonKey, token) {
  return {
    url, anonKey,
    headers: { 'Content-Type': 'application/json', apikey: anonKey, Authorization: `Bearer ${token || anonKey}` },
  };
}

async function callRpc(url, anonKey, token, fnName, params = {}) {
  const res = await fetch(`${url}/rest/v1/rpc/${fnName}`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json', apikey: anonKey, Authorization: `Bearer ${token || anonKey}` },
    body: JSON.stringify(params),
  });
  if (!res.ok) throw new Error(`RPC ${fnName} failed: ${res.status}`);
  return res.json();
}

function openRealtimeChannel(wsBase, anonKey, table, onPayload) {
  try {
    const url = `${wsBase}/realtime/v1/websocket?apikey=${anonKey}&vsn=1.0.0`;
    const ws = new WebSocket(url);
    const ref = { current: 1 };
    ws.onopen = () => {
      ws.send(JSON.stringify({ topic: 'realtime:public', event: 'phx_join', payload: {
        config: { broadcast: { ack: false, self: false }, presence: { key: '' },
          postgres_changes: [{ event: '*', schema: 'public', table }] },
      }, ref: String(ref.current++) }));
    };
    ws.onmessage = (e) => {
      try {
        const msg = JSON.parse(e.data);
        if (msg.event === 'postgres_changes') onPayload(msg.payload);
      } catch (_) {}
    };
    return ws;
  } catch (_) { return null; }
}

export default {
  props: {
    /* wwEditor:start */
    wwEditorState: { type: Object, required: true },
    /* wwEditor:end */
    content:        { type: Object, required: true },
    wwFrontState:   { type: Object, required: true },
    wwElementState: { type: Object, required: true },
  },

  emits: ['trigger-event', 'update:content'],

  directives: {
    clickOutside: {
      mounted(el, binding) {
        el._clickOutsideHandler = (e) => {
          if (!el.contains(e.target)) binding.value(e);
        };
        try {
          const doc = typeof wwLib !== 'undefined' ? wwLib.getFrontDocument() : document;
          doc.addEventListener('click', el._clickOutsideHandler);
        } catch (_) {}
      },
      unmounted(el) {
        try {
          const doc = typeof wwLib !== 'undefined' ? wwLib.getFrontDocument() : document;
          doc.removeEventListener('click', el._clickOutsideHandler);
        } catch (_) {}
      },
    },
  },

  data() {
    return {
      NAV_LINKS,
      mobileDrawerOpen: false,
      sidebarOpen:      false,
      activePath:       '',
      userMenuOpen:     false,
      alertBanners:     [],
      dismissedIds:     [],
      cartCount:        0,
      _alertPoll:       null,
      _cartPoll:        null,
      _realtimeWs:      null,
      ceraNovaIcon,
      themeMode:        'system',
      _mq:              null,
      _mqHandler:       null,
    };
  },

  mounted() {
    this._initActivePath();
    this._startAlertPolling();
    this._startCartPolling();
    this._openAlertsChannel();
    this.initTheme();
  },

  beforeUnmount() {
    clearInterval(this._alertPoll);
    clearInterval(this._cartPoll);
    try { if (this._realtimeWs) this._realtimeWs.close(); } catch (_) {}
    if (this._mq) { try { this._mq.removeEventListener('change', this._mqHandler); } catch (_) {} }
  },

  watch: {
    wwFrontState:    { deep: true, handler() { this._initActivePath(); } },
    'content.accessToken':    { handler() { this._startAlertPolling(); this._startCartPolling(); this._openAlertsChannel(); } },
    'content.pollIntervalAlerts': { handler() { this._startAlertPolling(); } },
    'content.pollIntervalCart':   { handler() { this._startCartPolling(); } },
    'content.themePreference'(val) {
      if (val && ['light', 'dark', 'system'].includes(val)) this.applyTheme(val);
    },
  },

  computed: {
    initials() {
      const n = this.content.userName || '';
      return n.split(' ').filter(Boolean).map(p => p[0]).join('').toUpperCase().slice(0, 2) || '?';
    },
    themeToggleLabel() {
      const labels = { system: 'Theme: System', light: 'Theme: Light', dark: 'Theme: Dark' };
      return labels[this.themeMode] || 'Toggle theme';
    },
    visibleAlerts() {
      const max = this.content.alertBannerMaxCount || 3;
      return this.alertBanners.filter(a => !this.dismissedIds.includes(a.id)).slice(0, max);
    },
  },

  methods: {
    _initActivePath() {
      try {
        const win = typeof wwLib !== 'undefined' ? wwLib.getFrontWindow() : window;
        if (win && win.location) this.activePath = win.location.pathname;
      } catch (_) {}
    },

    _startAlertPolling() {
      clearInterval(this._alertPoll);
      if (!this.content.showAlertBanner || !this.content.supabaseUrl) return;
      const ms = Math.max((this.content.pollIntervalAlerts || 60) * 1000, 15000);
      this._fetchAlerts();
      this._alertPoll = setInterval(() => this._fetchAlerts(), ms);
    },

    _startCartPolling() {
      clearInterval(this._cartPoll);
      if (!this.content.supabaseUrl || !this.content.userId) return;
      const ms = Math.max((this.content.pollIntervalCart || 30) * 1000, 10000);
      this._fetchCartCount();
      this._cartPoll = setInterval(() => this._fetchCartCount(), ms);
    },

    _openAlertsChannel() {
      try { if (this._realtimeWs) this._realtimeWs.close(); } catch (_) {}
      if (!this.content.supabaseUrl || !this.content.supabaseAnonKey) return;
      const wsBase = this.content.supabaseUrl.replace(/^http/, 'ws');
      this._realtimeWs = openRealtimeChannel(wsBase, this.content.supabaseAnonKey, 'service_alerts', () => {
        this._fetchAlerts();
      });
    },

    async _fetchAlerts() {
      try {
        const { supabaseUrl, supabaseAnonKey, accessToken } = this.content;
        if (!supabaseUrl || !supabaseAnonKey) return;
        const client = createSpreadClient(supabaseUrl, supabaseAnonKey, accessToken);
        const res = await fetch(
          `${supabaseUrl}/rest/v1/service_alerts?status=eq.active&order=created_at.desc&limit=5`,
          { headers: client.headers }
        );
        if (res.ok) this.alertBanners = await res.json();
      } catch (_) {}
    },

    async _fetchCartCount() {
      try {
        const { supabaseUrl, supabaseAnonKey, accessToken, userId } = this.content;
        if (!supabaseUrl || !userId) return;
        const data = await callRpc(supabaseUrl, supabaseAnonKey, accessToken, 'get_cart_item_count', { p_user_id: userId });
        if (typeof data === 'number') this.cartCount = data;
        else if (data && typeof data.count === 'number') this.cartCount = data.count;
      } catch (_) {}
    },

    navigate(path, label) {
      this.activePath = path;
      this.sidebarOpen = false;
      this.$emit('trigger-event', { name: 'nav:navigate', event: { path, label } });
    },

    isActive(path) { return this.activePath === path; },

    toggleNav() {
      try {
        const win = typeof wwLib !== 'undefined' ? wwLib.getFrontWindow() : window;
        const w = win ? win.innerWidth : 0;
        if (w > 0 && w < 768) {
          this.mobileDrawerOpen = !this.mobileDrawerOpen;
          this.$emit('trigger-event', { name: 'nav:menu-toggle', event: { isOpen: this.mobileDrawerOpen } });
        } else {
          this.sidebarOpen = !this.sidebarOpen;
          this.$emit('trigger-event', { name: 'nav:menu-toggle', event: { isOpen: this.sidebarOpen } });
        }
      } catch (_) {
        this.mobileDrawerOpen = !this.mobileDrawerOpen;
        this.$emit('trigger-event', { name: 'nav:menu-toggle', event: { isOpen: this.mobileDrawerOpen } });
      }
    },

    closeMobileDrawer() { this.mobileDrawerOpen = false; },

    toggleUserMenu() {
      this.userMenuOpen = !this.userMenuOpen;
      if (this.userMenuOpen) this.$emit('trigger-event', { name: 'nav:user-menu-open', event: {} });
    },

    closeUserMenu() { this.userMenuOpen = false; },

    handleLogout() {
      this.userMenuOpen = false;
      this.closeMobileDrawer();
      this.$emit('trigger-event', { name: 'nav:logout', event: {} });
    },

    handleCart() {
      this.$emit('trigger-event', { name: 'nav:cart', event: { itemCount: this.cartCount } });
    },

    onHaveYourSay() {
      this.$emit('trigger-event', { name: 'nav:have-your-say', event: {} });
    },

    onNotificationBell() {
      this.$emit('trigger-event', { name: 'nav:notification-bell-clicked', event: { alertCount: this.alertBanners.length } });
    },

    dismissAlert(id) {
      this.dismissedIds.push(id);
      this.$emit('trigger-event', { name: 'nav:alert-banner-dismissed', event: { alertId: id } });
    },

    onAlertNotificationClick(alertId) {
      this.$emit('trigger-event', { name: 'nav:alert-notification-clicked', event: { alertId } });
    },

    initTheme() {
      try {
        const doc = typeof wwLib !== 'undefined' ? wwLib.getFrontDocument() : document;
        const win = typeof wwLib !== 'undefined' ? wwLib.getFrontWindow() : window;
        let mode = this.content?.themePreference;
        if (!mode || !['light', 'dark', 'system'].includes(mode)) {
          const cookie = doc.cookie || '';
          const match = cookie.match(/spread-theme=(light|dark|system)/);
          mode = match ? match[1] : 'system';
        }
        this.applyTheme(mode);
        this._mqHandler = () => { if (this.themeMode === 'system') this.applyTheme('system'); };
        try {
          this._mq = win?.matchMedia?.('(prefers-color-scheme: dark)');
          this._mq?.addEventListener?.('change', this._mqHandler);
        } catch (_) {}
      } catch (_) {}
    },

    applyTheme(mode) {
      this.themeMode = mode;
      try {
        const doc = typeof wwLib !== 'undefined' ? wwLib.getFrontDocument() : document;
        const win = typeof wwLib !== 'undefined' ? wwLib.getFrontWindow() : window;
        const isDark = mode === 'dark' || (mode === 'system' && (win?.matchMedia?.('(prefers-color-scheme: dark)')?.matches ?? false));
        doc.documentElement.classList.toggle('dark', isDark);
        doc.cookie = `spread-theme=${mode}; path=/; max-age=${365 * 24 * 60 * 60}; SameSite=Lax`;
        this.$emit('trigger-event', { name: 'theme:change', event: { mode, isDark } });
      } catch (_) {}
    },

    cycleTheme() {
      const order = ['system', 'light', 'dark'];
      const next = order[(order.indexOf(this.themeMode) + 1) % order.length];
      this.applyTheme(next);
    },
  },
};
</script>

<style scoped>
/* ── Design tokens ─────────────────────────────────────────────────────── */
.spread-member-navbar {
  --spread-primary:        #4B162D;
  --spread-accent:         #CE6632;
  --spread-accent-hover:   #B85A2B;
  --spread-black:          #141414;
  --spread-text-secondary: #2B2B2B;
  --spread-text-tertiary:  #4B5563;
  --spread-white:          #FFFFFF;
  --spread-surface:        #FFFFFF;
  --spread-background:     #FBFAF8;
  --spread-border:         #F3EADF;
  --spread-border-outer:   #EFE7DE;
  --spread-sand-light:     #EDC79F;
  --spread-radius-sm:      8px;
  --spread-radius-md:      12px;
  --spread-radius-lg:      16px;
  --spread-radius-full:    9999px;
  --spread-shadow-lg:      0 10px 15px rgba(0,0,0,0.1);
  --spread-font-family:    'Work Sans', ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
  --spread-topbar-height:        64px;
  --spread-topbar-height-mobile: 56px;
  --spread-alert-height:         40px;
  --spread-sidebar-collapsed:    56px;
  --spread-sidebar-expanded:     220px;
}

.spread-member-navbar *, .spread-member-navbar *::before, .spread-member-navbar *::after {
  box-sizing: border-box; margin: 0; padding: 0;
}
.spread-member-navbar ul { list-style: none; }

/* ── ALERT BANNER ────────────────────────────────────────────────────── */
.spread-member-navbar__alert-banner {
  position: fixed; top: 0; left: 0; right: 0; z-index: 250;
  background: #7C2D12; /* amber-900 equivalent for service alerts */
  min-height: var(--spread-alert-height);
  font-family: var(--spread-font-family);
}
.spread-member-navbar__alert-banner-inner { padding: 0 1rem; }
.spread-member-navbar__alert-banner-item {
  display: flex; align-items: center; gap: 8px;
  min-height: var(--spread-alert-height); padding: 6px 0;
  border-bottom: 1px solid rgba(255,255,255,0.1);
}
.spread-member-navbar__alert-banner-item:last-child { border-bottom: none; }
.spread-member-navbar__alert-banner-dot {
  flex-shrink: 0; width: 8px; height: 8px; border-radius: 50%;
  background: #FCA5A5;
}
.spread-member-navbar__alert-banner-text {
  flex: 1; font-size: 13px; font-weight: 500; color: #FEF2F2;
  line-height: 1.4; overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}
.spread-member-navbar__alert-banner-dismiss {
  flex-shrink: 0; width: 28px; height: 28px; display: flex; align-items: center;
  justify-content: center; background: rgba(255,255,255,0.08); border: none;
  border-radius: var(--spread-radius-sm); color: rgba(255,255,255,0.7); cursor: pointer;
  transition: background 0.12s ease;
}
.spread-member-navbar__alert-banner-dismiss:hover { background: rgba(255,255,255,0.18); color: #FEF2F2; }

/* ── TOP BAR ─────────────────────────────────────────────────────────── */
.spread-member-navbar__topbar {
  position: fixed; top: 0; left: 0; right: 0;
  height: var(--spread-topbar-height-mobile);
  background: var(--spread-primary);
  display: flex; align-items: center; justify-content: space-between;
  padding: 0 1rem; z-index: 200; font-family: var(--spread-font-family);
  transition: top 0.2s ease;
}
/* nudge topbar down when alert banner present */
.spread-member-navbar:has(.spread-member-navbar__alert-banner) .spread-member-navbar__topbar {
  top: auto; /* let alert banner push it via margin via JS if needed — rely on wrapper alignment */
}

@media (min-width: 768px) {
  .spread-member-navbar__topbar {
    padding: 0 1rem 0 calc(var(--spread-sidebar-collapsed) + 1rem);
  }
}
@media (min-width: 1024px) {
  .spread-member-navbar__topbar { height: var(--spread-topbar-height); }
}

.spread-member-navbar__topbar-left {
  width: 44px; display: flex; align-items: center; flex-shrink: 0;
}
.spread-member-navbar__topbar-right {
  display: flex; align-items: center; gap: 0.5rem; flex-shrink: 0;
}
@media (min-width: 480px) {
  .spread-member-navbar__topbar-left { width: auto; }
  .spread-member-navbar__topbar-right { gap: 0.75rem; }
}

/* Brand */
.spread-member-navbar__brand {
  position: absolute; left: 50%; transform: translateX(-50%);
  cursor: pointer; display: flex; align-items: center;
  max-width: calc(100% - 160px);
}
.spread-member-navbar__logo { display: block; height: auto; }
.spread-member-navbar__logo--full { display: none; }
.spread-member-navbar__logo--mark {
  display: block; width: 36px; height: 36px;
  border-radius: var(--spread-radius-full); object-fit: contain;
}
@media (min-width: 480px) {
  .spread-member-navbar__logo--full { display: block; width: 150px; }
  .spread-member-navbar__logo--mark { display: none; }
}
@media (min-width: 1024px) {
  .spread-member-navbar__logo--full { width: 170px; }
}

/* Hamburger */
.spread-member-navbar__hamburger {
  display: flex; flex-direction: column; justify-content: center;
  gap: 5px; width: 44px; height: 44px; padding: 10px 8px;
  background: none; border: none; cursor: pointer; flex-shrink: 0;
}
@media (min-width: 1024px) { .spread-member-navbar__hamburger { display: none; } }
.spread-member-navbar__hamburger-line {
  display: block; width: 100%; height: 2px;
  background: var(--spread-white); border-radius: 1px;
  transition: transform 0.2s ease, opacity 0.2s ease;
}
.spread-member-navbar--mobile-open .spread-member-navbar__hamburger-line:nth-child(1) { transform: translateY(7px) rotate(45deg); }
.spread-member-navbar--mobile-open .spread-member-navbar__hamburger-line:nth-child(2) { opacity: 0; }
.spread-member-navbar--mobile-open .spread-member-navbar__hamburger-line:nth-child(3) { transform: translateY(-7px) rotate(-45deg); }

/* Icon buttons (bell, cart) */
.spread-member-navbar__icon-btn {
  position: relative; display: flex; align-items: center; justify-content: center;
  width: 40px; height: 40px; border: none;
  border-radius: var(--spread-radius-sm);
  background: rgba(255,255,255,0.08); color: rgba(255,255,255,0.85);
  cursor: pointer; transition: background 0.15s ease, color 0.15s ease; flex-shrink: 0;
}
.spread-member-navbar__icon-btn:hover { background: rgba(255,255,255,0.18); color: var(--spread-white); }
.spread-member-navbar__icon-badge {
  position: absolute; top: 3px; right: 3px; min-width: 18px; height: 18px;
  padding: 0 4px; border-radius: var(--spread-radius-full);
  background: var(--spread-accent); color: var(--spread-white);
  font-family: var(--spread-font-family); font-size: 11px; font-weight: 700;
  line-height: 18px; text-align: center; pointer-events: none;
}

/* Avatar */
.spread-member-navbar__avatar-wrap { position: relative; }
.spread-member-navbar__avatar-btn {
  display: flex; align-items: center; justify-content: center;
  width: 36px; height: 36px; border-radius: var(--spread-radius-full);
  background: rgba(255,255,255,0.15); border: 2px solid rgba(255,255,255,0.3);
  cursor: pointer; padding: 0; overflow: hidden;
  transition: border-color 0.15s ease;
}
.spread-member-navbar__avatar-btn:hover { border-color: rgba(255,255,255,0.7); }
.spread-member-navbar__avatar-img { width: 100%; height: 100%; object-fit: cover; }
.spread-member-navbar__avatar-initials {
  font-family: var(--spread-font-family); font-size: 13px; font-weight: 700;
  color: var(--spread-white); letter-spacing: 0.05em;
}

/* User popover */
.spread-member-navbar__user-popover {
  position: absolute; top: calc(100% + 8px); right: 0;
  min-width: 220px; background: var(--spread-surface);
  border: 1px solid var(--spread-border-outer); border-radius: var(--spread-radius-md);
  box-shadow: var(--spread-shadow-lg); z-index: 300; overflow: hidden;
  font-family: var(--spread-font-family);
  animation: spread-member-navbar-pop-in 0.15s ease;
}
@keyframes spread-member-navbar-pop-in { from { opacity: 0; transform: translateY(-4px) scale(0.97); } to { opacity: 1; transform: none; } }
.spread-member-navbar__user-popover-meta { padding: 14px 16px; }
.spread-member-navbar__user-popover-name {
  font-size: 14px; font-weight: 600; color: var(--spread-black);
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}
.spread-member-navbar__user-popover-email {
  font-size: 12px; color: var(--spread-text-tertiary); margin-top: 3px;
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}
.spread-member-navbar__user-popover-divider {
  border: none; border-top: 1px solid var(--spread-border); margin: 0;
}
.spread-member-navbar__user-popover-item {
  display: block; width: 100%; padding: 11px 16px;
  font-family: var(--spread-font-family); font-size: 14px; font-weight: 500;
  color: var(--spread-text-secondary); background: none; border: none;
  cursor: pointer; text-align: left; transition: background 0.12s ease;
}
.spread-member-navbar__user-popover-item:hover { background: var(--spread-background); color: var(--spread-black); }
.spread-member-navbar__user-popover-signout {
  display: block; width: 100%; padding: 11px 16px;
  font-family: var(--spread-font-family); font-size: 14px; font-weight: 500;
  color: #B91C1C; background: none; border: none;
  cursor: pointer; text-align: left; transition: background 0.12s ease;
}
.spread-member-navbar__user-popover-signout:hover { background: #FEF2F2; }

/* ── SIDEBAR ─────────────────────────────────────────────────────────── */
.spread-member-navbar__sidebar { display: none; }

@media (min-width: 768px) {
  .spread-member-navbar__sidebar {
    display: flex; flex-direction: column; position: fixed;
    top: var(--spread-topbar-height-mobile); left: 0; bottom: 0;
    width: var(--spread-sidebar-collapsed); background: var(--spread-primary);
    z-index: 190; overflow: hidden; transition: width 0.22s ease;
    font-family: var(--spread-font-family);
  }
  .spread-member-navbar--sidebar-open .spread-member-navbar__sidebar {
    width: var(--spread-sidebar-expanded);
  }
}
@media (min-width: 1024px) {
  .spread-member-navbar__sidebar { top: var(--spread-topbar-height); }
  .spread-member-navbar__sidebar:hover { width: var(--spread-sidebar-expanded); }
}

.spread-member-navbar__have-your-say-wrap {
  padding: 8px 0 4px; border-bottom: 1px solid rgba(255,255,255,0.1); flex-shrink: 0;
}
.spread-member-navbar__have-your-say {
  display: flex; align-items: center; gap: 12px; width: 100%; height: 44px;
  padding: 0 0 0 19px; background: rgba(206,102,50,0.15);
  border: none; cursor: pointer; color: var(--spread-sand-light);
  white-space: nowrap; overflow: hidden; transition: background 0.15s ease;
}
.spread-member-navbar__have-your-say:hover { background: rgba(206,102,50,0.28); color: var(--spread-white); }
.spread-member-navbar__have-your-say svg { stroke: currentColor; flex-shrink: 0; }

.spread-member-navbar__sidebar-list {
  flex: 1; overflow-y: auto; overflow-x: hidden; padding: 8px 0;
  scrollbar-width: thin; scrollbar-color: rgba(255,255,255,0.15) transparent;
}
.spread-member-navbar__sidebar-item { width: 100%; }
.spread-member-navbar__sidebar-link {
  display: flex; align-items: center; gap: 12px; width: 100%; height: 48px;
  padding: 0 0 0 19px; background: none; border: none; cursor: pointer;
  color: rgba(255,255,255,0.7); text-align: left;
  transition: color 0.15s ease, background 0.15s ease;
  white-space: nowrap; overflow: hidden;
}
.spread-member-navbar__sidebar-link:hover { color: var(--spread-white); background: rgba(255,255,255,0.08); }
.spread-member-navbar__sidebar-link--active { color: var(--spread-white); background: rgba(255,255,255,0.15); }
.spread-member-navbar__sidebar-icon {
  display: flex; align-items: center; justify-content: center;
  flex-shrink: 0; width: 20px; height: 20px;
}
.spread-member-navbar__sidebar-icon svg { stroke: currentColor; }
.spread-member-navbar__sidebar-label {
  font-size: 14px; font-weight: 500; line-height: 1;
  opacity: 0; transition: opacity 0.15s ease 0.05s; pointer-events: none;
}
@media (min-width: 768px) {
  .spread-member-navbar--sidebar-open .spread-member-navbar__sidebar-label { opacity: 1; }
}
@media (min-width: 1024px) {
  .spread-member-navbar__sidebar:hover .spread-member-navbar__sidebar-label { opacity: 1; }
}

.spread-member-navbar__sidebar-footer {
  flex-shrink: 0; padding: 8px 0; border-top: 1px solid rgba(255,255,255,0.1);
}
.spread-member-navbar__sidebar-signout {
  display: flex; align-items: center; gap: 12px; width: 100%; height: 48px;
  padding: 0 0 0 19px; background: none; border: none; cursor: pointer;
  color: rgba(255,255,255,0.55); text-align: left;
  transition: color 0.15s ease, background 0.15s ease;
  white-space: nowrap; overflow: hidden; font-family: var(--spread-font-family);
}
.spread-member-navbar__sidebar-signout:hover { color: #FCA5A5; background: rgba(185,28,28,0.1); }
.spread-member-navbar__sidebar-signout svg { stroke: currentColor; flex-shrink: 0; }

/* ── MOBILE DRAWER ───────────────────────────────────────────────────── */
.spread-member-navbar__drawer { position: fixed; inset: 0; z-index: 195; }
.spread-member-navbar__drawer-backdrop {
  position: absolute; inset: 0; background: rgba(0,0,0,0.45);
  animation: spread-member-navbar-fade-in 0.2s ease;
}
@keyframes spread-member-navbar-fade-in { from { opacity: 0; } to { opacity: 1; } }
.spread-member-navbar__drawer-panel {
  position: absolute; top: 0; left: 0; bottom: 0;
  width: 300px; max-width: calc(100vw - 48px);
  background: var(--spread-primary); display: flex; flex-direction: column;
  overflow-y: auto; animation: spread-member-navbar-slide-in 0.22s ease-out;
  font-family: var(--spread-font-family);
}
@keyframes spread-member-navbar-slide-in { from { transform: translateX(-100%); } to { transform: translateX(0); } }
@media (min-width: 1024px) { .spread-member-navbar__drawer { display: none; } }

.spread-member-navbar__drawer-header {
  display: flex; align-items: center; justify-content: space-between;
  padding: 14px 16px; border-bottom: 1px solid rgba(255,255,255,0.1);
  min-height: 72px;
}
.spread-member-navbar__drawer-member { display: flex; align-items: center; gap: 12px; min-width: 0; }
.spread-member-navbar__drawer-avatar {
  width: 40px; height: 40px; border-radius: var(--spread-radius-full);
  object-fit: cover; flex-shrink: 0;
}
.spread-member-navbar__drawer-avatar-initials {
  width: 40px; height: 40px; border-radius: var(--spread-radius-full);
  background: rgba(255,255,255,0.15); display: flex; align-items: center; justify-content: center;
  font-size: 14px; font-weight: 700; color: var(--spread-white); flex-shrink: 0;
}
.spread-member-navbar__drawer-name {
  font-size: 14px; font-weight: 600; color: var(--spread-white);
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}
.spread-member-navbar__drawer-email {
  font-size: 12px; color: rgba(255,255,255,0.6); margin-top: 2px;
  overflow: hidden; text-overflow: ellipsis; white-space: nowrap;
}
.spread-member-navbar__drawer-close {
  display: flex; align-items: center; justify-content: center;
  width: 32px; height: 32px; background: rgba(255,255,255,0.08);
  border: none; border-radius: var(--spread-radius-sm);
  color: rgba(255,255,255,0.7); cursor: pointer; flex-shrink: 0;
  transition: background 0.12s ease;
}
.spread-member-navbar__drawer-close:hover { background: rgba(255,255,255,0.18); color: var(--spread-white); }

.spread-member-navbar__drawer-have-your-say { padding: 10px 16px; border-bottom: 1px solid rgba(255,255,255,0.08); }
.spread-member-navbar__drawer-hys-btn {
  display: flex; align-items: center; gap: 8px; width: 100%; padding: 10px 12px;
  background: rgba(206,102,50,0.18); border: none;
  border-radius: var(--spread-radius-sm); font-family: var(--spread-font-family);
  font-size: 14px; font-weight: 600; color: var(--spread-sand-light);
  cursor: pointer; transition: background 0.12s ease;
}
.spread-member-navbar__drawer-hys-btn:hover { background: rgba(206,102,50,0.3); color: var(--spread-white); }
.spread-member-navbar__drawer-hys-btn svg { stroke: currentColor; flex-shrink: 0; }

.spread-member-navbar__drawer-list { flex: 1; padding: 8px 0; }
.spread-member-navbar__drawer-link {
  display: flex; align-items: center; gap: 12px; width: 100%; padding: 13px 20px;
  background: none; border: none; font-family: var(--spread-font-family);
  font-size: 15px; font-weight: 500; color: rgba(255,255,255,0.8);
  cursor: pointer; text-align: left; transition: background 0.12s ease, color 0.12s ease;
}
.spread-member-navbar__drawer-link:hover { background: rgba(255,255,255,0.08); color: var(--spread-white); }
.spread-member-navbar__drawer-link--active { color: var(--spread-white); background: rgba(255,255,255,0.15); }
.spread-member-navbar__drawer-icon { display: flex; align-items: center; flex-shrink: 0; }
.spread-member-navbar__drawer-icon svg { stroke: currentColor; }

.spread-member-navbar__drawer-footer { padding: 16px; border-top: 1px solid rgba(255,255,255,0.1); }
.spread-member-navbar__drawer-signout {
  display: block; width: 100%; font-family: var(--spread-font-family);
  font-size: 14px; font-weight: 600; padding: 12px;
  border-radius: var(--spread-radius-md); cursor: pointer; text-align: center;
  background: rgba(185,28,28,0.15); color: #FCA5A5; border: 1px solid rgba(185,28,28,0.3);
  transition: background 0.15s ease;
}
.spread-member-navbar__drawer-signout:hover { background: rgba(185,28,28,0.28); color: #FEE2E2; }

/* ── CERANOVANAV BADGE ───────────────────────────────────────────────── */
.spread-member-navbar__ceranovanav {
  display: flex; align-items: center; gap: 10px;
  padding: 11px 12px 11px 13px; margin-top: auto; flex-shrink: 0;
  border-top: 1px solid rgba(255,255,255,0.1);
  text-decoration: none; overflow: hidden; white-space: nowrap;
  transition: background 0.15s ease;
}
.spread-member-navbar__ceranovanav:hover { background: rgba(255,255,255,0.06); }
.spread-member-navbar__ceranovanav-content {
  display: flex; flex-direction: column; flex-shrink: 0;
  opacity: 0; pointer-events: none; transition: opacity 0.15s ease 0.05s;
}
.spread-member-navbar__ceranovanav-icon {
  flex-shrink: 0; width: 32px; height: 32px; opacity: 0.5; border-radius: 6px; transition: opacity 0.2s ease;
}
.spread-member-navbar__ceranovanav-built-by {
  font-size: 10px; font-weight: 400; color: #9ca3af;
  line-height: 1.3; font-family: var(--spread-font-family);
}
.spread-member-navbar__ceranovanav-brand {
  font-size: 12px; font-weight: 600; line-height: 1.3;
  font-family: var(--spread-font-family);
}
.spread-member-navbar__ceranovanav-cera { color: #e6d8ca; }
.spread-member-navbar__ceranovanav-nova { color: #c0392b; }
@media (min-width: 768px) {
  .spread-member-navbar--sidebar-open .spread-member-navbar__ceranovanav-content { opacity: 1; }
  .spread-member-navbar--sidebar-open .spread-member-navbar__ceranovanav-icon { opacity: 0.85; }
}
@media (min-width: 1024px) {
  .spread-member-navbar__sidebar:hover .spread-member-navbar__ceranovanav-content { opacity: 1; }
  .spread-member-navbar__sidebar:hover .spread-member-navbar__ceranovanav-icon { opacity: 0.85; }
}
/* Mobile drawer CeraNova badge */
.spread-member-navbar__drawer-ceranovanav {
  display: flex; align-items: center; gap: 10px;
  padding: 11px 16px; flex-shrink: 0; margin-top: auto;
  border-top: 1px solid rgba(255,255,255,0.1);
  text-decoration: none; transition: background 0.15s ease;
}
.spread-member-navbar__drawer-ceranovanav:hover { background: rgba(255,255,255,0.06); }
.spread-member-navbar__drawer-ceranovanav .spread-member-navbar__ceranovanav-icon { opacity: 0.85; }
.spread-member-navbar__drawer-ceranovanav-content {
  display: flex; flex-direction: column;
}

/* ── Theme toggle ──────────────────────────────────────────────────────── */
.spread-member-navbar__theme-toggle {
  border: 1.5px solid rgba(255, 255, 255, 0.25);
  transition: border-color 0.15s ease, color 0.15s ease, background 0.15s ease;
}
.spread-member-navbar__theme-toggle--dark {
  border-color: #bead38;
  color: #bead38 !important;
  background: rgba(190, 173, 56, 0.12) !important;
}
.spread-member-navbar__theme-toggle--light {
  border-color: rgba(255, 200, 100, 0.55);
  color: rgba(255, 220, 130, 0.9) !important;
}
.spread-member-navbar__theme-toggle:hover {
  border-color: rgba(255, 255, 255, 0.5) !important;
}

/* ── Dark mode overrides (light surfaces) ─────────────────────────────── */
/* User popover */
:global(html.dark) .spread-member-navbar__user-popover {
  background: #1a0f14;
  border-color: rgba(230, 216, 202, 0.15);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.6);
}
:global(html.dark) .spread-member-navbar__user-popover-name {
  color: #e6d8ca;
}
:global(html.dark) .spread-member-navbar__user-popover-email {
  color: rgba(230, 216, 202, 0.55);
}
:global(html.dark) .spread-member-navbar__user-popover-divider {
  border-top-color: rgba(230, 216, 202, 0.12);
}
:global(html.dark) .spread-member-navbar__user-popover-item {
  color: #e6d8ca;
}
:global(html.dark) .spread-member-navbar__user-popover-item:hover {
  background: rgba(230, 216, 202, 0.07);
  color: #e6d8ca;
}
:global(html.dark) .spread-member-navbar__user-popover-signout {
  color: #f87171;
}
:global(html.dark) .spread-member-navbar__user-popover-signout:hover {
  background: rgba(248, 113, 113, 0.1);
}
/* Mobile drawer header name/email (drawer bg is Tyrian — stays) */
:global(html.dark) .spread-member-navbar__drawer-name {
  color: #e6d8ca;
}
:global(html.dark) .spread-member-navbar__drawer-email {
  color: rgba(230, 216, 202, 0.55);
}
</style>
