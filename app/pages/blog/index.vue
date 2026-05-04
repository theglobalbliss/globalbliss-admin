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

const posts = ref([]);
const isLoading = ref(true);
const successMessage = ref("");
const errorMessage = ref("");

const fallbackImage = "/admin/images/globalbliss-logo.png";

const getImageUrl = (imageUrl) => {
  if (!imageUrl) return fallbackImage;

  if (imageUrl.startsWith("http://") || imageUrl.startsWith("https://")) {
    return imageUrl;
  }

  if (imageUrl.startsWith("/")) {
    return imageUrl;
  }

  return `/${imageUrl}`;
};

const formatDate = (date) => {
  if (!date) return "Not available";

  return new Date(date).toLocaleDateString("en-NG", {
    year: "numeric",
    month: "short",
    day: "numeric",
  });
};

const fetchPosts = async () => {
  isLoading.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("blog_posts")
    .select("*")
    .order("sort_order", { ascending: true });

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  posts.value = data || [];
  isLoading.value = false;
};

const togglePublishStatus = async (post) => {
  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("blog_posts")
    .update({
      is_published: !post.is_published,
      updated_at: new Date().toISOString(),
    })
    .eq("id", post.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = post.is_published
    ? "Blog post moved to draft."
    : "Blog post published successfully.";

  await fetchPosts();
};

const deletePost = async (post) => {
  const confirmed = confirm(`Delete "${post.title}" permanently?`);

  if (!confirmed) return;

  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("blog_posts")
    .delete()
    .eq("id", post.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = "Blog post deleted successfully.";
  await fetchPosts();
};

onMounted(() => {
  fetchPosts();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Blog Posts</h5>
        <p class="text-muted mb-0">
          Create, edit, publish, draft, and manage blog posts for your portfolio website.
        </p>
      </div>

      <div class="d-flex flex-wrap gap-2">
        <button class="btn btn-outline-dark" @click="fetchPosts">
          <i class="bi bi-arrow-clockwise me-2"></i>
          Refresh
        </button>

        <NuxtLink to="/blog/add" class="btn btn-primary">
          <i class="bi bi-plus-circle me-2"></i>
          Add Blog Post
        </NuxtLink>
      </div>
    </div>

    <div v-if="successMessage" class="alert alert-success">
      {{ successMessage }}
    </div>

    <div v-if="errorMessage" class="alert alert-danger">
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="admin-card">
      <div class="d-flex align-items-center gap-3">
        <div class="spinner-border text-primary" role="status"></div>
        <p class="mb-0 text-muted">Loading blog posts...</p>
      </div>
    </div>

    <div v-else class="admin-card">
      <div v-if="posts.length" class="table-responsive">
        <table class="table table-hover align-middle">
          <thead>
            <tr>
              <th>Image</th>
              <th>Post</th>
              <th>Category</th>
              <th>Slug</th>
              <th>Order</th>
              <th>Status</th>
              <th>Date Added</th>
              <th class="text-end">Actions</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="post in posts" :key="post.id">
              <td>
                <img
                  :src="getImageUrl(post.image_url)"
                  :alt="post.title"
                  style="width: 75px; height: 55px; object-fit: cover; border-radius: 14px;"
                />
              </td>

              <td>
                <div class="fw-bold">
                  {{ post.title }}
                </div>

                <div v-if="post.excerpt || post.description" class="small text-muted blog-desc">
                  {{ post.excerpt || post.description }}
                </div>
              </td>

              <td>
                {{ post.category || "Not set" }}
              </td>

              <td>
                <code style="font-size: 12px;">
                  {{ post.slug }}
                </code>
              </td>

              <td>
                {{ post.sort_order || 0 }}
              </td>

              <td>
                <span
                  class="badge"
                  :class="post.is_published ? 'bg-success' : 'bg-secondary'"
                >
                  {{ post.is_published ? "Published" : "Draft" }}
                </span>
              </td>

              <td>
                {{ formatDate(post.created_at) }}
              </td>

              <td class="text-end">
                <div class="d-flex justify-content-end gap-2">
                  <button
                    class="btn btn-sm btn-outline-dark"
                    @click="togglePublishStatus(post)"
                  >
                    {{ post.is_published ? "Draft" : "Publish" }}
                  </button>

                  <NuxtLink
                    :to="`/blog-edit/${post.id}`"
                    class="btn btn-sm btn-outline-dark"
                  >
                    Edit
                  </NuxtLink>

                  <a
                    v-if="post.slug"
                    :href="`https://theglobalbliss.online/blog/${post.slug}`"
                    target="_blank"
                    rel="noopener noreferrer"
                    class="btn btn-sm btn-outline-dark"
                  >
                    View
                  </a>

                  <button
                    class="btn btn-sm btn-outline-danger"
                    @click="deletePost(post)"
                  >
                    Delete
                  </button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-else class="text-center py-5">
        <div class="stat-icon mx-auto">
          <i class="bi bi-journal-text"></i>
        </div>

        <h5 class="fw-bold mt-3">No blog posts found yet.</h5>

        <p class="text-muted">
          Add your first blog post and it will appear here.
        </p>

        <NuxtLink to="/blog/add" class="btn btn-primary">
          <i class="bi bi-plus-circle me-2"></i>
          Add Blog Post
        </NuxtLink>
      </div>
    </div>
  </div>
</template>

<style scoped>
.blog-desc {
  max-width: 280px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.btn-outline-danger {
  border-color: rgba(220, 53, 69, 0.3);
  color: #dc3545;
}

.btn-outline-danger:hover {
  background: #dc3545;
  color: #fff;
}
</style>