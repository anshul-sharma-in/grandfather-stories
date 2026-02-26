<template>
  <div class="accordion" ref="accordionRef">
    <div v-for="story in stories" :key="story.id" class="accordion-item">
      <button class="accordion-title" @click="toggle(story)">
        <span>{{ story.title }}</span>

        <span class="arrow" :class="{ open: openId === story.id }"> › </span>
      </button>

      <transition name="accordion">
        <div
          v-if="openId === story.id"
          class="accordion-content"
          v-html="story.content"
        ></div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { marked } from "marked";

const accordionRef = ref(null);
const openId = ref(null);

const modules = import.meta.glob("../stories/*.md", {
  eager: true,
  query: '?raw',
  import: 'default',
});

const stories = ref(
  Object.entries(modules).map(([path, content], index) => {
    const titleMatch = content.match(/^#\s+(.*)/);
    return {
      id: index + 1,
      title: titleMatch ? titleMatch[1] : "Untitled Story",
      content: marked(content),
    };
  }),
);

const toggle = (story) => {
  openId.value = openId.value === story.id ? null : story.id;
};

const handleClickOutside = (event) => {
  if (accordionRef.value && !accordionRef.value.contains(event.target)) {
    openId.value = null;
  }
};

onMounted(() => {
  document.addEventListener("click", handleClickOutside);
});

onBeforeUnmount(() => {
  document.removeEventListener("click", handleClickOutside);
});
</script>

<style scoped>
.accordion-item {
  margin-bottom: 28px;
  border-radius: 20px;
  overflow: hidden;
  background: linear-gradient(
    145deg,
    rgba(255, 255, 255, 0.06),
    rgba(255, 255, 255, 0.02)
  );
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(14px);
  transition:
    transform 0.25s ease,
    box-shadow 0.25s ease;
}

.accordion-item:hover {
  transform: translateY(-6px);
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
}

.accordion-title {
  width: 100%;
  padding: 24px 28px;
  font-size: clamp(18px, 1.5vw, 22px);
  text-align: left;
  background: transparent;
  color: #ffffff;
  border: none;
  cursor: pointer;
  font-weight: 500;
  letter-spacing: 0.3px;
}

.accordion-enter-active,
.accordion-leave-active {
  transition: all 0.35s ease;
  overflow: hidden;
}

.accordion-enter-from,
.accordion-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.accordion-content {
  padding: 40px;
  font-size: clamp(16px, 1.1vw, 18px);
  line-height: 1.9;
  background: #ffffff;
  color: #222;
  font-family: "Inter", sans-serif;
  border-top: 1px solid rgba(0, 0, 0, 0.06);
}

/* Markdown styling */
.accordion-content h1 {
  font-size: 28px;
  margin-bottom: 20px;
  font-weight: 600;
}

.accordion-content h2 {
  font-size: 22px;
  margin-top: 30px;
  margin-bottom: 12px;
}

.accordion-content p {
  margin-bottom: 18px;
}

.accordion-content hr {
  margin: 30px 0;
  border: none;
  height: 1px;
  background: rgba(0, 0, 0, 0.08);
}

.accordion-title {
  width: 100%;
  padding: 24px 28px;
  font-size: clamp(18px, 1.5vw, 22px);
  text-align: left;
  background: transparent;
  color: #ffffff;
  border: none;
  cursor: pointer;
  font-weight: 500;
  letter-spacing: 0.3px;

  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* Arrow styling */
.arrow {
  font-size: 22px;
  transition: transform 0.3s ease;
  color: #ffcc33;
}

/* Rotate when open */
.arrow.open {
  transform: rotate(90deg);
}
</style>
