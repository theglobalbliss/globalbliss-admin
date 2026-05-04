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

const form = ref({
  section_key: "main",
  eyebrow: "",
  name: "",
  role: "",
  title: "",
  description: "",
  description_two: "",
  description_three: "",
  profile_image_url: "",
  availability_text: "",
});

const selectedFile = ref(null);
const previewUrl = ref("");
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

const fetchAboutContent = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("about_content")
    .select("*")
    .eq("section_key", "main")
    .single();

  if (error && error.code !== "PGRST116") {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  if (data) {
    form.value = {
      section_key: data.section_key || "main",
      eyebrow: data.eyebrow || "",
      name: data.name || "",
      role: data.role || "",
      title: data.title || "",
      description: data.description || "",
      description_two: data.description_two || "",
      description_three: data.description_three || "",
      profile_image_url: data.profile_image_url || "",
      availability_text: data.availability_text || "",
    };
  }

  isLoading.value = false;
};

const handleFileSelect = (event) => {
  const file = event.target.files?.[0];

  if (!file) return;

  selectedFile.value = file;
  previewUrl.value = URL.createObjectURL(file);
};

const uploadProfileImage = async () => {
  if (!selectedFile.value) {
    return form.value.profile_image_url;
  }

  const file = selectedFile.value;
  const fileExt = file.name.split(".").pop();

  const fileName = `about-profile-${Date.now()}.${fileExt}`;
  const filePath = `about/${fileName}`;

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

const saveAboutContent = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  try {
    const profileImageUrl = await uploadProfileImage();

    const payload = {
      section_key: "main",
      eyebrow: form.value.eyebrow,
      name: form.value.name,
      role: form.value.role,
      title: form.value.title,
      description: form.value.description,
      description_two: form.value.description_two,
      description_three: form.value.description_three,
      profile_image_url: profileImageUrl,
      availability_text: form.value.availability_text,
      updated_at: new Date().toISOString(),
    };

    const { error } = await supabase
      .from("about_content")
      .upsert(payload, {
        onConflict: "section_key",
      });

    if (error) {
      throw error;
    }

    form.value.profile_image_url = profileImageUrl;
    selectedFile.value = null;
    previewUrl.value = "";

    successMessage.value = "About content updated successfully.";
  } catch (error) {
    errorMessage.value = error.message || "Unable to update about content.";
  } finally {
    isSaving.value = false;
  }
};

onMounted(() => {
  fetchAboutContent();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">About Content</h5>
        <p class="text-muted mb-0">
          Update the About page content shown on your portfolio website.
        </p>
      </div>

      <NuxtLink to="/dashboard" class="btn btn-outline-dark">
        <i class="bi bi-arrow-left me-2"></i>
        Back to Dashboard
      </NuxtLink>
    </div>

    <div v-if="isLoading" class="admin-card">
      <div class="d-flex align-items-center gap-3">
        <div class="spinner-border text-primary" role="status"></div>
        <p class="mb-0 text-muted">Loading about content...</p>
      </div>
    </div>

    <div v-else class="admin-card">
      <form @submit.prevent="saveAboutContent">
        <div class="row g-4">
          <div class="col-lg-8">
            <div class="row g-3">
              <div class="col-md-6">
                <label class="form-label">Eyebrow Text</label>
                <input
                  v-model="form.eyebrow"
                  type="text"
                  class="form-control"
                  placeholder="Hello There!"
                />
              </div>

              <div class="col-md-6">
                <label class="form-label">Name</label>
                <input
                  v-model="form.name"
                  type="text"
                  class="form-control"
                  placeholder="Anuoluwapo Bliss"
                />
              </div>

              <div class="col-md-12">
                <label class="form-label">Role</label>
                <input
                  v-model="form.role"
                  type="text"
                  class="form-control"
                  placeholder="Creative Designer, Website Developer & Brand Strategist"
                />
              </div>

              <div class="col-md-12">
                <label class="form-label">Main Title</label>
                <textarea
                  v-model="form.title"
                  class="form-control"
                  rows="3"
                  placeholder="Main about headline"
                ></textarea>
              </div>

              <div class="col-md-12">
                <label class="form-label">Description One</label>
                <textarea
                  v-model="form.description"
                  class="form-control"
                  rows="4"
                ></textarea>
              </div>

              <div class="col-md-12">
                <label class="form-label">Description Two</label>
                <textarea
                  v-model="form.description_two"
                  class="form-control"
                  rows="4"
                ></textarea>
              </div>

              <div class="col-md-12">
                <label class="form-label">Description Three</label>
                <textarea
                  v-model="form.description_three"
                  class="form-control"
                  rows="4"
                ></textarea>
              </div>

              <div class="col-md-12">
                <label class="form-label">Availability Text</label>
                <input
                  v-model="form.availability_text"
                  type="text"
                  class="form-control"
                  placeholder="Available for selected creative projects..."
                />
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
                  {{ isSaving ? "Saving..." : "Save About Content" }}
                </button>
              </div>
            </div>
          </div>

          <div class="col-lg-4">
            <div class="p-3 rounded-4" style="background: rgba(235, 93, 58, 0.06);">
              <h6 class="fw-bold mb-3">Profile Image</h6>

              <div class="mb-3">
                <img
                  :src="previewUrl || getImageUrl(form.profile_image_url)"
                  :alt="form.name"
                  style="width: 100%; height: 260px; object-fit: cover; border-radius: 18px;"
                />
              </div>

              <label class="form-label">Replace Profile Image</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                @change="handleFileSelect"
              />

              <small class="text-muted d-block mt-2">
                Leave empty if you want to keep the current image.
              </small>
            </div>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>