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

const resumeItems = ref([]);
const isLoading = ref(true);
const isSaving = ref(false);
const successMessage = ref("");
const errorMessage = ref("");

const form = ref({
  item_type: "experience",
  years: "",
  title: "",
  institution: "",
  description: "",
  icon: "ri-briefcase-line",
  is_active: true,
  sort_order: 1,
});

const editingId = ref(null);

const iconOptions = [
  "ri-briefcase-line",
  "ri-palette-line",
  "ri-layout-4-line",
  "ri-graduation-cap-line",
  "ri-bank-line",
  "ri-computer-line",
  "ri-book-line",
  "ri-code-s-slash-line",
  "ri-lightbulb-line",
  "ri-global-line",
];

const getAdminIcon = (icon) => {
  const iconMap = {
    "ri-briefcase-line": "bi bi-briefcase",
    "ri-palette-line": "bi bi-palette",
    "ri-layout-4-line": "bi bi-window-sidebar",
    "ri-graduation-cap-line": "bi bi-mortarboard",
    "ri-bank-line": "bi bi-bank",
    "ri-computer-line": "bi bi-laptop",
    "ri-book-line": "bi bi-book",
    "ri-code-s-slash-line": "bi bi-code-slash",
    "ri-lightbulb-line": "bi bi-lightbulb",
    "ri-global-line": "bi bi-globe2",
  };

  return iconMap[icon] || "bi bi-briefcase";
};

const resetForm = () => {
  editingId.value = null;

  form.value = {
    item_type: "experience",
    years: "",
    title: "",
    institution: "",
    description: "",
    icon: "ri-briefcase-line",
    is_active: true,
    sort_order: 1,
  };
};

const fetchResumeItems = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("resume_items")
    .select("*")
    .order("item_type", { ascending: true })
    .order("sort_order", { ascending: true });

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  resumeItems.value = data || [];
  isLoading.value = false;
};

const saveResumeItem = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  const payload = {
    item_type: form.value.item_type,
    years: form.value.years,
    title: form.value.title,
    institution: form.value.institution,
    description: form.value.description,
    icon: form.value.icon,
    is_active: form.value.is_active,
    sort_order: Number(form.value.sort_order),
    updated_at: new Date().toISOString(),
  };

  let error;

  if (editingId.value) {
    const response = await supabase
      .from("resume_items")
      .update(payload)
      .eq("id", editingId.value);

    error = response.error;
  } else {
    const response = await supabase
      .from("resume_items")
      .insert(payload);

    error = response.error;
  }

  if (error) {
    errorMessage.value = error.message;
    isSaving.value = false;
    return;
  }

  successMessage.value = editingId.value
    ? "Resume item updated successfully."
    : "Resume item added successfully.";

  resetForm();
  await fetchResumeItems();

  isSaving.value = false;
};

const editResumeItem = (item) => {
  editingId.value = item.id;

  form.value = {
    item_type: item.item_type || "experience",
    years: item.years || "",
    title: item.title || "",
    institution: item.institution || "",
    description: item.description || "",
    icon: item.icon || "ri-briefcase-line",
    is_active: item.is_active ?? true,
    sort_order: item.sort_order || 1,
  };

  window.scrollTo({
    top: 0,
    behavior: "smooth",
  });
};

const deleteResumeItem = async (item) => {
  const confirmed = confirm(`Delete "${item.title}"?`);

  if (!confirmed) return;

  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("resume_items")
    .delete()
    .eq("id", item.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = "Resume item deleted successfully.";
  await fetchResumeItems();
};

onMounted(() => {
  fetchResumeItems();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Resume Manager</h5>
        <p class="text-muted mb-0">
          Manage your Experience and Education section on the About page.
        </p>
      </div>

      <NuxtLink to="/dashboard" class="btn btn-outline-dark">
        <i class="bi bi-arrow-left me-2"></i>
        Back to Dashboard
      </NuxtLink>
    </div>

    <div v-if="successMessage" class="alert alert-success">
      {{ successMessage }}
    </div>

    <div v-if="errorMessage" class="alert alert-danger">
      {{ errorMessage }}
    </div>

    <div class="admin-card mb-4">
      <h5 class="fw-bold mb-3">
        {{ editingId ? "Edit Resume Item" : "Add Resume Item" }}
      </h5>

      <form @submit.prevent="saveResumeItem">
        <div class="row g-3">
          <div class="col-md-4">
            <label class="form-label">Type</label>
            <select v-model="form.item_type" class="form-select" required>
              <option value="experience">Experience</option>
              <option value="education">Education</option>
            </select>
          </div>

          <div class="col-md-4">
            <label class="form-label">Years</label>
            <input
              v-model="form.years"
              type="text"
              class="form-control"
              placeholder="2024 - Present"
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
            <label class="form-label">Title</label>
            <input
              v-model="form.title"
              type="text"
              class="form-control"
              placeholder="Brand Designer"
              required
            />
          </div>

          <div class="col-md-6">
            <label class="form-label">Company / School</label>
            <input
              v-model="form.institution"
              type="text"
              class="form-control"
              placeholder="Clayheart Studios"
            />
          </div>

          <div class="col-md-6">
            <label class="form-label">Icon</label>
            <select v-model="form.icon" class="form-select">
              <option v-for="icon in iconOptions" :key="icon" :value="icon">
                {{ icon }}
              </option>
            </select>
          </div>

          <div class="col-md-6">
            <label class="form-label">Visibility</label>

            <div class="form-check mt-2">
              <input
                v-model="form.is_active"
                class="form-check-input"
                type="checkbox"
                id="isActive"
              />

              <label class="form-check-label" for="isActive">
                Show this item on portfolio
              </label>
            </div>
          </div>

          <div class="col-md-12">
            <label class="form-label">Description</label>
            <textarea
              v-model="form.description"
              class="form-control"
              rows="4"
              placeholder="Short description"
            ></textarea>
          </div>

          <div class="col-12 d-flex gap-2">
            <button type="submit" class="btn btn-primary" :disabled="isSaving">
              <i class="bi bi-check-circle me-2"></i>
              {{ isSaving ? "Saving..." : editingId ? "Update Item" : "Add Item" }}
            </button>

            <button
              v-if="editingId"
              type="button"
              class="btn btn-outline-dark"
              @click="resetForm"
            >
              Cancel Edit
            </button>
          </div>
        </div>
      </form>
    </div>

    <div v-if="isLoading" class="admin-card">
      <div class="d-flex align-items-center gap-3">
        <div class="spinner-border text-primary" role="status"></div>
        <p class="mb-0 text-muted">Loading resume items...</p>
      </div>
    </div>

    <div v-else class="admin-card">
      <div v-if="resumeItems.length" class="table-responsive">
        <table class="table table-hover align-middle">
          <thead>
            <tr>
              <th>Icon</th>
              <th>Type</th>
              <th>Years</th>
              <th>Title</th>
              <th>Company/School</th>
              <th>Order</th>
              <th>Status</th>
              <th class="text-end">Actions</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="item in resumeItems" :key="item.id">
              <td>
                <div class="stat-icon mb-0">
                  <i :class="getAdminIcon(item.icon)"></i>
                </div>
              </td>

              <td class="text-capitalize">
                {{ item.item_type }}
              </td>

              <td>
                {{ item.years }}
              </td>

              <td class="fw-semibold">
                {{ item.title }}
              </td>

              <td>
                {{ item.institution }}
              </td>

              <td>
                {{ item.sort_order }}
              </td>

              <td>
                <span
                  class="badge"
                  :class="item.is_active ? 'bg-success' : 'bg-secondary'"
                >
                  {{ item.is_active ? "Visible" : "Hidden" }}
                </span>
              </td>

              <td class="text-end">
                <button
                  class="btn btn-sm btn-outline-dark me-2"
                  @click="editResumeItem(item)"
                >
                  Edit
                </button>

                <button
                  class="btn btn-sm btn-outline-danger"
                  @click="deleteResumeItem(item)"
                >
                  Delete
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>

      <p v-else class="text-muted mb-0">
        No resume items found yet.
      </p>
    </div>
  </div>
</template>

<style scoped>
.btn-outline-danger {
  border-color: rgba(220, 53, 69, 0.3);
  color: #dc3545;
}

.btn-outline-danger:hover {
  background: #dc3545;
  color: #fff;
}
</style>