<template>
  <div class="accordion">
    <div v-for="story in stories" :key="story.id" class="accordion-item">
      <button class="accordion-title" @click="toggle(story)">
        {{ story.title }}
      </button>

      <transition name="accordion">
        <div v-if="openId === story.id" class="accordion-content">
          <div
            v-for="(paragraph, index) in storyContent[story.id]?.split('\n')"
            :key="index"
            class="paragraph"
          >
            {{ paragraph }}
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const stories = ref([]);
const openId = ref(null);
const storyContent = ref({});

const toggle = async (story) => {
  if (openId.value === story.id) {
    openId.value = null;
    return;
  }

  openId.value = story.id;

  if (!storyContent.value[story.id]) {
    const response = await fetch(`/stories/${story.file}`);
    const text = await response.text();
    storyContent.value[story.id] = text;
  }
};

onMounted(async () => {
  const response = await fetch("/stories/stories.json");
  stories.value = await response.json();
});
</script>

<style scoped>
/* .accordion {
  width: 100%;
}

.accordion-item {
  margin-bottom: 24px;
  border-radius: 18px;
  overflow: hidden;
  background: linear-gradient(
    145deg,
    rgba(255, 255, 255, 0.06),
    rgba(255, 255, 255, 0.03)
  );
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(12px);
  transition: all 0.25s ease;
}

.accordion-item:hover {
  transform: translateY(-4px);
  border-color: rgba(255, 255, 255, 0.15);
}

.accordion-title {
  width: 100%;
  padding: 22px 26px;
  font-size: clamp(18px, 2vw, 22px);
  text-align: left;
  background: transparent;
  color: #ffffff;
  border: none;
  cursor: pointer;
  font-weight: 500;
}

.accordion-content {
  padding: 26px;
  font-size: clamp(16px, 1.2vw, 18px);
  line-height: 1.9;
  background: #ffffff;
  color: #222;
} */

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
  padding: 32px;
  font-size: clamp(16px, 1.2vw, 18px);
  line-height: 1.9;
  background: #ffffff;
  color: #222;
  font-family: "Inter", sans-serif;
}
</style>
