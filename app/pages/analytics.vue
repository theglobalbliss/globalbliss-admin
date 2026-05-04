<script setup>
import { createClient } from "@supabase/supabase-js";

definePageMeta({
  layout: "admin",
});

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const pageViews = ref([]);
const totalViews = ref(0);
const uniqueSessions = ref(0);
const todayViews = ref(0);
const mobileViews = ref(0);
const isLoading = ref(true);
const errorMessage = ref("");

const formatDate = (date) => {
  if (!date) return "";

  return new Date(date).toLocaleString("en-NG", {
    year: "numeric",
    month: "short",
    day: "numeric",
    hour: "2-digit",
    minute: "2-digit",
  });
};

const fetchAnalytics = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("page_views")
    .select("*")
    .order("created_at", { ascending: false })
    .limit(200);

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  pageViews.value = data || [];
  totalViews.value = pageViews.value.length;

  const sessions = new Set(pageViews.value.map((view) => view.session_id).filter(Boolean));
  uniqueSessions.value = sessions.size;

  const today = new Date().toISOString().slice(0, 10);

  todayViews.value = pageViews.value.filter((view) =>
    view.created_at?.startsWith(today)
  ).length;

  mobileViews.value = pageViews.value.filter(
    (view) => view.device_type === "Mobile"
  ).length;

  isLoading.value = false;
};

const topPages = computed(() => {
  const counts = {};

  pageViews.value.forEach((view) => {
    const path = view.page_path || "/";
    counts[path] = (counts[path] || 0) + 1;
  });

  return Object.entries(counts)
    .map(([path, count]) => ({ path, count }))
    .sort((a, b) => b.count - a.count)
    .slice(0, 8);
});

const deviceStats = computed(() => {
  const counts = {};

  pageViews.value.forEach((view) => {
    const device = view.device_type || "Unknown";
    counts[device] = (counts[device] || 0) + 1;
  });

  return Object.entries(counts)
    .map(([device, count]) => ({ device, count }))
    .sort((a, b) => b.count - a.count);
});

const browserStats = computed(() => {
  const counts = {};

  pageViews.value.forEach((view) => {
    const browser = view.browser || "Unknown";
    counts[browser] = (counts[browser] || 0) + 1;
  });

  return Object.entries(counts)
    .map(([browser, count]) => ({ browser, count }))
    .sort((a, b) => b.count - a.count);
});

onMounted(() => {
  fetchAnalytics();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Site Analytics</h5>
        <p class="text-muted mb-0">
          Track portfolio visits, popular pages, devices, browsers, and recent activity.
        </p>
      </div>

      <button class="btn btn-outline-dark" @click="fetchAnalytics">
        <i class="bi bi-arrow-clockwise me-2"></i>
        Refresh
      </button>
    </div>

    <div v-if="errorMessage" class="alert alert-danger">
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="admin-card">
      <div class="d-flex align-items-center gap-3">
        <div class="spinner-border text-primary" role="status"></div>
        <p class="mb-0 text-muted">Loading analytics...</p>
      </div>
    </div>

    <template v-else>
      <div class="row g-4 mb-4">
        <div class="col-md-3">
          <div class="admin-card stat-card">
            <div class="stat-icon">
              <i class="bi bi-eye"></i>
            </div>
            <p class="stat-value">{{ totalViews }}</p>
            <p class="stat-label">Recent Page Views</p>
          </div>
        </div>

        <div class="col-md-3">
          <div class="admin-card stat-card">
            <div class="stat-icon">
              <i class="bi bi-people"></i>
            </div>
            <p class="stat-value">{{ uniqueSessions }}</p>
            <p class="stat-label">Unique Sessions</p>
          </div>
        </div>

        <div class="col-md-3">
          <div class="admin-card stat-card">
            <div class="stat-icon">
              <i class="bi bi-calendar-day"></i>
            </div>
            <p class="stat-value">{{ todayViews }}</p>
            <p class="stat-label">Today’s Views</p>
          </div>
        </div>

        <div class="col-md-3">
          <div class="admin-card stat-card">
            <div class="stat-icon">
              <i class="bi bi-phone"></i>
            </div>
            <p class="stat-value">{{ mobileViews }}</p>
            <p class="stat-label">Mobile Views</p>
          </div>
        </div>
      </div>

      <div class="row g-4 mb-4">
        <div class="col-lg-6">
          <div class="admin-card h-100">
            <h5 class="fw-bold mb-3">Top Pages</h5>

            <div v-if="topPages.length" class="d-grid gap-3">
              <div
                v-for="page in topPages"
                :key="page.path"
                class="d-flex justify-content-between align-items-center p-3 rounded-4"
                style="background: rgba(17, 17, 17, 0.04);"
              >
                <div>
                  <p class="fw-semibold mb-0">{{ page.path }}</p>
                  <small class="text-muted">Page path</small>
                </div>

                <span class="badge bg-primary">
                  {{ page.count }} views
                </span>
              </div>
            </div>

            <p v-else class="text-muted mb-0">
              No page views recorded yet.
            </p>
          </div>
        </div>

        <div class="col-lg-3">
          <div class="admin-card h-100">
            <h5 class="fw-bold mb-3">Devices</h5>

            <div v-if="deviceStats.length" class="d-grid gap-2">
              <div
                v-for="device in deviceStats"
                :key="device.device"
                class="d-flex justify-content-between"
              >
                <span>{{ device.device }}</span>
                <strong>{{ device.count }}</strong>
              </div>
            </div>

            <p v-else class="text-muted mb-0">
              No device data yet.
            </p>
          </div>
        </div>

        <div class="col-lg-3">
          <div class="admin-card h-100">
            <h5 class="fw-bold mb-3">Browsers</h5>

            <div v-if="browserStats.length" class="d-grid gap-2">
              <div
                v-for="browser in browserStats"
                :key="browser.browser"
                class="d-flex justify-content-between"
              >
                <span>{{ browser.browser }}</span>
                <strong>{{ browser.count }}</strong>
              </div>
            </div>

            <p v-else class="text-muted mb-0">
              No browser data yet.
            </p>
          </div>
        </div>
      </div>

      <div class="admin-card">
        <h5 class="fw-bold mb-3">Recent Visits</h5>

        <div v-if="pageViews.length" class="table-responsive">
          <table class="table table-hover align-middle">
            <thead>
              <tr>
                <th>Page</th>
                <th>Title</th>
                <th>Device</th>
                <th>Browser</th>
                <th>Referrer</th>
                <th>Date</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="view in pageViews" :key="view.id">
                <td>
                  <code>{{ view.page_path }}</code>
                </td>

                <td>
                  {{ view.page_title || "Not available" }}
                </td>

                <td>
                  {{ view.device_type || "Unknown" }}
                </td>

                <td>
                  {{ view.browser || "Unknown" }}
                </td>

                <td style="max-width: 220px;">
                  <span class="text-muted">
                    {{ view.referrer || "Direct" }}
                  </span>
                </td>

                <td>
                  {{ formatDate(view.created_at) }}
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <p v-else class="text-muted mb-0">
          No visits recorded yet. Open your portfolio website and refresh this page.
        </p>
      </div>
    </template>
  </div>
</template>