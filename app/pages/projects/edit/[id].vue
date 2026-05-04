<script setup>
import { createClient } from "@supabase/supabase-js";

definePageMeta({
  layout: "admin",
});

const route = useRoute();
const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const projectId = route.params.id;

const form = ref({
  title: "",
  category: "",
  description: "",
  image_url: "",
  project_url: "",
  client: "",
  year: "",
  service: "",
  is_featured: true,
  sort_order: 1,
});

const selectedFile = ref(null);
const previewUrl = ref("");

const galleryImages = ref([]);
const selectedGalleryFiles = ref([]);
const galleryPreviewUrls = ref([]);

const isLoading = ref(true);
const isSaving = ref(false);
const successMessage = ref("");
const errorMessage = ref("");

const fallbackImage = "/admin/images/globalbliss-logo.png";

const getImageUrl = (imagePath) => {
  if (!imagePath) return fallbackImage;

  if (imagePath.startsWith("http://") || imagePath.startsWith("https://")) {
    return imagePath;
  }

  if (imagePath.startsWith("/")) {
    return imagePath;
  }

  return `/${imagePath}`;
};

const fetchGalleryImages = async () => {
  const { data, error } = await supabase
    .from("project_gallery")
    .select("*")
    .eq("project_id", projectId)
    .order("sort_order", { ascending: true });

  if (error) {
    console.error(error.message);
    return;
  }

  galleryImages.value = data || [];
};

const fetchProject = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("projects")
    .select("*")
    .eq("id", projectId)
    .single();

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  form.value = {
    title: data.title || "",
    category: data.category || "",
    description: data.description || "",
    image_url: data.image_url || "",
    project_url: data.project_url || "",
    client: data.client || "",
    year: data.year || "",
    service: data.service || "",
    is_featured: data.is_featured ?? true,
    sort_order: data.sort_order || 1,
  };

  await fetchGalleryImages();

  isLoading.value = false;
};

const handleFileSelect = (event) => {
  const file = event.target.files?.[0];

  if (!file) return;

  selectedFile.value = file;
  previewUrl.value = URL.createObjectURL(file);
};

const handleGalleryFilesSelect = (event) => {
  const files = Array.from(event.target.files || []);

  if (!files.length) return;

  selectedGalleryFiles.value = files;
  galleryPreviewUrls.value = files.map((file) => URL.createObjectURL(file));
};

const uploadImage = async () => {
  if (!selectedFile.value) {
    return form.value.image_url;
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

const uploadGalleryImages = async () => {
  if (!selectedGalleryFiles.value.length) return;

  for (let index = 0; index < selectedGalleryFiles.value.length; index++) {
    const file = selectedGalleryFiles.value[index];
    const fileExt = file.name.split(".").pop();

    const safeTitle = form.value.title
      .toLowerCase()
      .trim()
      .replace(/[^a-z0-9]+/g, "-")
      .replace(/(^-|-$)+/g, "");

    const fileName = `${safeTitle || "project"}-gallery-${Date.now()}-${index}.${fileExt}`;
    const filePath = `project-gallery/${fileName}`;

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

    const nextOrder = galleryImages.value.length + index + 1;

    const { error: insertError } = await supabase
      .from("project_gallery")
      .insert({
        project_id: Number(projectId),
        image_url: data.publicUrl,
        sort_order: nextOrder,
      });

    if (insertError) {
      throw insertError;
    }
  }

  selectedGalleryFiles.value = [];
  galleryPreviewUrls.value = [];

  await fetchGalleryImages();
};

const deleteGalleryImage = async (image) => {
  const confirmed = confirm("Delete this gallery image?");

  if (!confirmed) return;

  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("project_gallery")
    .delete()
    .eq("id", image.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = "Gallery image deleted successfully.";
  await fetchGalleryImages();
};

const updateProject = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  try {
    const imageUrl = await uploadImage();

    const { error } = await supabase
      .from("projects")
      .update({
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
        updated_at: new Date().toISOString(),
      })
      .eq("id", projectId);

    if (error) {
      throw error;
    }

    await uploadGalleryImages();

    successMessage.value = "Project updated successfully.";

    setTimeout(() => {
      navigateTo("/projects");
    }, 900);
  } catch (error) {
    errorMessage.value = error.message || "Unable to update project.";
  } finally {
    isSaving.value = false;
  }
};

onMounted(() => {
  fetchProject();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Edit Project</h5>
        <p class="text-muted mb-0">
          Update project details, visibility, order, image, and gallery.
        </p>
      </div>

      <NuxtLink to="/projects" class="btn btn-outline-dark">
        <i class="bi bi-arrow-left me-2"></i>
        Back to Projects
      </NuxtLink>
    </div>

    <div v-if="isLoading" class="admin-card">
      <div class="d-flex align-items-center gap-3">
        <div class="spinner-border text-primary" role="status"></div>
        <p class="mb-0 text-muted">Loading project...</p>
      </div>
    </div>

    <div v-else-if="errorMessage && !form.title" class="admin-card">
      <div class="alert alert-danger mb-0">
        {{ errorMessage }}
      </div>
    </div>

    <div v-else class="admin-card">
      <form @submit.prevent="updateProject">
        <div class="row g-4">
          <div class="col-lg-8">
            <div class="row g-3">
              <div class="col-md-6">
                <label class="form-label">Project Title</label>
                <input
                  v-model="form.title"
                  type="text"
                  class="form-control"
                  placeholder="Project title"
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
                  placeholder="Client name"
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
                  placeholder="Website Design, Branding, Social Media Design"
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
                <label class="form-label">Description</label>
                <textarea
                  v-model="form.description"
                  class="form-control"
                  rows="5"
                  placeholder="Write a short description of the project"
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
                  <i class="bi bi-check-circle me-2"></i>
                  {{ isSaving ? "Saving..." : "Update Project" }}
                </button>
              </div>
            </div>
          </div>

          <div class="col-lg-4">
            <div class="p-3 rounded-4" style="background: rgba(235, 93, 58, 0.06);">
              <h6 class="fw-bold mb-3">Project Image</h6>

              <div class="mb-3">
                <img
                  :src="previewUrl || getImageUrl(form.image_url)"
                  :alt="form.title"
                  style="width: 100%; height: 230px; object-fit: cover; border-radius: 18px;"
                />
              </div>

              <label class="form-label">Replace Main Image</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                @change="handleFileSelect"
              />

              <small class="text-muted d-block mt-2">
                Leave empty if you want to keep the current main image.
              </small>
            </div>

            <div class="p-3 rounded-4 mt-3" style="background: rgba(17, 17, 17, 0.04);">
              <h6 class="fw-bold mb-2">Visibility</h6>

              <p class="text-muted mb-2">
                Current status:
              </p>

              <span
                class="badge"
                :class="form.is_featured ? 'bg-success' : 'bg-secondary'"
              >
                {{ form.is_featured ? "Visible on portfolio" : "Hidden from portfolio" }}
              </span>
            </div>

            <div class="p-3 rounded-4 mt-3" style="background: rgba(235, 93, 58, 0.06);">
              <h6 class="fw-bold mb-3">Project Gallery Images</h6>

              <label class="form-label">Upload Gallery Images</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                multiple
                @change="handleGalleryFilesSelect"
              />

              <small class="text-muted d-block mt-2">
                You can select multiple images. They will appear on the single project page.
              </small>
            </div>
          </div>

          <div v-if="galleryPreviewUrls.length" class="col-lg-12">
            <div class="p-3 rounded-4" style="background: rgba(17, 17, 17, 0.04);">
              <h6 class="fw-bold mb-3">New Gallery Preview</h6>

              <div class="row g-3">
                <div
                  v-for="(preview, index) in galleryPreviewUrls"
                  :key="index"
                  class="col-md-4"
                >
                  <img
                    :src="preview"
                    alt="Gallery preview"
                    style="width: 100%; height: 180px; object-fit: cover; border-radius: 14px;"
                  />
                </div>
              </div>
            </div>
          </div>

          <div v-if="galleryImages.length" class="col-lg-12">
            <div class="p-3 rounded-4" style="background: rgba(17, 17, 17, 0.04);">
              <h6 class="fw-bold mb-3">Existing Gallery Images</h6>

              <div class="row g-3">
                <div
                  v-for="image in galleryImages"
                  :key="image.id"
                  class="col-md-4"
                >
                  <div class="p-2 border rounded-4">
                    <img
                      :src="image.image_url"
                      alt="Project gallery"
                      style="width: 100%; height: 180px; object-fit: cover; border-radius: 14px;"
                    />

                    <button
                      type="button"
                      class="btn btn-sm btn-outline-danger mt-2 w-100"
                      @click="deleteGalleryImage(image)"
                    >
                      Delete Image
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>