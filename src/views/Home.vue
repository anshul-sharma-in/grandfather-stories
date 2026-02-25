<template>
  <div class="page">
    <div class="background"></div>

    <main class="container">
      <h1 class="title">नाना की कहानियाँ</h1>
      <StoryAccordion />
    </main>
  </div>
</template>

<script setup>
import StoryAccordion from "../components/StoryAccordion.vue"
</script>

<style scoped>
.page {
  min-height: 100vh;
  position: relative;
  background: #0f1115;
  overflow-x: hidden;
}

/* Soft background layer */
.background {
  position: fixed;
  inset: 0;
  background: radial-gradient(
      circle at 20% 30%,
      rgba(255, 200, 150, 0.08),
      transparent 40%
    ),
    radial-gradient(
      circle at 80% 70%,
      rgba(150, 200, 255, 0.08),
      transparent 40%
    );
  z-index: 0;
}

/* Main content container */
.container {
  position: relative;
  z-index: 1;
  max-width: 860px;
  margin: 0 auto;
  padding: 80px 24px 120px 24px;
}

/* Title */
/* .title {
  text-align: center;
  font-size: clamp(32px, 5vw, 48px);
  font-weight: 600;
  margin-bottom: 60px;
  color: #ffffff;
  letter-spacing: 0.5px;
} */

.title {
  font-family: 'Playfair Display', serif;
  text-align: center;
  font-size: clamp(38px, 4vw, 56px);
  font-weight: 600;
  margin-bottom: 80px;
  color: #ffffff;
  letter-spacing: 0.5px;
}

.divider {
  width: 80px;
  height: 3px;
  background: linear-gradient(to right, #ffb347, #ffcc33);
  margin: 0 auto 60px auto;
  border-radius: 2px;
}
</style>