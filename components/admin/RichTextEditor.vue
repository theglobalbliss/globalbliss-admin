<script setup>
import { watch } from "vue";
import { EditorContent, useEditor } from "@tiptap/vue-3";
import StarterKit from "@tiptap/starter-kit";
import Underline from "@tiptap/extension-underline";
import Link from "@tiptap/extension-link";

const props = defineProps({
  modelValue: {
    type: String,
    default: "",
  },
});

const emit = defineEmits(["update:modelValue"]);

const editor = useEditor({
  content: props.modelValue || "",
  extensions: [
    StarterKit,
    Underline,
    Link.configure({
      openOnClick: false,
      HTMLAttributes: {
        target: "_blank",
        rel: "noopener noreferrer",
      },
    }),
  ],
  editorProps: {
    attributes: {
      class: "rich-editor-content",
    },
  },
  onUpdate({ editor }) {
    emit("update:modelValue", editor.getHTML());
  },
});

watch(
  () => props.modelValue,
  (value) => {
    if (!editor.value) return;

    const currentContent = editor.value.getHTML();

    if (value !== currentContent) {
      editor.value.commands.setContent(value || "", false);
    }
  }
);

const setLink = () => {
  if (!editor.value) return;

  const previousUrl = editor.value.getAttributes("link").href;
  const url = window.prompt(
    "Enter link URL. Use /about for internal links or https://example.com for external links.",
    previousUrl || ""
  );

  if (url === null) return;

  if (url === "") {
    editor.value.chain().focus().extendMarkRange("link").unsetLink().run();
    return;
  }

  editor.value
    .chain()
    .focus()
    .extendMarkRange("link")
    .setLink({ href: url })
    .run();
};

const transformSelection = (type) => {
  if (!editor.value) return;

  const { state } = editor.value;
  const { from, to } = state.selection;

  if (from === to) {
    alert("Highlight the text you want to transform first.");
    return;
  }

  const selectedText = state.doc.textBetween(from, to, " ");

  let transformedText = selectedText;

  if (type === "uppercase") {
    transformedText = selectedText.toUpperCase();
  }

  if (type === "capitalize") {
    transformedText = selectedText
      .toLowerCase()
      .replace(/\b\w/g, (letter) => letter.toUpperCase());
  }

  editor.value.chain().focus().insertContentAt({ from, to }, transformedText).run();
};
</script>

<template>
  <div class="rich-editor-wrapper">
    <div v-if="editor" class="rich-editor-toolbar">
      <button
        type="button"
        :class="{ active: editor.isActive('bold') }"
        @click="editor.chain().focus().toggleBold().run()"
      >
        Bold
      </button>

      <button
        type="button"
        :class="{ active: editor.isActive('italic') }"
        @click="editor.chain().focus().toggleItalic().run()"
      >
        Italic
      </button>

      <button
        type="button"
        :class="{ active: editor.isActive('underline') }"
        @click="editor.chain().focus().toggleUnderline().run()"
      >
        Underline
      </button>

      <button
        type="button"
        :class="{ active: editor.isActive('heading', { level: 2 }) }"
        @click="editor.chain().focus().toggleHeading({ level: 2 }).run()"
      >
        H2
      </button>

      <button
        type="button"
        :class="{ active: editor.isActive('heading', { level: 3 }) }"
        @click="editor.chain().focus().toggleHeading({ level: 3 }).run()"
      >
        H3
      </button>

      <button
        type="button"
        :class="{ active: editor.isActive('bulletList') }"
        @click="editor.chain().focus().toggleBulletList().run()"
      >
        Bullets
      </button>

      <button
        type="button"
        :class="{ active: editor.isActive('orderedList') }"
        @click="editor.chain().focus().toggleOrderedList().run()"
      >
        Numbers
      </button>

      <button type="button" @click="setLink">
        Link
      </button>

      <button
        type="button"
        @click="editor.chain().focus().unsetLink().run()"
      >
        Unlink
      </button>

      <button type="button" @click="transformSelection('uppercase')">
        Uppercase
      </button>

      <button type="button" @click="transformSelection('capitalize')">
        Capitalize
      </button>

      <button
        type="button"
        @click="editor.chain().focus().clearNodes().unsetAllMarks().run()"
      >
        Clear
      </button>
    </div>

    <EditorContent :editor="editor" />
  </div>
</template>

<style scoped>
.rich-editor-wrapper {
  border: 1px solid rgba(17, 17, 17, 0.12);
  border-radius: 16px;
  overflow: hidden;
  background: #ffffff;
}

.rich-editor-toolbar {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
  padding: 12px;
  background: rgba(235, 93, 58, 0.06);
  border-bottom: 1px solid rgba(17, 17, 17, 0.08);
}

.rich-editor-toolbar button {
  border: 1px solid rgba(17, 17, 17, 0.12);
  background: #ffffff;
  color: #111111;
  border-radius: 999px;
  padding: 7px 12px;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  transition: 0.2s ease;
}

.rich-editor-toolbar button:hover,
.rich-editor-toolbar button.active {
  background: #eb5d3a;
  color: #ffffff;
  border-color: #eb5d3a;
}

:deep(.rich-editor-content) {
  min-height: 320px;
  padding: 18px;
  outline: none;
  color: #111111;
  line-height: 1.8;
}

:deep(.rich-editor-content p) {
  margin-bottom: 14px;
}

:deep(.rich-editor-content h2) {
  font-size: 26px;
  margin: 24px 0 12px;
  font-weight: 800;
}

:deep(.rich-editor-content h3) {
  font-size: 21px;
  margin: 20px 0 10px;
  font-weight: 700;
}

:deep(.rich-editor-content ul),
:deep(.rich-editor-content ol) {
  padding-left: 24px;
  margin-bottom: 16px;
}

:deep(.rich-editor-content a) {
  color: #eb5d3a;
  text-decoration: underline;
  font-weight: 600;
}
</style>