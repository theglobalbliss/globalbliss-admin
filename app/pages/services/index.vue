<script setup>
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const services = ref([]);
const isLoading = ref(true);
const successMessage = ref("");
const errorMessage = ref("");

const getAdminIcon = (icon) => {
  const iconMap = {
    "ri-palette-line": "bi bi-palette",
    "ri-quill-pen-line": "bi bi-vector-pen",
    "ri-code-s-slash-line": "bi bi-code-slash",
    "ri-global-fill": "bi bi-globe2",
    "ri-instagram-line": "bi bi-instagram",
    "ri-pantone-fill": "bi bi-palette2",
    "ri-magic-line": "bi bi-stars",
    "ri-service-line": "bi bi-briefcase",
  };

  if (!icon) return "bi bi-briefcase";

  if (icon.startsWith("bi ")) return icon;

  if (icon.startsWith("bi-")) return `bi ${icon}`;

  return iconMap[icon] || "bi bi-briefcase";
};

const logout = () => {
  localStorage.removeItem("globalbliss_admin_logged_in");
  navigateTo("/login");
};

const fetchServices = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("services")
    .select("*")
    .order("sort_order", { ascending: true });

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  services.value = data || [];
  isLoading.value = false;
};

const deleteService = async (service) => {
  const confirmed = confirm(
    `Are you sure you want to delete "${service.title}"?`
  );

  if (!confirmed) return;

  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("services")
    .delete()
    .eq("id", service.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = "Service deleted successfully.";
  await fetchServices();
};

onMounted(() => {
  const isLoggedIn = localStorage.getItem("globalbliss_admin_logged_in");

  if (!isLoggedIn) {
    navigateTo("/login");
    return;
  }

  fetchServices();
});
</script>

<template>
  <div>
    <aside class="admin-sidebar">
      <div class="admin-brand">
        The GlobalBliss<br />
        <span class="text-primary">Admin</span>
        <small>Portfolio Control Center</small>
      </div>

      <NuxtLink to="/dashboard">
        <i class="bi bi-grid"></i>
        Dashboard
      </NuxtLink>

      <NuxtLink to="/projects">
        <i class="bi bi-folder2-open"></i>
        Projects
      </NuxtLink>

      <NuxtLink to="/projects/add">
        <i class="bi bi-plus-circle"></i>
        Add Project
      </NuxtLink>

      <NuxtLink to="/services">
        <i class="bi bi-briefcase"></i>
        Services
      </NuxtLink>

      <NuxtLink to="/services/add">
        <i class="bi bi-plus-square"></i>
        Add Service
      </NuxtLink>

      <NuxtLink to="/content/homepage">
        <i class="bi bi-pencil-square"></i>
        Homepage Content
      </NuxtLink>

      <a href="#" class="logout-link" @click.prevent="logout">
        <i class="bi bi-box-arrow-left"></i>
        Logout
      </a>
    </aside>

    <main class="admin-main">
      <div class="admin-topbar d-flex justify-content-between align-items-center">
        <div>
          <h5 class="fw-bold mb-1">Services Manager</h5>
          <p class="text-muted mb-0">
            Manage the services shown on The GlobalBliss portfolio website.
          </p>
        </div>

        <NuxtLink to="/services/add" class="btn btn-primary">
          <i class="bi bi-plus-circle me-2"></i>
          Add Service
        </NuxtLink>
      </div>

      <div v-if="successMessage" class="alert alert-success">
        {{ successMessage }}
      </div>

      <div v-if="isLoading" class="admin-card">
        Loading services...
      </div>

      <div v-else-if="errorMessage" class="admin-card text-danger">
        {{ errorMessage }}
      </div>

      <div v-else class="admin-card">
        <div class="table-responsive">
          <table class="table table-hover align-middle">
            <thead>
              <tr>
                <th>Icon</th>
                <th>Title</th>
                <th>Description</th>
                <th>Order</th>
                <th>Status</th>
                <th class="text-end">Actions</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="service in services" :key="service.id">
                <td>
                  <div class="stat-icon mb-0">
                    <i :class="getAdminIcon(service.icon)"></i>
                  </div>
                </td>

                <td class="fw-semibold">
                  {{ service.title }}
                </td>

                <td style="max-width: 420px;">
                  {{ service.description }}
                </td>

                <td>
                  {{ service.sort_order }}
                </td>

                <td>
                  <span
                    class="badge"
                    :class="service.is_active ? 'bg-success' : 'bg-secondary'"
                  >
                    {{ service.is_active ? "Visible" : "Hidden" }}
                  </span>
                </td>

                <td class="text-end">
                  <NuxtLink
                    :to="`/services/edit/${service.id}`"
                    class="btn btn-sm btn-outline-dark me-2"
                  >
                    Edit
                  </NuxtLink>

                  <button
                    class="btn btn-sm btn-outline-danger"
                    @click="deleteService(service)"
                  >
                    Delete
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <p v-if="services.length === 0" class="text-muted mb-0">
          No services found yet.
        </p>
      </div>
    </main>
  </div>
</template>