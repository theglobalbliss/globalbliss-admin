<script setup>
import { createClient } from "@supabase/supabase-js";

const route = useRoute();
const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const testimonialId = route.params.id;

const form = ref({
  name: "",
  role: "",
  company: "",
  message: "",
  image_url: "",
  rating: 5,
  is_active: true,
  sort_order: 1,
});

const selectedFile = ref(null);
const previewUrl = ref("");
const isLoading = ref(true);
const isSaving = ref(false);
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

const fetchTestimonial = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("testimonials")
    .select("*")
    .eq("id", testimonialId)
    .single();

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  form.value = {
    name: data.name || "",
    role: data.role || "",
    company: data.company || "",
    message: data.message || "",
    image_url: data.image_url || "",
    rating: data.rating || 5,
    is_active: data.is_active ?? true,
    sort_order: data.sort_order || 1,
  };

  isLoading.value = false;
};

const handleFileSelect = (event) => {
  const file = event.target.files?.[0];

  if (!file) return;

  selectedFile.value = file;
  previewUrl.value = URL.createObjectURL(file);
};

const uploadImage = async () => {
  if (!selectedFile.value) {
    return form.value.image_url;
  }

  const file = selectedFile.value;
  const fileExt = file.name.split(".").pop();

  const safeName = form.value.name
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)+/g, "");

  const fileName = `${safeName || "testimonial"}-${Date.now()}.${fileExt}`;
  const filePath = `testimonials/${fileName}`;

  const { error: uploadError } = await supabase.storage
    .from("project-images")
    .upload(filePath, file, {
      cacheControl: "3600",
      upsert: false,
    });

  if (uploadError) {
    throw uploadError;
  }

  const { data } = supabase.storage
    .from("project-images")
    .getPublicUrl(filePath);

  return data.publicUrl;
};

const updateTestimonial = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  try {
    const imageUrl = await uploadImage();

    const { error } = await supabase
      .from("testimonials")
      .update({
        name: form.value.name,
        role: form.value.role,
        company: form.value.company,
        message: form.value.message,
        image_url: imageUrl,
        rating: Number(form.value.rating),
        is_active: form.value.is_active,
        sort_order: Number(form.value.sort_order),
        updated_at: new Date().toISOString(),
      })
      .eq("id", testimonialId);

    if (error) {
      throw error;
    }

    successMessage.value = "Testimonial updated successfully.";

    setTimeout(() => {
      navigateTo("/testimonials");
    }, 900);
  } catch (error) {
    errorMessage.value = error.message || "Unable to update testimonial.";
  } finally {
    isSaving.value = false;
  }
};

onMounted(() => {
  const isLoggedIn = localStorage.getItem("globalbliss_admin_logged_in");

  if (!isLoggedIn) {
    navigateTo("/login");
    return;
  }

  fetchTestimonial();
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
          <h5 class="fw-bold mb-1">Edit Testimonial</h5>
          <p class="text-muted mb-0">
            Update client review, rating, visibility, and photo.
          </p>
        </div>

        <NuxtLink to="/testimonials" class="btn btn-outline-dark">
          Back to Testimonials
        </NuxtLink>
      </div>

      <div v-if="isLoading" class="admin-card">
        Loading testimonial...
      </div>

      <div v-else-if="errorMessage && !form.name" class="admin-card text-danger">
        {{ errorMessage }}
      </div>

      <div v-else class="admin-card">
        <form @submit.prevent="updateTestimonial">
          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">Client Name</label>
              <input
                v-model="form.name"
                type="text"
                class="form-control"
                required
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Role</label>
              <input
                v-model="form.role"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Company</label>
              <input
                v-model="form.company"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-3">
              <label class="form-label">Rating</label>
              <input
                v-model="form.rating"
                type="number"
                class="form-control"
                min="1"
                max="5"
              />
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
              <label class="form-label">Current Photo</label>
              <div v-if="form.image_url">
                <img
                  :src="getImageUrl(form.image_url)"
                  :alt="form.name"
                  style="width: 120px; height: 120px; object-fit: cover; border-radius: 50%;"
                />
              </div>
              <p v-else class="text-muted mb-0">No photo uploaded yet.</p>
            </div>

            <div class="col-md-12">
              <label class="form-label">Replace Photo</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                @change="handleFileSelect"
              />
            </div>

            <div v-if="previewUrl" class="col-md-12">
              <label class="form-label">New Photo Preview</label>
              <div>
                <img
                  :src="previewUrl"
                  alt="Preview"
                  style="width: 120px; height: 120px; object-fit: cover; border-radius: 50%;"
                />
              </div>
            </div>

            <div class="col-md-12">
              <label class="form-label">Message</label>
              <textarea
                v-model="form.message"
                class="form-control"
                rows="5"
                required
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
                  Show this testimonial on portfolio website
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
                {{ isSaving ? "Saving..." : "Update Testimonial" }}
              </button>
            </div>
          </div>
        </form>
      </div>
    </main>
  </div>
</template>