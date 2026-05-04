<script setup>
import { createClient } from "@supabase/supabase-js";

const config = useRuntimeConfig();

const supabase = createClient(
  config.public.supabaseUrl,
  config.public.supabaseAnonKey
);

const isLoading = ref(true);
const isSaving = ref(false);
const isUploadingHero = ref(false);
const successMessage = ref("");
const errorMessage = ref("");

const heroImageFile = ref(null);
const heroImagePreview = ref("");

const sections = ref({
  hero: {
    section_key: "hero",
    title: "",
    subtitle: "",
    body: "",
    button_text: "",
    button_url: "",
    extra_data: {
      secondary_button_text: "",
      secondary_button_url: "",
      hero_image_url: "",
    },
  },
  about: {
    section_key: "about",
    title: "",
    subtitle: "",
    body: "",
    button_text: "",
    button_url: "",
    extra_data: {},
  },
  cta: {
    section_key: "cta",
    title: "",
    subtitle: "",
    body: "",
    button_text: "",
    button_url: "",
    extra_data: {},
  },
});

const socials = ref({
  instagram_url: "",
  linkedin_url: "",
  twitter_url: "",
  facebook_url: "",
  behance_url: "",
  github_url: "",
  whatsapp_url: "",
  email_address: "",
});

const logout = () => {
  localStorage.removeItem("globalbliss_admin_logged_in");
  navigateTo("/login");
};

const normalizeHeroExtraData = () => {
  if (!sections.value.hero.extra_data) {
    sections.value.hero.extra_data = {};
  }

  if (!sections.value.hero.extra_data.secondary_button_text) {
    sections.value.hero.extra_data.secondary_button_text = "";
  }

  if (!sections.value.hero.extra_data.secondary_button_url) {
    sections.value.hero.extra_data.secondary_button_url = "";
  }

  if (!sections.value.hero.extra_data.hero_image_url) {
    sections.value.hero.extra_data.hero_image_url = "";
  }
};

const fetchHomepageContent = async () => {
  const { data, error } = await supabase
    .from("homepage_content")
    .select("*")
    .in("section_key", ["hero", "about", "cta"]);

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  data.forEach((item) => {
    sections.value[item.section_key] = {
      section_key: item.section_key,
      title: item.title || "",
      subtitle: item.subtitle || "",
      body: item.body || "",
      button_text: item.button_text || "",
      button_url: item.button_url || "",
      extra_data: item.extra_data || {},
    };
  });

  normalizeHeroExtraData();
};

const fetchSocialSettings = async () => {
  const { data, error } = await supabase
    .from("site_settings")
    .select("*");

  if (error) {
    errorMessage.value = error.message;
    return;
  }

  data.forEach((item) => {
    if (item.setting_key in socials.value) {
      socials.value[item.setting_key] = item.setting_value || "";
    }
  });
};

const fetchPageData = async () => {
  isLoading.value = true;
  errorMessage.value = "";

  await fetchHomepageContent();
  await fetchSocialSettings();

  isLoading.value = false;
};

const handleHeroImageSelect = (event) => {
  const file = event.target.files?.[0];

  if (!file) return;

  heroImageFile.value = file;
  heroImagePreview.value = URL.createObjectURL(file);
};

const uploadHeroImage = async () => {
  if (!heroImageFile.value) {
    return sections.value.hero.extra_data.hero_image_url;
  }

  isUploadingHero.value = true;

  const file = heroImageFile.value;
  const fileExt = file.name.split(".").pop();
  const fileName = `hero-image-${Date.now()}.${fileExt}`;
  const filePath = `homepage/${fileName}`;

  const { error: uploadError } = await supabase.storage
    .from("project-images")
    .upload(filePath, file, {
      cacheControl: "3600",
      upsert: false,
    });

  if (uploadError) {
    isUploadingHero.value = false;
    throw uploadError;
  }

  const { data } = supabase.storage
    .from("project-images")
    .getPublicUrl(filePath);

  isUploadingHero.value = false;

  return data.publicUrl;
};

const saveSection = async (sectionKey) => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  try {
    const section = sections.value[sectionKey];

    if (sectionKey === "hero") {
      const heroImageUrl = await uploadHeroImage();

      section.extra_data = {
        ...section.extra_data,
        hero_image_url: heroImageUrl,
      };
    }

    const { error } = await supabase
      .from("homepage_content")
      .upsert(
        {
          section_key: section.section_key,
          title: section.title,
          subtitle: section.subtitle,
          body: section.body,
          button_text: section.button_text,
          button_url: section.button_url,
          extra_data: section.extra_data || {},
          updated_at: new Date().toISOString(),
        },
        {
          onConflict: "section_key",
        }
      );

    if (error) {
      throw error;
    }

    successMessage.value = `${sectionKey.toUpperCase()} section updated successfully.`;
    heroImageFile.value = null;
    heroImagePreview.value = "";
  } catch (error) {
    errorMessage.value = error.message || "Unable to save section.";
  } finally {
    isSaving.value = false;
    isUploadingHero.value = false;
  }
};

const saveSocialSettings = async () => {
  isSaving.value = true;
  successMessage.value = "";
  errorMessage.value = "";

  try {
    const payload = Object.entries(socials.value).map(([key, value]) => ({
      setting_key: key,
      setting_value: value,
      updated_at: new Date().toISOString(),
    }));

    const { error } = await supabase
      .from("site_settings")
      .upsert(payload, {
        onConflict: "setting_key",
      });

    if (error) {
      throw error;
    }

    successMessage.value = "Social links and contact settings updated successfully.";
  } catch (error) {
    errorMessage.value = error.message || "Unable to save social settings.";
  } finally {
    isSaving.value = false;
  }
};

onMounted(() => {
  const isLoggedIn = localStorage.getItem("globalbliss_admin_logged_in");

  if (!isLoggedIn) {
    navigateTo("/login");
    return;
  }

  fetchPageData();
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
          <h5 class="fw-bold mb-1">Homepage Content</h5>
          <p class="text-muted mb-0">
            Edit your hero image, homepage text, social links, and contact handles.
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

      <div v-if="isLoading" class="admin-card">
        Loading homepage content...
      </div>

      <template v-else>
        <div v-if="successMessage" class="alert alert-success">
          {{ successMessage }}
        </div>

        <div v-if="errorMessage" class="alert alert-danger">
          {{ errorMessage }}
        </div>

        <section class="admin-card mb-4">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <div>
              <h4 class="fw-bold mb-1">Hero Section</h4>
              <p class="text-muted mb-0">
                Main headline, intro, buttons, and hero image.
              </p>
            </div>

            <button
              class="btn btn-primary"
              :disabled="isSaving || isUploadingHero"
              @click="saveSection('hero')"
            >
              {{ isSaving || isUploadingHero ? "Saving..." : "Save Hero" }}
            </button>
          </div>

          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">Hero Title</label>
              <input
                v-model="sections.hero.title"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Hero Subtitle</label>
              <input
                v-model="sections.hero.subtitle"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-12">
              <label class="form-label">Hero Body</label>
              <textarea
                v-model="sections.hero.body"
                class="form-control"
                rows="4"
              ></textarea>
            </div>

            <div class="col-md-6">
              <label class="form-label">Primary Button Text</label>
              <input
                v-model="sections.hero.button_text"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Primary Button URL</label>
              <input
                v-model="sections.hero.button_url"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Secondary Button Text</label>
              <input
                v-model="sections.hero.extra_data.secondary_button_text"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Secondary Button URL</label>
              <input
                v-model="sections.hero.extra_data.secondary_button_url"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-12">
              <label class="form-label">Hero Image</label>
              <input
                type="file"
                class="form-control"
                accept="image/*"
                @change="handleHeroImageSelect"
              />
              <small class="text-muted">
                Upload the main hero image for your portfolio homepage.
              </small>
            </div>

            <div
              v-if="heroImagePreview || sections.hero.extra_data.hero_image_url"
              class="col-md-12"
            >
              <label class="form-label">Hero Image Preview</label>
              <div>
                <img
                  :src="heroImagePreview || sections.hero.extra_data.hero_image_url"
                  alt="Hero Image Preview"
                  style="max-width: 320px; height: 220px; object-fit: cover; border-radius: 18px;"
                />
              </div>
            </div>
          </div>
        </section>

        <section class="admin-card mb-4">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <div>
              <h4 class="fw-bold mb-1">About Section</h4>
              <p class="text-muted mb-0">
                Manage the homepage about copy.
              </p>
            </div>

            <button
              class="btn btn-primary"
              :disabled="isSaving"
              @click="saveSection('about')"
            >
              {{ isSaving ? "Saving..." : "Save About" }}
            </button>
          </div>

          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">About Title</label>
              <input
                v-model="sections.about.title"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">About Subtitle</label>
              <input
                v-model="sections.about.subtitle"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-12">
              <label class="form-label">About Body</label>
              <textarea
                v-model="sections.about.body"
                class="form-control"
                rows="5"
              ></textarea>
            </div>

            <div class="col-md-6">
              <label class="form-label">Button Text</label>
              <input
                v-model="sections.about.button_text"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Button URL</label>
              <input
                v-model="sections.about.button_url"
                type="text"
                class="form-control"
              />
            </div>
          </div>
        </section>

        <section class="admin-card mb-4">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <div>
              <h4 class="fw-bold mb-1">CTA Section</h4>
              <p class="text-muted mb-0">
                Manage your final call-to-action area.
              </p>
            </div>

            <button
              class="btn btn-primary"
              :disabled="isSaving"
              @click="saveSection('cta')"
            >
              {{ isSaving ? "Saving..." : "Save CTA" }}
            </button>
          </div>

          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">CTA Title</label>
              <input
                v-model="sections.cta.title"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">CTA Subtitle</label>
              <input
                v-model="sections.cta.subtitle"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-12">
              <label class="form-label">CTA Body</label>
              <textarea
                v-model="sections.cta.body"
                class="form-control"
                rows="4"
              ></textarea>
            </div>

            <div class="col-md-6">
              <label class="form-label">Button Text</label>
              <input
                v-model="sections.cta.button_text"
                type="text"
                class="form-control"
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Button URL</label>
              <input
                v-model="sections.cta.button_url"
                type="text"
                class="form-control"
              />
            </div>
          </div>
        </section>

        <section class="admin-card">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <div>
              <h4 class="fw-bold mb-1">Social Handles & Contact Links</h4>
              <p class="text-muted mb-0">
                Manage the links used on your portfolio website.
              </p>
            </div>

            <button
              class="btn btn-primary"
              :disabled="isSaving"
              @click="saveSocialSettings"
            >
              {{ isSaving ? "Saving..." : "Save Socials" }}
            </button>
          </div>

          <div class="row g-3">
            <div class="col-md-6">
              <label class="form-label">Instagram URL</label>
              <input
                v-model="socials.instagram_url"
                type="text"
                class="form-control"
                placeholder="https://instagram.com/..."
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">LinkedIn URL</label>
              <input
                v-model="socials.linkedin_url"
                type="text"
                class="form-control"
                placeholder="https://linkedin.com/in/..."
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Twitter/X URL</label>
              <input
                v-model="socials.twitter_url"
                type="text"
                class="form-control"
                placeholder="https://x.com/..."
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Facebook URL</label>
              <input
                v-model="socials.facebook_url"
                type="text"
                class="form-control"
                placeholder="https://facebook.com/..."
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Behance URL</label>
              <input
                v-model="socials.behance_url"
                type="text"
                class="form-control"
                placeholder="https://behance.net/..."
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">GitHub URL</label>
              <input
                v-model="socials.github_url"
                type="text"
                class="form-control"
                placeholder="https://github.com/..."
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">WhatsApp URL</label>
              <input
                v-model="socials.whatsapp_url"
                type="text"
                class="form-control"
                placeholder="https://wa.me/234..."
              />
            </div>

            <div class="col-md-6">
              <label class="form-label">Email Address</label>
              <input
                v-model="socials.email_address"
                type="email"
                class="form-control"
                placeholder="hello@example.com"
              />
            </div>
          </div>
        </section>
      </template>
    </main>
  </div>
</template>