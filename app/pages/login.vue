<script setup>
import { createClient } from "@supabase/supabase-js";

definePageMeta({
  layout: false,
});

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const email = ref("");
const password = ref("");
const errorMessage = ref("");
const successMessage = ref("");
const showPassword = ref(false);
const showForgotPassword = ref(false);
const isLoading = ref(false);
const isSendingReset = ref(false);

const logoUrl = "/admin/images/globalbliss-logo.png";

const login = async () => {
  errorMessage.value = "";
  successMessage.value = "";
  isLoading.value = true;

  const { data, error } = await supabase.auth.signInWithPassword({
    email: email.value,
    password: password.value,
  });

  if (error) {
    errorMessage.value = "Invalid email or password.";
    isLoading.value = false;
    return;
  }

  if (data.session) {
    localStorage.setItem("globalbliss_admin_logged_in", "true");
    navigateTo("/dashboard");
  }

  isLoading.value = false;
};

const openForgotPassword = () => {
  errorMessage.value = "";
  successMessage.value = "";
  showForgotPassword.value = true;
};

const closeForgotPassword = () => {
  showForgotPassword.value = false;
};

const sendPasswordReset = async () => {
  errorMessage.value = "";
  successMessage.value = "";

  if (!email.value) {
    errorMessage.value =
      "Enter your admin email first, then click forgot password again.";
    return;
  }

  isSendingReset.value = true;

  const { error } = await supabase.auth.resetPasswordForEmail(email.value, {
    redirectTo: `${window.location.origin}/admin/reset-password`,
  });

  if (error) {
    errorMessage.value = error.message;
    isSendingReset.value = false;
    return;
  }

  successMessage.value =
    "Password reset email sent. Check your inbox for the recovery link.";

  isSendingReset.value = false;
  showForgotPassword.value = false;
};

onMounted(async () => {
  const { data } = await supabase.auth.getSession();

  if (data.session) {
    localStorage.setItem("globalbliss_admin_logged_in", "true");
    navigateTo("/dashboard");
  }
});
</script>

<template>
  <main class="gb-login-page">
    <div class="gb-login-orb gb-login-orb-one"></div>
    <div class="gb-login-orb gb-login-orb-two"></div>
    <div class="gb-login-grid"></div>

    <section class="gb-login-shell">
      <div class="gb-login-brand-panel">
        <div class="gb-brand-badge">
          <i class="bi bi-stars"></i>
          The GlobalBliss Brand
        </div>

        <h1>
          Your creative control room, built with bliss.
        </h1>

        <p>
          Manage projects, services, testimonials, blog posts, homepage content,
          social handles, visuals, and everything that powers your portfolio.
        </p>

        <div class="gb-login-highlights">
          <div>
            <span>
              <i class="bi bi-folder2-open"></i>
            </span>
            Projects Manager
          </div>

          <div>
            <span>
              <i class="bi bi-journal-text"></i>
            </span>
            Blog Manager
          </div>

          <div>
            <span>
              <i class="bi bi-cloud-upload"></i>
            </span>
            Supabase Uploads
          </div>
        </div>

        <div class="gb-login-quote">
          <p>
            “Design, structure, strategy and digital presence, all from one
            beautiful dashboard.”
          </p>
        </div>
      </div>

      <div class="gb-login-card">
        <div class="gb-logo-wrap">
          <img
            :src="logoUrl"
            alt="The GlobalBliss Brand"
          />
        </div>

        <div class="text-center mb-4">
          <p class="gb-login-kicker">
            Admin Dashboard
          </p>

          <h2>
            Welcome Back
          </h2>

          <p class="gb-login-subtitle">
            Sign in to continue managing The GlobalBliss portfolio.
          </p>
        </div>

        <form @submit.prevent="login">
          <div class="gb-form-group">
            <label>Email Address</label>

            <div class="gb-input-wrap">
              <i class="bi bi-envelope"></i>

              <input
                v-model.trim="email"
                type="email"
                placeholder="Enter admin email"
                autocomplete="email"
                required
              />
            </div>
          </div>

          <div class="gb-form-group">
            <label>Password</label>

            <div class="gb-input-wrap">
              <i class="bi bi-lock"></i>

              <input
                v-model="password"
                :type="showPassword ? 'text' : 'password'"
                placeholder="Enter password"
                autocomplete="current-password"
                required
              />

              <button
                type="button"
                class="gb-password-toggle"
                @click="showPassword = !showPassword"
              >
                <i :class="showPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"></i>
              </button>
            </div>
          </div>

          <div class="gb-login-meta">
            <label>
              <input type="checkbox" checked />
              Keep me logged in
            </label>

            <button
              type="button"
              class="gb-forgot-btn"
              @click="openForgotPassword"
            >
              Forgot password?
            </button>
          </div>

          <div v-if="errorMessage" class="gb-login-error">
            <i class="bi bi-exclamation-circle"></i>
            {{ errorMessage }}
          </div>

          <div v-if="successMessage" class="gb-login-success">
            <i class="bi bi-check-circle"></i>
            {{ successMessage }}
          </div>

          <button type="submit" class="gb-login-btn" :disabled="isLoading">
            <span>
              {{ isLoading ? "Logging in..." : "Login to Dashboard" }}
            </span>
            <i class="bi bi-arrow-right"></i>
          </button>
        </form>

        <p class="gb-login-footer">
          The GlobalBliss Admin Dashboard
        </p>
      </div>
    </section>

    <div
      v-if="showForgotPassword"
      class="gb-forgot-overlay"
      @click.self="closeForgotPassword"
    >
      <div class="gb-forgot-modal">
        <button
          type="button"
          class="gb-forgot-close"
          @click="closeForgotPassword"
        >
          <i class="bi bi-x-lg"></i>
        </button>

        <div class="gb-forgot-icon">
          <i class="bi bi-shield-lock"></i>
        </div>

        <h3>
          Password Recovery
        </h3>

        <p>
          Enter your admin email in the login form first. Then click the button
          below to receive a password reset email.
        </p>

        <div class="gb-forgot-note">
          <strong>Current email:</strong>
          {{ email || "No email entered yet." }}
        </div>

        <button
          type="button"
          class="gb-forgot-action"
          :disabled="isSendingReset"
          @click="sendPasswordReset"
        >
          <i class="bi bi-envelope"></i>
          {{ isSendingReset ? "Sending..." : "Send Password Reset Email" }}
        </button>

        <button
          type="button"
          class="gb-forgot-secondary"
          @click="closeForgotPassword"
        >
          Back to Login
        </button>
      </div>
    </div>
  </main>
</template>