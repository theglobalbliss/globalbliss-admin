<script setup>
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const form = ref({
  title: "",
  description: "",
  icon: "bi-briefcase",
  service_url: "",
  is_active: true,
  sort_order: 5,
});

const isSaving = ref(false);
const successMessage = ref("");
const errorMessage = ref("");

const iconOptions = [
  "bi-briefcase",
  "bi-palette",
  "bi-code-slash",
  "bi-instagram",
  "bi-stars",
  "bi-globe",
  "bi-megaphone",
  "bi-camera",
  "bi-laptop",
  "bi-lightbulb",
  "bi-brush",
  "bi-phone",
];

const logout = () => {
  localStorage.removeItem("globalbliss_admin_logged_in");
  navigateTo("/login");
};

const addService = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase.from("services").insert([
    {
      title: form.value.title,
      description: form.value.description,
      icon: form.value.icon,
      service_url: form.value.service_url,
      is_active: form.value.is_active,
      sort_order: Number(form.value.sort_order),
      updated_at: new Date().toISOString(),
    },
  ]);

  if (error) {
    errorMessage.value = error.message;
    isSaving.value = false;
    return;
  }

  successMessage.value = "Service added successfully.";

  setTimeout(() => {
    navigateTo("/services");
  }, 900);

  isSaving.value = false;
};

onMounted(() => {
  const isLoggedIn = localStorage.getItem("globalbliss_admin_logged_in");

  if (!isLoggedIn) {
    navigateTo("/login");
  }
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
          <h5 class="fw-bold mb-1">Add Service</h5>
          <p class="text-muted mb-0">
            Create a new service for your portfolio website.
          </p>
        </div>

        <NuxtLink to="/services" class="btn btn-outline-dark">
          Back to Services
        </NuxtLink>
      </div>

      <div class="admin-card">
        <form @submit.prevent="addService">
          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">Service Title</label>
              <input
                v-model="form.title"
                type="text"
                class="form-control"
                placeholder="e.g. Website Design"
                required
              />
            </div>

            <div class="col-md-3">
              <label class="form-label">Icon</label>
              <select v-model="form.icon" class="form-select">
                <option v-for="icon in iconOptions" :key="icon" :value="icon">
                  {{ icon }}
                </option>
              </select>
            </div>

            <div class="col-md-3">
              <label class="form-label">Sort Order</label>
              <input
                v-model="form.sort_order"
                type="number"
                class="form-control"
                min="1"
              />
            </div>

            <div class="col-md-12">
              <label class="form-label">Service URL</label>
              <input
                v-model="form.service_url"
                type="text"
                class="form-control"
                placeholder="#contact or https://..."
              />
            </div>

            <div class="col-md-12">
              <label class="form-label">Description</label>
              <textarea
                v-model="form.description"
                class="form-control"
                rows="5"
                placeholder="Describe this service clearly"
              ></textarea>
            </div>

            <div class="col-md-12">
              <div class="form-check">
                <input
                  v-model="form.is_active"
                  class="form-check-input"
                  type="checkbox"
                  id="isActive"
                />
                <label class="form-check-label" for="isActive">
                  Show this service on portfolio website
                </label>
              </div>
            </div>

            <div v-if="successMessage" class="col-12">
              <div class="alert alert-success">
                {{ successMessage }}
              </div>
            </div>

            <div v-if="errorMessage" class="col-12">
              <div class="alert alert-danger">
                {{ errorMessage }}
              </div>
            </div>

            <div class="col-12">
              <button type="submit" class="btn btn-primary" :disabled="isSaving">
                {{ isSaving ? "Saving..." : "Add Service" }}
              </button>
            </div>
          </div>
        </form>
      </div>
    </main>
  </div>
</template>