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

const projects = ref([]);
const isLoading = ref(true);
const successMessage = ref("");
const errorMessage = ref("");

const fallbackImage = "/admin/images/globalbliss-logo.png";

const getImageUrl = (url) => {
  if (!url) return fallbackImage;

  if (url.startsWith("http://") || url.startsWith("https://")) {
    return url;
  }

  if (url.startsWith("/")) {
    return url;
  }

  return `/${url}`;
};

const fetchProjects = async () => {
  isLoading.value = true;
  errorMessage.value = "";
  successMessage.value = "";

  const { data, error } = await supabase
    .from("projects")
    .select("*")
    .order("sort_order", { ascending: true });

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  projects.value = data || [];
  isLoading.value = false;
};

const toggleVisibility = async (project) => {
  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("projects")
    .update({
      is_featured: !project.is_featured,
      updated_at: new Date().toISOString(),
    })
    .eq("id", project.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = project.is_featured
    ? "Project hidden from portfolio."
    : "Project is now visible on portfolio.";

  await fetchProjects();
};

const deleteProject = async (project) => {
  const confirmed = confirm(`Delete "${project.title}" permanently?`);

  if (!confirmed) return;

  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("projects")
    .delete()
    .eq("id", project.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = "Project deleted successfully.";
  await fetchProjects();
};

const formatDate = (date) => {
  if (!date) return "Not available";

  return new Date(date).toLocaleDateString("en-NG", {
    year: "numeric",
    month: "short",
    day: "numeric",
  });
};

onMounted(() => {
  fetchProjects();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Portfolio Projects</h5>
        <p class="text-muted mb-0">
          Manage all projects showing on The GlobalBliss portfolio website.
        </p>
      </div>

      <div class="d-flex flex-wrap gap-2">
        <button class="btn btn-outline-dark" @click="fetchProjects">
          <i class="bi bi-arrow-clockwise me-2"></i>
          Refresh
        </button>

        <NuxtLink to="/projects/add" class="btn btn-primary">
          <i class="bi bi-plus-circle me-2"></i>
          Add New Project
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
        <p class="mb-0 text-muted">Loading projects...</p>
      </div>
    </div>

    <div v-else class="admin-card">
      <div v-if="projects.length" class="table-responsive">
        <table class="table table-hover align-middle">
          <thead>
            <tr>
              <th>Image</th>
              <th>Project</th>
              <th>Category</th>
              <th>Client</th>
              <th>Year</th>
              <th>Order</th>
              <th>Status</th>
              <th>Date Added</th>
              <th class="text-end">Actions</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="project in projects" :key="project.id">
              <td>
                <img
                  :src="getImageUrl(project.image_url)"
                  :alt="project.title"
                  style="width: 70px; height: 54px; object-fit: cover; border-radius: 14px;"
                />
              </td>

              <td>
                <div class="fw-bold">
                  {{ project.title }}
                </div>

                <div v-if="project.description" class="small text-muted project-desc">
                  {{ project.description }}
                </div>
              </td>

              <td>
                {{ project.category || "Not set" }}
              </td>

              <td>
                {{ project.client || "Not set" }}
              </td>

              <td>
                {{ project.year || "Not set" }}
              </td>

              <td>
                {{ project.sort_order || 0 }}
              </td>

              <td>
                <span
                  class="badge"
                  :class="project.is_featured ? 'bg-success' : 'bg-secondary'"
                >
                  {{ project.is_featured ? "Visible" : "Hidden" }}
                </span>
              </td>

              <td>
                {{ formatDate(project.created_at) }}
              </td>

              <td class="text-end">
                <div class="d-flex justify-content-end gap-2">
                  <button
                    class="btn btn-sm btn-outline-dark"
                    @click="toggleVisibility(project)"
                  >
                    {{ project.is_featured ? "Hide" : "Show" }}
                  </button>

                  <NuxtLink
                    :to="`/projects/edit/${project.id}`"
                    class="btn btn-sm btn-outline-dark"
                  >
                    Edit
                  </NuxtLink>

                  <button
                    class="btn btn-sm btn-outline-danger"
                    @click="deleteProject(project)"
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
          <i class="bi bi-folder2-open"></i>
        </div>

        <h5 class="fw-bold mt-3">No projects found yet.</h5>

        <p class="text-muted">
          Add your first portfolio project and it will appear here.
        </p>

        <NuxtLink to="/projects/add" class="btn btn-primary">
          <i class="bi bi-plus-circle me-2"></i>
          Add New Project
        </NuxtLink>
      </div>
    </div>
  </div>
</template>

<style scoped>
.project-desc {
  max-width: 270px;
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