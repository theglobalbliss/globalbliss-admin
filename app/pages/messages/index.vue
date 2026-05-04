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

const messages = ref([]);
const isLoading = ref(true);
const successMessage = ref("");
const errorMessage = ref("");
const selectedMessage = ref(null);

const fetchMessages = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  const { data, error } = await supabase
    .from("contact_messages")
    .select("*")
    .order("created_at", { ascending: false });

  if (error) {
    errorMessage.value = error.message;
    isLoading.value = false;
    return;
  }

  messages.value = data || [];
  isLoading.value = false;
};

const openMessage = async (message) => {
  selectedMessage.value = message;

  if (message.status !== "read") {
    await supabase
      .from("contact_messages")
      .update({
        status: "read",
        updated_at: new Date().toISOString(),
      })
      .eq("id", message.id);

    await fetchMessages();
  }
};

const closeMessage = () => {
  selectedMessage.value = null;
};

const markUnread = async (message) => {
  const { error } = await supabase
    .from("contact_messages")
    .update({
      status: "unread",
      updated_at: new Date().toISOString(),
    })
    .eq("id", message.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  successMessage.value = "Message marked as unread.";
  await fetchMessages();
};

const deleteMessage = async (message) => {
  const confirmed = confirm(
    `Delete message from ${message.name}?`
  );

  if (!confirmed) return;

  successMessage.value = "";
  errorMessage.value = "";

  const { error } = await supabase
    .from("contact_messages")
    .delete()
    .eq("id", message.id);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  if (selectedMessage.value?.id === message.id) {
    selectedMessage.value = null;
  }

  successMessage.value = "Message deleted successfully.";
  await fetchMessages();
};

const formatDate = (date) => {
  if (!date) return "";

  return new Date(date).toLocaleString("en-NG", {
    dateStyle: "medium",
    timeStyle: "short",
  });
};

onMounted(() => {
  fetchMessages();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Contact Messages</h5>
        <p class="text-muted mb-0">
          Read and manage messages sent from your portfolio website.
        </p>
      </div>

      <button class="btn btn-outline-dark" @click="fetchMessages">
        <i class="bi bi-arrow-clockwise me-2"></i>
        Refresh
      </button>
    </div>

    <div v-if="successMessage" class="alert alert-success">
      {{ successMessage }}
    </div>

    <div v-if="errorMessage" class="alert alert-danger">
      {{ errorMessage }}
    </div>

    <div v-if="isLoading" class="admin-card">
      Loading messages...
    </div>

    <div v-else class="row g-4">
      <div class="col-lg-7">
        <div class="admin-card">
          <div class="table-responsive">
            <table class="table table-hover align-middle">
              <thead>
                <tr>
                  <th>Status</th>
                  <th>Sender</th>
                  <th>Subject</th>
                  <th>Date</th>
                  <th class="text-end">Actions</th>
                </tr>
              </thead>

              <tbody>
                <tr v-for="message in messages" :key="message.id">
                  <td>
                    <span
                      class="badge"
                      :class="message.status === 'read' ? 'bg-secondary' : 'bg-success'"
                    >
                      {{ message.status === "read" ? "Read" : "Unread" }}
                    </span>
                  </td>

                  <td>
                    <div class="fw-semibold">
                      {{ message.name }}
                    </div>
                    <div class="small text-muted">
                      {{ message.email }}
                    </div>
                  </td>

                  <td>
                    {{ message.subject || "No subject" }}
                  </td>

                  <td>
                    {{ formatDate(message.created_at) }}
                  </td>

                  <td class="text-end">
                    <button
                      class="btn btn-sm btn-outline-dark me-2"
                      @click="openMessage(message)"
                    >
                      View
                    </button>

                    <button
                      class="btn btn-sm btn-outline-danger"
                      @click="deleteMessage(message)"
                    >
                      Delete
                    </button>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <p v-if="messages.length === 0" class="text-muted mb-0">
            No contact messages yet.
          </p>
        </div>
      </div>

      <div class="col-lg-5">
        <div class="admin-card h-100">
          <template v-if="selectedMessage">
            <div class="d-flex justify-content-between align-items-start mb-3">
              <div>
                <h4 class="fw-bold mb-1">
                  {{ selectedMessage.subject || "No subject" }}
                </h4>
                <p class="text-muted mb-0">
                  From {{ selectedMessage.name }}
                </p>
              </div>

              <button class="btn btn-sm btn-outline-dark" @click="closeMessage">
                Close
              </button>
            </div>

            <div class="mb-3">
              <p class="mb-1">
                <strong>Email:</strong>
                <a :href="`mailto:${selectedMessage.email}`">
                  {{ selectedMessage.email }}
                </a>
              </p>

              <p class="mb-0">
                <strong>Date:</strong>
                {{ formatDate(selectedMessage.created_at) }}
              </p>
            </div>

            <hr />

            <p style="white-space: pre-line; line-height: 1.8;">
              {{ selectedMessage.message }}
            </p>

            <div class="d-flex flex-wrap gap-2 mt-4">
              <a
                :href="`mailto:${selectedMessage.email}?subject=Re: ${selectedMessage.subject || 'Your message to The GlobalBliss Brand'}`"
                class="btn btn-primary"
              >
                <i class="bi bi-reply me-2"></i>
                Reply via Email
              </a>

              <button
                class="btn btn-outline-dark"
                @click="markUnread(selectedMessage)"
              >
                Mark Unread
              </button>

              <button
                class="btn btn-outline-danger"
                @click="deleteMessage(selectedMessage)"
              >
                Delete
              </button>
            </div>
          </template>

          <template v-else>
            <div class="text-center py-5">
              <div class="stat-icon">
                <i class="bi bi-envelope-open"></i>
              </div>

              <h5 class="fw-bold mt-3">
                Select a message
              </h5>

              <p class="text-muted mb-0">
                Click view beside any message to read it here.
              </p>
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>