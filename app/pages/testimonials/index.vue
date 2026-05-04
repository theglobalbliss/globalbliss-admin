<script setup>
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const testimonials = ref([]);
const isLoading = ref(true);
const successMessage = ref("");
const errorMessage = ref("");

const logout = () => {
  localStorage.removeItem("globalbliss_admin_logged_in");
  navigateTo("/login");
};

const getImageUrl = (imagePath) => {
  if (!imagePath) return "";

  if (imagePath.startsWith("http")) {
    return imagePath;
  }

  return `/admin${imagePath}`;
};

const fetchTestimonials = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("testimonials")
    .select("*")
    .order("sort_order", { ascending: true });

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  testimonials.value = data || [];
  isLoading.value = false;
};

const deleteTestimonial = async (testimonial) => {
  const confirmed = confirm(
    `Are you sure you want to delete "${testimonial.name}"?`
  );

  if (!confirmed) return;

  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("testimonials")
    .delete()
    .eq("id", testimonial.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = "Testimonial deleted successfully.";
  await fetchTestimonials();
};

onMounted(() => {
  const isLoggedIn = localStorage.getItem("globalbliss_admin_logged_in");

  if (!isLoggedIn) {
    navigateTo("/login");
    return;
  }

  fetchTestimonials();
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

      <NuxtLink to="/testimonials">
        <i class="bi bi-chat-quote"></i>
        Testimonials
      </NuxtLink>

      <NuxtLink to="/testimonials/add">
        <i class="bi bi-plus-circle-dotted"></i>
        Add Testimonial
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
          <h5 class="fw-bold mb-1">Testimonials Manager</h5>
          <p class="text-muted mb-0">
            Manage reviews and client feedback shown on your portfolio website.
          </p>
        </div>

        <NuxtLink to="/testimonials/add" class="btn btn-primary">
          <i class="bi bi-plus-circle me-2"></i>
          Add Testimonial
        </NuxtLink>
      </div>

      <div v-if="successMessage" class="alert alert-success">
        {{ successMessage }}
      </div>

      <div v-if="isLoading" class="admin-card">
        Loading testimonials...
      </div>

      <div v-else-if="errorMessage" class="admin-card text-danger">
        {{ errorMessage }}
      </div>

      <div v-else class="admin-card">
        <div class="table-responsive">
          <table class="table table-hover align-middle">
            <thead>
              <tr>
                <th>Photo</th>
                <th>Name</th>
                <th>Role</th>
                <th>Message</th>
                <th>Rating</th>
                <th>Order</th>
                <th>Status</th>
                <th class="text-end">Actions</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="testimonial in testimonials" :key="testimonial.id">
                <td>
                  <img
                    v-if="testimonial.image_url"
                    :src="getImageUrl(testimonial.image_url)"
                    :alt="testimonial.name"
                    style="width: 56px; height: 56px; object-fit: cover; border-radius: 50%;"
                  />

                  <div
                    v-else
                    class="stat-icon mb-0"
                    style="width: 56px; height: 56px;"
                  >
                    <i class="bi bi-person"></i>
                  </div>
                </td>

                <td class="fw-semibold">
                  {{ testimonial.name }}
                  <div v-if="testimonial.company" class="small text-muted">
                    {{ testimonial.company }}
                  </div>
                </td>

                <td>
                  {{ testimonial.role }}
                </td>

                <td style="max-width: 420px;">
                  {{ testimonial.message }}
                </td>

                <td>
                  {{ testimonial.rating }}/5
                </td>

                <td>
                  {{ testimonial.sort_order }}
                </td>

                <td>
                  <span
                    class="badge"
                    :class="testimonial.is_active ? 'bg-success' : 'bg-secondary'"
                  >
                    {{ testimonial.is_active ? "Visible" : "Hidden" }}
                  </span>
                </td>

                <td class="text-end">
                  <NuxtLink
                    :to="`/testimonials/edit/${testimonial.id}`"
                    class="btn btn-sm btn-outline-dark me-2"
                  >
                    Edit
                  </NuxtLink>

                  <button
                    class="btn btn-sm btn-outline-danger"
                    @click="deleteTestimonial(testimonial)"
                  >
                    Delete
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <p v-if="testimonials.length === 0" class="text-muted mb-0">
          No testimonials found yet.
        </p>
      </div>
    </main>
  </div>
</template>