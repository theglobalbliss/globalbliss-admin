<script setup>
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const isCheckingAuth = ref(true);

const logout = async () => {
  await supabase.auth.signOut();

  localStorage.removeItem("globalbliss_admin_logged_in");

  navigateTo("/login");
};

onMounted(async () => {
  const { data } = await supabase.auth.getSession();

  if (!data.session) {
    localStorage.removeItem("globalbliss_admin_logged_in");
    navigateTo("/login");
    return;
  }

  localStorage.setItem("globalbliss_admin_logged_in", "true");
  isCheckingAuth.value = false;
});
</script>

<template>
  <div>
    <div
      v-if="isCheckingAuth"
      class="d-flex align-items-center justify-content-center"
      style="min-height: 100vh;"
    >
      <div class="text-center">
        <div
          class="spinner-border text-primary mb-3"
          role="status"
        ></div>

        <p class="text-muted mb-0">
          Checking admin access...
        </p>
      </div>
    </div>

    <div v-else>
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

        <NuxtLink to="/analytics">
          <i class="bi bi-graph-up-arrow"></i>
          Analytics
        </NuxtLink>

        <NuxtLink to="/content/homepage">
          <i class="bi bi-pencil-square"></i>
          Homepage Content
        </NuxtLink>

        <NuxtLink to="/content/about">
          <i class="bi bi-person-lines-fill"></i>
          About Content
        </NuxtLink>

        <NuxtLink to="/resume">
          <i class="bi bi-person-workspace"></i>
          Resume
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

        <NuxtLink to="/messages">
          <i class="bi bi-envelope"></i>
          Messages
        </NuxtLink>

        <a href="#" class="logout-link" @click.prevent="logout">
          <i class="bi bi-box-arrow-left"></i>
          Logout
        </a>
      </aside>

      <main class="admin-main">
        <slot />
      </main>
    </div>
  </div>
</template>