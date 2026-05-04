<script setup>
import { createClient } from "@supabase/supabase-js";
import RichTextEditor from "~/components/admin/RichTextEditor.vue";

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
  content: "",
  image_url: "",
  author: "GlobalBliss",
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
    errorMessage.value = error.message || "Unable to fetch blog post.";
    isLoading.value = false;
    return;
  }

  form.value = {
    title: data.title || "",
    slug: data.slug || "",
    category: data.category || "",
    excerpt: data.excerpt || "",
    content: data.content || "",
    image_url: data.image_url || "",
    author: data.author || "GlobalBliss",
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

  const safeSlug =
    form.value.slug ||
    form.value.title
      .toLowerCase()
      .trim()
      .replace(/[^a-z0-9]+/g, "-")
      .replace(/(^-|-$)+/g, "") ||
    "blog";

  const fileName = `${safeSlug}-${Date.now()}.${fileExt}`;
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

  try {
    if (!form.value.slug) {
      generateSlug();
    }

    const imageUrl = await uploadImage();

    const { error } = await supabase
      .from("blog_posts")
      .update({
        title: form.value.title,
        slug: form.value.slug,
        category: form.value.category,
        excerpt: form.value.excerpt,
        content: form.value.content,
        image_url: imageUrl,
        author: form.value.author,
        is_published: form.value.is_published,
        sort_order: Number(form.value.sort_order),
        updated_at: new Date().toISOString(),
      })
      .eq("id", postId);

    if (error) {
      throw error;
    }

    form.value.image_url = imageUrl;
    selectedFile.value = null;
    previewUrl.value = "";

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
          Update this blog post, rich content, SEO excerpt, image, and visibility.
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
                <label class="form-label">Post Title</label>
                <input
                  v-model="form.title"
                  type="text"
                  class="form-control"
                  placeholder="Blog post title"
                  required
                  @blur="generateSlug"
                />
              </div>

              <div class="col-md-4">
                <label class="form-label">Category</label>
                <input
                  v-model="form.category"
                  type="text"
                  class="form-control"
                  placeholder="Branding, Design, Web..."
                />
              </div>

              <div class="col-md-8">
                <label class="form-label">Slug</label>
                <input
                  v-model="form.slug"
                  type="text"
                  class="form-control"
                  placeholder="blog-post-slug"
                  required
                />
              </div>

              <div class="col-md-4">
                <label class="form-label">Author</label>
                <input
                  v-model="form.author"
                  type="text"
                  class="form-control"
                />
              </div>

              <div class="col-md-12">
                <label class="form-label">Excerpt / Meta Description</label>
                <textarea
                  v-model="form.excerpt"
                  class="form-control"
                  rows="3"
                  placeholder="Short summary of the post. This also helps your blog SEO."
                ></textarea>

                <small class="text-muted d-block mt-1">
                  Recommended: 140 to 160 characters.
                </small>
              </div>

              <div class="col-md-12">
                <label class="form-label">Blog Content</label>

                <RichTextEditor v-model="form.content" />

                <small class="text-muted d-block mt-2">
                  Use the editor tools to bold, italicize, underline, add headings,
                  create lists, and insert internal or external links.
                </small>
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

              <div class="col-md-9 d-flex align-items-end">
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
              <h6 class="fw-bold mb-3">Featured Image</h6>

              <div class="mb-3">
                <img
                  :src="previewUrl || getImageUrl(form.image_url)"
                  :alt="form.title || 'Blog image'"
                  style="width: 100%; height: 220px; object-fit: cover; border-radius: 18px;"
                />
              </div>

              <label class="form-label">Replace Blog Image</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                @change="handleFileSelect"
              />

              <small class="text-muted d-block mt-2">
                Recommended size: 1200 × 675px. Use WebP if possible.
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

              <p class="text-muted mb-0 mt-3">
                Published posts will appear on your portfolio blog page and can be indexed by Google.
              </p>
            </div>
          </div>
        </div>
      </form>
    </div>
  </div>
</template>