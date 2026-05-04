<script setup>
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const form = ref({
  title: "",
  slug: "",
  category: "",
  excerpt: "",
  content: "",
  image_url: "",
  author: "GlobalBliss",
  is_published: true,
  sort_order: 3,
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

const generateSlug = () => {
  form.value.slug = form.value.title
    .toLowerCase()
    .trim()
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)+/g, "");
};

const handleFileSelect = (event) => {
  const file = event.target.files?.[0];

  if (!file) return;

  selectedFile.value = file;
  previewUrl.value = URL.createObjectURL(file);
};

const uploadImage = async () => {
  if (!selectedFile.value) {
    return "";
  }

  const file = selectedFile.value;
  const fileExt = file.name.split(".").pop();
  const fileName = `${form.value.slug || "blog"}-${Date.now()}.${fileExt}`;
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

const addPost = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  try {
    if (!form.value.slug) {
      generateSlug();
    }

    const imageUrl = await uploadImage();

    const { error } = await supabase.from("blog_posts").insert([
      {
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
      },
    ]);

    if (error) {
      throw error;
    }

    successMessage.value = "Blog post added successfully.";

    setTimeout(() => {
      navigateTo("/blog");
    }, 900);
  } catch (error) {
    errorMessage.value = error.message || "Unable to add blog post.";
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

      <NuxtLink to="/blog">
        <i class="bi bi-journal-text"></i>
        Blog
      </NuxtLink>

      <NuxtLink to="/blog/add">
        <i class="bi bi-pencil"></i>
        Add Blog Post
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
          <h5 class="fw-bold mb-1">Add Blog Post</h5>
          <p class="text-muted mb-0">
            Create a new article, update, or brand story for your portfolio website.
          </p>
        </div>

        <NuxtLink to="/blog" class="btn btn-outline-dark">
          Back to Blog
        </NuxtLink>
      </div>

      <div class="admin-card">
        <form @submit.prevent="addPost">
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
              <label class="form-label">Featured Image</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                @change="handleFileSelect"
              />
            </div>

            <div v-if="previewUrl" class="col-md-12">
              <label class="form-label">Image Preview</label>
              <div>
                <img
                  :src="previewUrl"
                  alt="Preview"
                  style="max-width: 320px; height: 180px; object-fit: cover; border-radius: 18px;"
                />
              </div>
            </div>

            <div class="col-md-12">
              <label class="form-label">Excerpt</label>
              <textarea
                v-model="form.excerpt"
                class="form-control"
                rows="3"
                placeholder="Short summary of the post"
              ></textarea>
            </div>

            <div class="col-md-12">
              <label class="form-label">Content</label>
              <textarea
                v-model="form.content"
                class="form-control"
                rows="10"
                placeholder="Write the full blog post here"
              ></textarea>
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
                {{ isSaving ? "Saving..." : "Add Blog Post" }}
              </button>
            </div>
          </div>
        </form>
      </div>
    </main>
  </div>
</template>