<template>
  <div class="accordion" ref="accordionRef">
    <div v-for="story in stories" :key="story.id" class="accordion-item">
      <div class="accordion-title-wrapper">
        <button class="accordion-title" @click="toggle(story)">
          <span>{{ story.title }}</span>
        </button>

        <button
          class="icon-btn"
          :class="{
            playing: currentStoryId === story.id && !isPaused,
            paused: currentStoryId === story.id && isPaused,
          }"
          @click.stop="handleMainControl(story)"
        >
          {{ getIcon(story.id) }}
        </button>

        <button
          class="arrow"
          :class="{ open: openId === story.id }"
          @click="toggle(story)"
        >
          ›
        </button>
      </div>

      <transition name="accordion">
        <div v-if="openId === story.id" class="accordion-content">
          <div v-html="story.content"></div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { marked } from "marked";

/* ------------------------------------------
   STATE
------------------------------------------ */

const accordionRef = ref(null);
const openId = ref(null);

const isPlaying = ref(false);
const currentStoryId = ref(null);
const isPaused = ref(false);

let currentAudioInstance = null;
let isStopped = false;

/* ------------------------------------------
   LOAD STORIES
------------------------------------------ */

const modules = import.meta.glob("../stories/*.md", {
  eager: true,
  query: "?raw",
  import: "default",
});

const stories = ref(
  Object.entries(modules).map(([_, content], index) => {
    const titleMatch = content.match(/^#\s+(.*)/);
    return {
      id: index + 1,
      title: titleMatch ? titleMatch[1] : "Untitled Story",
      content: marked(content),
    };
  }),
);

/* ------------------------------------------
   ACCORDION
------------------------------------------ */

const toggle = (story) => {
  openId.value = openId.value === story.id ? null : story.id;
};

/* ------------------------------------------
   TEXT SPLIT (SMART)
------------------------------------------ */

const splitTextSmart = (text) => {
  const maxFirstChunk = 50;
  const maxChunk = 200;

  const sentences = text.split(/(?<=[।.!?])\s+/);

  const chunks = [];
  let currentChunk = "";
  let currentLimit = maxFirstChunk;

  for (const sentence of sentences) {
    if ((currentChunk + sentence).length > currentLimit) {
      chunks.push(currentChunk.trim());
      currentChunk = "";
      currentLimit = maxChunk;
    }
    currentChunk += sentence + " ";
  }

  if (currentChunk.trim()) chunks.push(currentChunk.trim());

  return chunks;
};

/* ------------------------------------------
   FETCH AUDIO
------------------------------------------ */

const fetchChunk = async (chunk) => {
  const response = await fetch("https://api.anshulsharma.net/speak-chunk", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ text: chunk }),
  });

  const blob = await response.blob();
  return URL.createObjectURL(blob);
};

/* ------------------------------------------
   PLAYBACK ENGINE (1 CHUNK LOOKAHEAD)
------------------------------------------ */

const playQueue = async (story) => {
  if (isPlaying.value && currentStoryId.value === story.id) return;

  stopAudio();

  isStopped = false;
  isPlaying.value = true;
  isPaused.value = false;
  currentStoryId.value = story.id;

  const tempDiv = document.createElement("div");
  tempDiv.innerHTML = story.content;
  const text = tempDiv.innerText;

  const chunks = splitTextSmart(text);

  let index = 0;
  let currentUrl = await fetchChunk(chunks[index]);
  let nextPromise = null;

  while (index < chunks.length && !isStopped) {
    const audio = new Audio(currentUrl);
    currentAudioInstance = audio;

    // Preload next chunk
    if (index + 1 < chunks.length) {
      nextPromise = fetchChunk(chunks[index + 1]);
    }

    await new Promise((resolve) => {
      audio.onended = resolve;
      audio.play();
    });

    index++;

    if (nextPromise) {
      currentUrl = await nextPromise;
    }
  }

  if (!isStopped) {
    isPlaying.value = false;
    currentStoryId.value = null;
  }
};

/* ------------------------------------------
   CONTROLS
------------------------------------------ */

const pauseAudio = () => {
  if (currentAudioInstance && !currentAudioInstance.paused) {
    currentAudioInstance.pause();
    isPaused.value = true;
  }
};

const resumeAudio = () => {
  if (currentAudioInstance && currentAudioInstance.paused) {
    currentAudioInstance.play();
    isPaused.value = false;
  }
};

const stopAudio = () => {
  isStopped = true;
  if (currentAudioInstance) {
    currentAudioInstance.pause();
    currentAudioInstance.currentTime = 0;
  }
  isPlaying.value = false;
  isPaused.value = false;
  currentStoryId.value = null;
};

const handleMainControl = (story) => {
  if (!isPlaying.value) {
    playQueue(story);
  } else if (currentStoryId.value !== story.id) {
    playQueue(story);
  } else if (!isPaused.value) {
    pauseAudio();
  } else {
    resumeAudio();
  }
};

const getIcon = (id) => {
  if (currentStoryId.value !== id) return "▶";
  if (isPaused.value) return "▶";
  return "⏸";
};

/* ------------------------------------------
   CLICK OUTSIDE
------------------------------------------ */

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
  flex: 1;
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

/* Arrow styling */
.arrow {
  background: transparent;
  border: none;
  font-size: 22px;
  transition: transform 0.3s ease;
  color: #ffcc33;
  cursor: pointer;
  padding: 4px;
}

.arrow:hover {
  opacity: 0.7;
}

/* Rotate when open */
.arrow.open {
  transform: rotate(90deg);
}

.accordion-title-wrapper {
  display: flex;
  align-items: center;
  padding: 2px 28px;
}

.icon-btn {
  background: transparent;
  border: none;
  color: #ffffff;
  font-size: 18px;
  cursor: pointer;
  margin-right: 12px;
  transition:
    color 0.2s ease,
    opacity 0.2s ease;
}

.icon-btn.playing {
  color: #22c55e;
}

.icon-btn.paused {
  color: #f59e0b;
}

.icon-btn:hover {
  opacity: 0.7;
}
</style>
