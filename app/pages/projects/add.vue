<script setup>
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const form = ref({
  title: "",
  category: "",
  description: "",
  image_url: "",
  project_url: "",
  client: "",
  year: "2026",
  service: "",
  is_featured: true,
  sort_order: 9,
});

const selectedFile = ref(null);
const previewUrl = ref("");
const isSaving = ref(false);
const successMessage = ref("");
const errorMessage = ref("");

const logout = () => {
  localStorage.removeItem("globalbliss_admin_logged_in");
  navigateTo("/login");
};

const handleFileSelect = (event) => {
  const file = event.target.files?.[0];

  if (!file) return;

  selectedFile.value = file;
  previewUrl.value = URL.createObjectURL(file);
};

const uploadImage = async () => {
  if (!selectedFile.value) {
    throw new Error("Please select a project image.");
  }

  const file = selectedFile.value;
  const fileExt = file.name.split(".").pop();

  const safeTitle = form.value.title
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)+/g, "");

  const fileName = `${safeTitle || "project"}-${Date.now()}.${fileExt}`;
  const filePath = `projects/${fileName}`;

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

const addProject = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  try {
    const imageUrl = await uploadImage();

    const { error } = await supabase.from("projects").insert([
      {
        title: form.value.title,
        category: form.value.category,
        description: form.value.description,
        image_url: imageUrl,
        project_url: form.value.project_url,
        client: form.value.client,
        year: form.value.year,
        service: form.value.service,
        is_featured: form.value.is_featured,
        sort_order: Number(form.value.sort_order),
      },
    ]);

    if (error) {
      throw error;
    }

    successMessage.value = "Project added successfully.";

    setTimeout(() => {
      navigateTo("/projects");
    }, 900);
  } catch (error) {
    errorMessage.value = error.message || "Unable to add project.";
  } finally {
    isSaving.value = false;
  }
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
      </div>

      <NuxtLink to="/dashboard">
        <i class="bi bi-grid me-2"></i>
        Dashboard
      </NuxtLink>

      <NuxtLink to="/projects">
        <i class="bi bi-folder2-open me-2"></i>
        Projects
      </NuxtLink>

      <NuxtLink to="/projects/add">
        <i class="bi bi-plus-circle me-2"></i>
        Add Project
      </NuxtLink>

      <a href="#" @click.prevent="logout">
        <i class="bi bi-box-arrow-left me-2"></i>
        Logout
      </a>
    </aside>

    <main class="admin-main">
      <div class="d-flex justify-content-between align-items-center mb-4">
        <div>
          <h1 class="fw-bold mb-1">Add New Project</h1>
          <p class="text-muted mb-0">
            Upload an image and add a new project to The GlobalBliss portfolio.
          </p>
        </div>

        <NuxtLink to="/projects" class="btn btn-outline-dark">
          Back to Projects
        </NuxtLink>
      </div>

      <div class="admin-card">
        <form @submit.prevent="addProject">
          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">Project Title</label>
              <input
                v-model="form.title"
                type="text"
                class="form-control"
                placeholder="e.g. Media Force Creative Hub"
                required
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Category</label>
              <select v-model="form.category" class="form-select" required>
                <option value="">Select category</option>
                <option value="Social Media Design">Social Media Design</option>
                <option value="Website Development">Website Development</option>
                <option value="Brand Identity">Brand Identity</option>
                <option value="UI/UX Design">UI/UX Design</option>
                <option value="Creative Design">Creative Design</option>
              </select>
            </div>

            <div class="col-md-6">
              <label class="form-label">Client</label>
              <input
                v-model="form.client"
                type="text"
                class="form-control"
                placeholder="Client or brand name"
              />
            </div>

            <div class="col-md-3">
              <label class="form-label">Year</label>
              <input
                v-model="form.year"
                type="text"
                class="form-control"
                placeholder="2026"
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

            <div class="col-md-6">
              <label class="form-label">Service</label>
              <input
                v-model="form.service"
                type="text"
                class="form-control"
                placeholder="e.g. Website Development"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Project URL</label>
              <input
                v-model="form.project_url"
                type="text"
                class="form-control"
                placeholder="https://example.com"
              />
            </div>

            <div class="col-md-12">
              <label class="form-label">Project Image</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                @change="handleFileSelect"
                required
              />
              <small class="text-muted">
                Select from your device. The image will upload to Supabase Storage automatically.
              </small>
            </div>

            <div v-if="previewUrl" class="col-md-12">
              <label class="form-label">Image Preview</label>
              <div>
                <img
                  :src="previewUrl"
                  alt="Preview"
                  style="max-width: 280px; height: 170px; object-fit: cover; border-radius: 14px;"
                />
              </div>
            </div>

            <div class="col-md-12">
              <label class="form-label">Description</label>
              <textarea
                v-model="form.description"
                class="form-control"
                rows="4"
                placeholder="Write a short project description"
              ></textarea>
            </div>

            <div class="col-md-12">
              <div class="form-check">
                <input
                  v-model="form.is_featured"
                  class="form-check-input"
                  type="checkbox"
                  id="isFeatured"
                />
                <label class="form-check-label" for="isFeatured">
                  Show this project on portfolio website
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
                {{ isSaving ? "Saving..." : "Add Project" }}
              </button>
            </div>
          </div>
        </form>
      </div>
    </main>
  </div>
</template>