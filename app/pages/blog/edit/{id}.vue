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

const postId = route.params.id;

const form = ref({
  title: "",
  slug: "",
  category: "",
  excerpt: "",
  description: "",
  content: "",
  body: "",
  image_url: "",
  is_published: true,
  sort_order: 1,
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

const generateSlug = () => {
  if (!form.value.title) return;

  form.value.slug = form.value.title
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)+/g, "");
};

const fetchPost = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("blog_posts")
    .select("*")
    .eq("id", postId)
    .single();

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  form.value = {
    title: data.title || "",
    slug: data.slug || "",
    category: data.category || "",
    excerpt: data.excerpt || "",
    description: data.description || "",
    content: data.content || data.body || "",
    body: data.body || data.content || "",
    image_url: data.image_url || "",
    is_published: data.is_published ?? true,
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

  const safeTitle =
    form.value.slug ||
    form.value.title
      .toLowerCase()
      .trim()
      .replace(/[^a-z0-9]+/g, "-")
      .replace(/(^-|-$)+/g, "");

  const fileName = `${safeTitle || "blog"}-${Date.now()}.${fileExt}`;
  const filePath = `blog/${fileName}`;

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

const updatePost = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  if (!form.value.slug) {
    generateSlug();
  }

  try {
    const imageUrl = await uploadImage();

    const { error } = await supabase
      .from("blog_posts")
      .update({
        title: form.value.title,
        slug: form.value.slug,
        category: form.value.category,
        excerpt: form.value.excerpt,
        description: form.value.description || form.value.excerpt,
        content: form.value.content,
        body: form.value.content,
        image_url: imageUrl,
        is_published: form.value.is_published,
        sort_order: Number(form.value.sort_order),
        updated_at: new Date().toISOString(),
      })
      .eq("id", postId);

    if (error) {
      throw error;
    }

    successMessage.value = "Blog post updated successfully.";

    setTimeout(() => {
      navigateTo("/blog");
    }, 900);
  } catch (error) {
    errorMessage.value = error.message || "Unable to update blog post.";
  } finally {
    isSaving.value = false;
  }
};

onMounted(() => {
  fetchPost();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Edit Blog Post</h5>
        <p class="text-muted mb-0">
          Update blog title, slug, image, content, and publishing status.
        </p>
      </div>

      <NuxtLink to="/blog" class="btn btn-outline-dark">
        <i class="bi bi-arrow-left me-2"></i>
        Back to Blog
      </NuxtLink>
    </div>

    <div v-if="isLoading" class="admin-card">
      <div class="d-flex align-items-center gap-3">
        <div class="spinner-border text-primary" role="status"></div>
        <p class="mb-0 text-muted">Loading blog post...</p>
      </div>
    </div>

    <div v-else-if="errorMessage && !form.title" class="admin-card">
      <div class="alert alert-danger mb-0">
        {{ errorMessage }}
      </div>
    </div>

    <div v-else class="admin-card">
      <form @submit.prevent="updatePost">
        <div class="row g-4">
          <div class="col-lg-8">
            <div class="row g-3">
              <div class="col-md-8">
                <label class="form-label">Blog Title</label>
                <input
                  v-model="form.title"
                  type="text"
                  class="form-control"
                  placeholder="Enter blog title"
                  required
                  @blur="generateSlug"
                />
              </div>

              <div class="col-md-4">
                <label class="form-label">Sort Order</label>
                <input
                  v-model="form.sort_order"
                  type="number"
                  class="form-control"
                  min="1"
                />
              </div>

              <div class="col-md-6">
                <label class="form-label">Slug</label>
                <input
                  v-model="form.slug"
                  type="text"
                  class="form-control"
                  placeholder="blog-post-slug"
                  required
                />
              </div>

              <div class="col-md-6">
                <label class="form-label">Category</label>
                <input
                  v-model="form.category"
                  type="text"
                  class="form-control"
                  placeholder="Brand Story"
                />
              </div>

              <div class="col-md-12">
                <label class="form-label">Excerpt</label>
                <textarea
                  v-model="form.excerpt"
                  class="form-control"
                  rows="3"
                  placeholder="Short summary for the blog card"
                ></textarea>
              </div>

              <div class="col-md-12">
                <label class="form-label">Blog Content</label>
                <textarea
                  v-model="form.content"
                  class="form-control"
                  rows="10"
                  placeholder="Write the full blog post here"
                  required
                ></textarea>
              </div>

              <div class="col-md-12">
                <div class="form-check">
                  <input
                    v-model="form.is_published"
                    class="form-check-input"
                    type="checkbox"
                    id="isPublished"
                  />

                  <label class="form-check-label" for="isPublished">
                    Publish this blog post
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
                  {{ isSaving ? "Saving..." : "Update Blog Post" }}
                </button>
              </div>
            </div>
          </div>

          <div class="col-lg-4">
            <div class="p-3 rounded-4" style="background: rgba(235, 93, 58, 0.06);">
              <h6 class="fw-bold mb-3">Blog Image</h6>

              <div class="mb-3">
                <img
                  :src="previewUrl || getImageUrl(form.image_url)"
                  :alt="form.title"
                  style="width: 100%; height: 230px; object-fit: cover; border-radius: 18px;"
                />
              </div>

              <label class="form-label">Replace Image</label>
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

            <div class="p-3 rounded-4 mt-3" style="background: rgba(17, 17, 17, 0.04);">
              <h6 class="fw-bold mb-2">Publishing Status</h6>

              <span
                class="badge"
                :class="form.is_published ? 'bg-success' : 'bg-secondary'"
              >
                {{ form.is_published ? "Published" : "Draft" }}
              </span>

              <p class="text-muted mt-3 mb-0">
                Published posts will appear on your public portfolio blog page.
              </p>
            </div>

            <div class="p-3 rounded-4 mt-3" style="background: rgba(17, 17, 17, 0.04);">
              <h6 class="fw-bold mb-2">Public URL</h6>

              <p class="text-muted mb-2">
                This post will open at:
              </p>

              <code style="font-size: 13px;">
                /blog/{{ form.slug }}
              </code>
            </div>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>