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

const password = ref("");
const confirmPassword = ref("");
const errorMessage = ref("");
const successMessage = ref("");
const showPassword = ref(false);
const showConfirmPassword = ref(false);
const isSaving = ref(false);

const logoUrl = "/admin/images/globalbliss-logo.png";

const updatePassword = async () => {
  errorMessage.value = "";
  successMessage.value = "";

  if (!password.value || !confirmPassword.value) {
    errorMessage.value = "Please enter and confirm your new password.";
    return;
  }

  if (password.value.length < 6) {
    errorMessage.value = "Password should be at least 6 characters.";
    return;
  }

  if (password.value !== confirmPassword.value) {
    errorMessage.value = "Passwords do not match.";
    return;
  }

  isSaving.value = true;

  const { error } = await supabase.auth.updateUser({
    password: password.value,
  });

  if (error) {
    errorMessage.value = error.message;
    isSaving.value = false;
    return;
  }

  successMessage.value = "Password updated successfully. Redirecting to login...";

  setTimeout(async () => {
    await supabase.auth.signOut();
    localStorage.removeItem("globalbliss_admin_logged_in");
    navigateTo("/login");
  }, 1200);

  isSaving.value = false;
};
</script>

<template>
  <main class="gb-login-page">
    <div class="gb-login-orb gb-login-orb-one"></div>
    <div class="gb-login-orb gb-login-orb-two"></div>
    <div class="gb-login-grid"></div>

    <section class="gb-reset-card">
      <div class="gb-logo-wrap">
        <img
          :src="logoUrl"
          alt="The GlobalBliss Brand"
        />
      </div>

      <div class="text-center mb-4">
        <p class="gb-login-kicker">
          Admin Recovery
        </p>

        <h2>
          Create New Password
        </h2>

        <p class="gb-login-subtitle">
          Enter a fresh password for your GlobalBliss Admin Dashboard.
        </p>
      </div>

      <form @submit.prevent="updatePassword">
        <div class="gb-form-group">
          <label>New Password</label>

          <div class="gb-input-wrap">
            <i class="bi bi-lock"></i>

            <input
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              placeholder="Enter new password"
              autocomplete="new-password"
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

        <div class="gb-form-group">
          <label>Confirm Password</label>

          <div class="gb-input-wrap">
            <i class="bi bi-shield-lock"></i>

            <input
              v-model="confirmPassword"
              :type="showConfirmPassword ? 'text' : 'password'"
              placeholder="Confirm new password"
              autocomplete="new-password"
              required
            />

            <button
              type="button"
              class="gb-password-toggle"
              @click="showConfirmPassword = !showConfirmPassword"
            >
              <i :class="showConfirmPassword ? 'bi bi-eye-slash' : 'bi bi-eye'"></i>
            </button>
          </div>
        </div>

        <div v-if="errorMessage" class="gb-login-error">
          <i class="bi bi-exclamation-circle"></i>
          {{ errorMessage }}
        </div>

        <div v-if="successMessage" class="gb-login-success">
          <i class="bi bi-check-circle"></i>
          {{ successMessage }}
        </div>

        <button type="submit" class="gb-login-btn" :disabled="isSaving">
          <span>
            {{ isSaving ? "Updating..." : "Update Password" }}
          </span>
          <i class="bi bi-arrow-right"></i>
        </button>

        <button
          type="button"
          class="gb-reset-back"
          @click="navigateTo('/login')"
        >
          Back to Login
        </button>
      </form>
    </section>
  </main>
</template>