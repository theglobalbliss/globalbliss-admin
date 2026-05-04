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

const heroContent = ref({
  title: "Manage your portfolio with clarity, beauty, and bliss.",
  subtitle: "The GlobalBliss Brand",
  body: "Add new works, update project visuals, arrange your portfolio order, and keep your public website fresh from one simple dashboard.",
  button_text: "Add New Project",
  button_url: "/projects/add",
});

const projectCount = ref("0");
const serviceCount = ref("0");
const testimonialCount = ref("0");
const blogCount = ref("0");
const messageCount = ref("0");
const unreadMessageCount = ref("0");

const fetchDashboardContent = async () => {
  const { data, error } = await supabase
    .from("homepage_content")
    .select("*")
    .eq("section_key", "hero")
    .single();

  if (!error && data) {
    heroContent.value = {
      title: data.title || heroContent.value.title,
      subtitle: data.subtitle || heroContent.value.subtitle,
      body: data.body || heroContent.value.body,
      button_text: data.button_text || heroContent.value.button_text,
      button_url: data.button_url || heroContent.value.button_url,
    };
  }
};

const fetchProjectCount = async () => {
  const { count, error } = await supabase
    .from("projects")
    .select("*", { count: "exact", head: true });

  if (!error && count !== null) {
    projectCount.value = count;
  }
};

const fetchServiceCount = async () => {
  const { count, error } = await supabase
    .from("services")
    .select("*", { count: "exact", head: true });

  if (!error && count !== null) {
    serviceCount.value = count;
  }
};

const fetchTestimonialCount = async () => {
  const { count, error } = await supabase
    .from("testimonials")
    .select("*", { count: "exact", head: true });

  if (!error && count !== null) {
    testimonialCount.value = count;
  }
};

const fetchBlogCount = async () => {
  const { count, error } = await supabase
    .from("blog_posts")
    .select("*", { count: "exact", head: true });

  if (!error && count !== null) {
    blogCount.value = count;
  }
};

const fetchMessageCount = async () => {
  const { count, error } = await supabase
    .from("contact_messages")
    .select("*", { count: "exact", head: true });

  if (!error && count !== null) {
    messageCount.value = count;
  }
};

const fetchUnreadMessageCount = async () => {
  const { count, error } = await supabase
    .from("contact_messages")
    .select("*", { count: "exact", head: true })
    .eq("status", "unread");

  if (!error && count !== null) {
    unreadMessageCount.value = count;
  }
};

onMounted(() => {
  fetchDashboardContent();
  fetchProjectCount();
  fetchServiceCount();
  fetchTestimonialCount();
  fetchBlogCount();
  fetchMessageCount();
  fetchUnreadMessageCount();
});
</script>

<template>
  <div>
    <div class="admin-topbar d-flex justify-content-between align-items-center">
      <div>
        <h5 class="fw-bold mb-1">Hello, GlobalBliss</h5>
        <p class="text-muted mb-0">
          Your creative control room is live and breathing.
        </p>
      </div>

      <a
        href="https://theglobalbliss.online"
        target="_blank"
        class="btn btn-outline-dark"
      >
        <i class="bi bi-box-arrow-up-right me-2"></i>
        View Website
      </a>
    </div>

    <section class="hero-panel mb-4">
      <div class="position-relative">
        <p class="text-uppercase small mb-2" style="letter-spacing: 1.6px;">
          {{ heroContent.subtitle }}
        </p>

        <h1 class="display-5 mb-3">
          {{ heroContent.title }}
        </h1>

        <p class="mb-4">
          {{ heroContent.body }}
        </p>

        <div class="d-flex flex-wrap gap-2">
          <NuxtLink to="/projects/add" class="btn btn-primary">
            <i class="bi bi-plus-circle me-2"></i>
            Add New Project
          </NuxtLink>

          <NuxtLink to="/blog/add" class="btn btn-light">
            <i class="bi bi-pencil me-2"></i>
            Add Blog Post
          </NuxtLink>

          <NuxtLink to="/messages" class="btn btn-outline-light">
            <i class="bi bi-envelope me-2"></i>
            View Messages
          </NuxtLink>

          <NuxtLink to="/content/homepage" class="btn btn-outline-light">
            <i class="bi bi-pencil-square me-2"></i>
            Edit Homepage
          </NuxtLink>
        </div>
      </div>
    </section>

    <div class="row g-4 mb-4">
      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-folder2-open"></i>
          </div>
          <p class="stat-value">{{ projectCount }}</p>
          <p class="stat-label">Portfolio Projects</p>
        </div>
      </div>

      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-briefcase"></i>
          </div>
          <p class="stat-value">{{ serviceCount }}</p>
          <p class="stat-label">Services</p>
        </div>
      </div>

      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-journal-text"></i>
          </div>
          <p class="stat-value">{{ blogCount }}</p>
          <p class="stat-label">Blog Posts</p>
        </div>
      </div>

      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-envelope"></i>
          </div>
          <p class="stat-value">{{ unreadMessageCount }}</p>
          <p class="stat-label">Unread Messages</p>
        </div>
      </div>
    </div>

    <div class="row g-4 mb-4">
      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-chat-quote"></i>
          </div>
          <p class="stat-value">{{ testimonialCount }}</p>
          <p class="stat-label">Testimonials</p>
        </div>
      </div>

      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-inbox"></i>
          </div>
          <p class="stat-value">{{ messageCount }}</p>
          <p class="stat-label">Total Messages</p>
        </div>
      </div>

      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-cloud-upload"></i>
          </div>
          <p class="stat-value">Live</p>
          <p class="stat-label">Supabase Uploads</p>
        </div>
      </div>

      <div class="col-md-3">
        <div class="admin-card stat-card">
          <div class="stat-icon">
            <i class="bi bi-shield-lock"></i>
          </div>
          <p class="stat-value">Auth</p>
          <p class="stat-label">Protected Admin</p>
        </div>
      </div>
    </div>

    <div class="row g-4">
      <div class="col-lg-7">
        <div class="admin-card h-100">
          <h4 class="fw-bold mb-3">Quick Actions</h4>

          <div class="d-grid gap-3">
            <NuxtLink to="/projects/add" class="quick-action">
              <div>
                <h6 class="fw-bold mb-1">Add a new portfolio project</h6>
                <p class="text-muted mb-0">
                  Upload image, add project details, and publish instantly.
                </p>
              </div>
              <i class="bi bi-arrow-right-circle"></i>
            </NuxtLink>

            <NuxtLink to="/blog" class="quick-action">
              <div>
                <h6 class="fw-bold mb-1">Manage blog posts</h6>
                <p class="text-muted mb-0">
                  Create, edit, publish, draft, or delete blog articles and brand stories.
                </p>
              </div>
              <i class="bi bi-journal-text"></i>
            </NuxtLink>

            <NuxtLink to="/messages" class="quick-action">
              <div>
                <h6 class="fw-bold mb-1">Read contact messages</h6>
                <p class="text-muted mb-0">
                  View, reply, mark unread, and delete messages from your portfolio form.
                </p>
              </div>
              <i class="bi bi-envelope"></i>
            </NuxtLink>

            <NuxtLink to="/services" class="quick-action">
              <div>
                <h6 class="fw-bold mb-1">Manage services</h6>
                <p class="text-muted mb-0">
                  Add, edit, hide, delete, and arrange your homepage services.
                </p>
              </div>
              <i class="bi bi-briefcase"></i>
            </NuxtLink>

            <NuxtLink to="/testimonials" class="quick-action">
              <div>
                <h6 class="fw-bold mb-1">Manage testimonials</h6>
                <p class="text-muted mb-0">
                  Add client reviews, update ratings, upload photos, and control visibility.
                </p>
              </div>
              <i class="bi bi-chat-quote"></i>
            </NuxtLink>

            <NuxtLink to="/content/homepage" class="quick-action">
              <div>
                <h6 class="fw-bold mb-1">Edit homepage content</h6>
                <p class="text-muted mb-0">
                  Update hero, about, CTA, social handles, and hero image.
                </p>
              </div>
              <i class="bi bi-pencil-square"></i>
            </NuxtLink>
          </div>
        </div>
      </div>

      <div class="col-lg-5">
        <div class="admin-card h-100">
          <h4 class="fw-bold mb-3">Admin Notes</h4>

          <div class="p-3 rounded-4" style="background: rgba(235, 93, 58, 0.08);">
            <p class="mb-2 fw-semibold">
              Current setup
            </p>
            <p class="text-muted mb-0">
              This dashboard updates your Supabase database. Your portfolio
              will read from the same database, so changes can reflect without
              touching the website code.
            </p>
          </div>

          <hr />

          <p class="text-muted mb-2">
            Admin modules:
          </p>

          <ul class="text-muted mb-0">
            <li>Projects Manager</li>
            <li>Services Manager</li>
            <li>Testimonials Manager</li>
            <li>Blog Manager</li>
            <li>Contact Messages Manager</li>
            <li>Homepage Content Manager</li>
            <li>Supabase Auth Protection</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>