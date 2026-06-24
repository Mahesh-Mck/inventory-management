<!--
  Reference shell for client/src/App.vue after the SaaS redesign.

  This shows the LAYOUT REGION only (template shell, nav data, and the
  layout-related styles). Copy the structure, not verbatim — keep App.vue's
  existing task/modal logic (loadTasks, addTask, deleteTask, toggleTask) and
  the global utility classes (.card, .badge, table, etc.) intact.
-->
<template>
  <div class="app">
    <!-- Mobile-only top bar; hidden above 768px -->
    <div class="mobile-bar">
      <button class="hamburger" @click="sidebarOpen = !sidebarOpen" aria-label="Toggle navigation">
        <MenuIcon :size="22" />
      </button>
      <span class="mobile-brand">{{ t('nav.companyName') }}</span>
    </div>

    <div v-if="sidebarOpen" class="sidebar-backdrop" @click="sidebarOpen = false"></div>

    <aside class="sidebar" :class="{ open: sidebarOpen }">
      <div class="sidebar-brand">
        <span class="brand-mark">IM</span>
        <div class="brand-text">
          <h1>{{ t('nav.companyName') }}</h1>
          <span class="subtitle">{{ t('nav.subtitle') }}</span>
        </div>
      </div>

      <nav class="sidebar-nav">
        <router-link
          v-for="item in navItems"
          :key="item.path"
          :to="item.path"
          class="nav-item"
          :class="{ active: $route.path === item.path }"
          @click="sidebarOpen = false"
        >
          <component :is="item.icon" :size="18" class="nav-icon" />
          <span class="nav-label">{{ item.label }}</span>
        </router-link>
      </nav>

      <div class="sidebar-footer">
        <LanguageSwitcher />
        <ProfileMenu
          @show-profile-details="showProfileDetails = true"
          @show-tasks="showTasks = true"
        />
      </div>
    </aside>

    <div class="content-area">
      <FilterBar />
      <main class="main-content">
        <router-view />
      </main>
    </div>

    <!-- Global modals remain rendered here, unchanged -->
  </div>
</template>

<script>
import { ref, computed, markRaw } from 'vue'
import {
  LayoutDashboard, Package, ShoppingCart, Wallet, TrendingUp, FileBarChart,
  Menu as MenuIcon
} from 'lucide-vue-next'
import { useI18n } from './composables/useI18n'
// ...keep the rest of the existing imports (useAuth, FilterBar, ProfileMenu, modals, LanguageSwitcher)

export default {
  name: 'App',
  components: { /* ...existing..., */ MenuIcon },
  setup() {
    const { t } = useI18n()
    const sidebarOpen = ref(false)

    const navItems = computed(() => [
      { path: '/', label: t('nav.overview'), icon: markRaw(LayoutDashboard) },
      { path: '/inventory', label: t('nav.inventory'), icon: markRaw(Package) },
      { path: '/orders', label: t('nav.orders'), icon: markRaw(ShoppingCart) },
      { path: '/spending', label: t('nav.finance'), icon: markRaw(Wallet) },
      { path: '/demand', label: t('nav.demandForecast'), icon: markRaw(TrendingUp) },
      { path: '/reports', label: 'Reports', icon: markRaw(FileBarChart) }
    ])

    // ...keep showProfileDetails, showTasks, tasks, addTask, deleteTask, toggleTask...

    return { t, sidebarOpen, navItems /* , ...the rest */ }
  }
}
</script>

<style>
/* Layout region only — the global utility classes below this in the real
   App.vue (.card, .badge, .stat-card, table, .loading, .error) stay as-is,
   migrated to tokens where values map exactly. */

.app { display: flex; flex-direction: row; min-height: 100vh; }

.sidebar {
  width: var(--sidebar-width);
  flex-shrink: 0;
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border-right: 1px solid var(--color-border);
  position: sticky;
  top: 0;
  height: 100vh;
  z-index: var(--z-sidebar);
}

.sidebar-brand {
  display: flex; align-items: center; gap: var(--space-3);
  padding: var(--space-6) var(--space-5);
  border-bottom: 1px solid var(--color-border);
}
.brand-mark {
  display: flex; align-items: center; justify-content: center;
  width: 38px; height: 38px; flex-shrink: 0;
  border-radius: var(--radius-md);
  background: linear-gradient(135deg, var(--color-primary), var(--color-primary-hover));
  color: #fff; font-weight: 700; font-size: 0.95rem; letter-spacing: -0.03em;
}
.brand-text { display: flex; flex-direction: column; min-width: 0; }
.brand-text h1 {
  font-size: 1.0625rem; font-weight: 700; color: var(--color-text);
  letter-spacing: -0.025em; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;
}
.brand-text .subtitle { font-size: 0.75rem; color: var(--color-text-muted); font-weight: 400; }

.sidebar-nav {
  flex: 1; display: flex; flex-direction: column; gap: 2px;
  padding: var(--space-4) var(--space-3); overflow-y: auto;
}
.nav-item {
  display: flex; align-items: center; gap: var(--space-3);
  padding: 0.625rem var(--space-3); border-radius: var(--radius-sm);
  color: var(--color-text-muted); text-decoration: none;
  font-weight: 500; font-size: 0.9rem; position: relative;
  transition: var(--transition-base);
}
.nav-item:hover { color: var(--color-text); background: var(--color-bg-subtle); }
.nav-item.active { color: var(--color-primary); background: var(--color-primary-soft); }
.nav-item.active::before {
  content: ''; position: absolute; left: 0; top: 50%; transform: translateY(-50%);
  width: 3px; height: 60%; border-radius: 0 var(--radius-full) var(--radius-full) 0;
  background: var(--color-primary);
}
.nav-icon { flex-shrink: 0; }
.nav-label { white-space: nowrap; }

.sidebar-footer {
  border-top: 1px solid var(--color-border);
  padding: var(--space-3);
  display: flex; flex-direction: column; gap: var(--space-2);
}

.content-area { flex: 1; display: flex; flex-direction: column; min-width: 0; }
.main-content { flex: 1; width: 100%; padding: var(--space-6) var(--space-8); }

.mobile-bar { display: none; }
.sidebar-backdrop { display: none; }

@media (max-width: 768px) {
  .app { flex-direction: column; }
  .mobile-bar {
    display: flex; align-items: center; gap: var(--space-3);
    position: sticky; top: 0; z-index: var(--z-sidebar);
    height: 56px; padding: var(--space-3) var(--space-4);
    background: var(--color-surface); border-bottom: 1px solid var(--color-border);
  }
  .hamburger {
    display: flex; align-items: center; justify-content: center; padding: var(--space-1);
    background: none; border: none; color: var(--color-text); cursor: pointer;
  }
  .mobile-brand { font-weight: 700; color: var(--color-text); letter-spacing: -0.025em; }
  .sidebar {
    position: fixed; top: 0; left: 0; height: 100vh;
    transform: translateX(-100%); transition: transform 0.25s ease; box-shadow: var(--shadow-lg);
  }
  .sidebar.open { transform: translateX(0); }
  .sidebar-backdrop {
    display: block; position: fixed; inset: 0;
    background: rgba(15, 23, 42, 0.5); z-index: calc(var(--z-sidebar) - 1);
  }
  .main-content { padding: var(--space-5) var(--space-4); }
}
</style>
