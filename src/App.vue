<template>
  <div class="aqua-container">
    
    <!-- Floating Background Animations -->
    <div class="bubbles-container">
      <div class="bubble" v-for="n in 12" :key="n"></div>
    </div>

    <!-- Header Text -->
    <div class="header-prompt" :class="{ 'fade-out': !isClosed }">
      <p>Love journey awaits ...</p>
    </div>

    <!-- The Levitating Book -->
    <div class="book-levitate-wrapper">
      <div class="book-wrapper">
        <div ref="bookRef" class="st-book">

          <div v-for="(page, index) in pages" :key="index" class="book-page" :class="page.cssClass">

            <!-- Front & Back Cover Layout -->
            <div v-if="page.type === 'cover'" class="page-content cover-layout">
              <div class="cover-border">
                <h1 class="title-text">{{ page.title }}</h1>
                <p class="subtitle-text">{{ page.subtitle }}</p>
              </div>
            </div>

            <!-- Blank Inside Pages -->
            <div v-else-if="page.type === 'blank'" class="page-content blank-layout"></div>

            <!-- Video Page Layout -->
            <div v-else-if="page.type === 'video'" class="page-content content-layout">
              <div class="media-frame">
                <video :src="page.video" controls class="book-video" playsinline></video>
              </div>
              <div class="text-frame" v-if="page.title || page.text">
                <h2 class="content-title">{{ page.title }}</h2>
                <p class="content-text">{{ page.text }}</p>
              </div>
            </div>

            <!-- Image Content Pages -->
            <div v-else class="page-content content-layout">
              <div v-if="page.image" class="media-frame">
                <img :src="page.image" :alt="page.title || 'Our Memory'" />
              </div>
              <div class="text-frame" v-if="page.title || page.text">
                <h2 class="content-title">{{ page.title }}</h2>
                <p class="content-text">{{ page.text }}</p>
              </div>
            </div>

          </div>
        </div>
      </div>
    </div>

    <!-- Smart Controls -->
    <div class="elegant-controls">
      <button v-if="isClosed" @click="goNext" class="action-btn primary-btn">
        Open Book
      </button>

      <template v-else>
        <button @click="goPrev" class="action-btn text-btn" :disabled="currentPage === 0">
          &#8592; Previous
        </button>
        <button @click="goNext" class="action-btn text-btn" :disabled="isAtEnd">
          Next &#8594;
        </button>
      </template>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount, nextTick } from 'vue';
import { PageFlip } from 'page-flip';

const bookRef = ref(null);
let pageFlipInstance = null;

const isClosed = ref(true);
const currentPage = ref(0);
const isAtEnd = ref(false);

const pages = ref([
  { type: "cover", title: "Book for My Nin🩵", subtitle: "Memory awaits", cssClass: "hardcover" },
  { type: "blank", cssClass: "inside-paper" },

  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1780830765/Messenger_creation_0A42DF5D-014F-4A98-87DF-EB069DD9F1DE_pm1f2k.jpg", title: "Where It Began", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1780476624/Messenger_creation_6647328F-2A97-4BA4-8236-B58D12011FEB_n4299h.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1780476623/Messenger_creation_6642F064-014D-4C29-AF73-E5C5779A6945_fimaqd.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193927/photo_2026-05-19_19-31-45_tbusdu.jpg", title: "Beautiful Moments", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193926/photo_2026-05-19_19-31-46_revl1a.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193924/photo_2026-05-19_19-31-44_gxfbkt.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193126/photo_2026-05-19_19-18-06_xolokj.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193126/photo_2026-05-19_19-17-50_d3gie9.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193125/photo_2026-05-19_19-17-52_fbwxfr.jpg", title: "Smiles & Laughter", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193123/photo_2026-05-19_19-17-55_rfrtc2.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193122/photo_2026-05-19_19-17-57_evfl1p.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193121/photo_2026-05-19_19-17-53_lieyfy.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193120/photo_2026-05-19_19-18-01_jblfmp.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193115/photo_2026-05-19_19-18-08_fghmy9.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193114/photo_2026-05-19_19-18-13_vkdrao.jpg", title: "Unforgettable", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193111/photo_2026-05-19_19-18-14_mmh46v.jpg", cssClass: "inside-paper" },
  { type: "content", image: "https://res.cloudinary.com/dvljcimlz/image/upload/v1779193115/photo_2026-05-19_19-18-10_qcetj1.jpg", cssClass: "inside-paper" },

  { type: "video", video: "https://res.cloudinary.com/dvljcimlz/video/upload/v1780830779/Messenger_creation_7E36B50C-B1CB-4E7C-8E29-D4A342B2EE9E_nj0cpt.mp4", title: "In Motion", cssClass: "inside-paper" },
  
  { type: "content", cssClass: "inside-paper" , title: "To be continued..."},
  { type: "cover", title: "Forever.", cssClass: "hardcover" }
]);

onMounted(async () => {
  await nextTick();

  pageFlipInstance = new PageFlip(bookRef.value, {
    width: 400,
    height: 550,
    size: "stretch",
    minWidth: 300,
    maxWidth: 450,
    minHeight: 400,
    maxHeight: 650,
    maxShadowOpacity: 0.25,
    showCover: true,
    mobileScrollSupport: true
  });

  const pageElements = bookRef.value.querySelectorAll('.book-page');
  pageFlipInstance.loadFromHTML(pageElements);

  pageFlipInstance.on("flip", (e) => {
    currentPage.value = e.data;
    isClosed.value = e.data === 0;
    isAtEnd.value = e.data >= pages.value.length - 2;
  });
});

onBeforeUnmount(() => {
  if (pageFlipInstance) pageFlipInstance.destroy();
});

const goNext = () => { if (pageFlipInstance) pageFlipInstance.flipNext(); };
const goPrev = () => { if (pageFlipInstance) pageFlipInstance.flipPrev(); };
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Lora:ital,wght@0,400;0,600;1,400&family=Quicksand:wght@400;500;600&display=swap');

.aqua-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  min-height: 100vh;
  background-color: #F0FAFB;
  font-family: 'Quicksand', sans-serif;
  padding: 40px 20px;
  position: relative;
  overflow: hidden; /* Keeps bubbles inside */
}

/* --- BACKGROUND BUBBLE ANIMATIONS --- */
.bubbles-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
  pointer-events: none; /* Allows clicks to pass through to the book */
}

.bubble {
  position: absolute;
  bottom: -100px;
  background: radial-gradient(circle, rgba(0, 168, 204, 0.15) 0%, rgba(0, 168, 204, 0) 70%);
  border-radius: 50%;
  animation: floatUp linear infinite;
}

/* Randomize bubble sizes, positions, and speeds */
.bubble:nth-child(1) { width: 80px; height: 80px; left: 10%; animation-duration: 12s; animation-delay: 0s; }
.bubble:nth-child(2) { width: 150px; height: 150px; left: 25%; animation-duration: 18s; animation-delay: 2s; }
.bubble:nth-child(3) { width: 60px; height: 60px; left: 40%; animation-duration: 10s; animation-delay: 5s; }
.bubble:nth-child(4) { width: 120px; height: 120px; left: 55%; animation-duration: 15s; animation-delay: 1s; }
.bubble:nth-child(5) { width: 90px; height: 90px; left: 70%; animation-duration: 14s; animation-delay: 4s; }
.bubble:nth-child(6) { width: 180px; height: 180px; left: 85%; animation-duration: 20s; animation-delay: 0s; }
.bubble:nth-child(7) { width: 50px; height: 50px; left: 15%; animation-duration: 9s; animation-delay: 6s; }
.bubble:nth-child(8) { width: 110px; height: 110px; left: 35%; animation-duration: 16s; animation-delay: 3s; }
.bubble:nth-child(9) { width: 70px; height: 70px; left: 65%; animation-duration: 11s; animation-delay: 2s; }
.bubble:nth-child(10){ width: 140px; height: 140px; left: 80%; animation-duration: 19s; animation-delay: 5s; }
.bubble:nth-child(11){ width: 65px; height: 65px; left: 50%; animation-duration: 13s; animation-delay: 7s; }
.bubble:nth-child(12){ width: 100px; height: 100px; left: 5%; animation-duration: 17s; animation-delay: 1s; }

@keyframes floatUp {
  0% { transform: translateY(0) scale(0.8); opacity: 0; }
  20% { opacity: 1; }
  80% { opacity: 1; }
  100% { transform: translateY(-120vh) scale(1.2); opacity: 0; }
}

/* --- BOOK LEVITATION ANIMATION --- */
.book-levitate-wrapper {
  z-index: 10;
  animation: levitateBook 6s ease-in-out infinite;
  margin-bottom: 40px;
  width: 100%;
  max-width: 900px;
  display: flex;
  justify-content: center;
}

@keyframes levitateBook {
  0% { transform: translateY(0px); filter: drop-shadow(0 15px 25px rgba(0, 168, 204, 0.15)); }
  50% { transform: translateY(-12px); filter: drop-shadow(0 25px 35px rgba(0, 168, 204, 0.25)); }
  100% { transform: translateY(0px); filter: drop-shadow(0 15px 25px rgba(0, 168, 204, 0.15)); }
}

.book-wrapper {
  perspective: 1500px;
  width: 100%;
}

.header-prompt {
  margin-bottom: 20px;
  color: #00A8CC;
  font-style: italic;
  font-family: 'Lora', serif;
  font-size: 1.3rem;
  z-index: 10;
  transition: opacity 0.5s ease, transform 0.5s ease;
}

.header-prompt.fade-out {
  opacity: 0;
  transform: translateY(-10px);
  pointer-events: none;
}

.book-page {
  background-color: white;
  box-shadow: inset 0 0 10px rgba(0, 0, 0, 0.02);
}

.hardcover {
  background: linear-gradient(135deg, #00A8CC 0%, #007EA7 100%);
}

.cover-layout {
  padding: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100%;
  box-sizing: border-box;
}

.cover-border {
  border: 1px solid #E0FBFC;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 20px;
  text-align: center;
}

.title-text {
  font-family: 'Lora', serif;
  color: white;
  font-size: 2.8rem;
  margin-bottom: 10px;
}

.subtitle-text {
  color: #E0FBFC;
  font-size: 0.95rem;
  letter-spacing: 3px;
  text-transform: uppercase;
  font-weight: 500;
}

.inside-paper { background-color: #ffffff; }
.blank-layout { background-color: #EBF8FA; }

.content-layout {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 25px;
  box-sizing: border-box;
}

.media-frame {
  flex: 1;
  min-height: 0;
  margin-bottom: 15px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.media-frame img, .book-video {
  width: 100%;
  height: 100%;
  object-fit: contain;
  border-radius: 6px;
}

.book-video {
  background-color: #f7f9fa;
  border: 1px solid #E0FBFC;
}

.text-frame { text-align: center; }

.content-title {
  font-family: 'Lora', serif;
  color: #007EA7;
  font-size: 1.8rem;
  margin: 0 0 5px 0;
}

.content-text {
  color: #555;
  font-size: 0.95rem;
  line-height: 1.4;
  margin: 0;
  font-weight: 500;
}

.elegant-controls {
  display: flex;
  gap: 20px;
  height: 45px;
  align-items: center;
  z-index: 10;
}

.action-btn {
  font-family: 'Quicksand', sans-serif;
  font-size: 0.95rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 1.5px;
  cursor: pointer;
  border: none;
  border-radius: 30px;
  transition: all 0.3s ease;
}

.action-btn:disabled { opacity: 0.3; cursor: not-allowed; }

.primary-btn {
  background-color: #00A8CC;
  color: white;
  padding: 12px 30px;
  box-shadow: 0 4px 15px rgba(0, 168, 204, 0.25);
}

.primary-btn:hover {
  background-color: #007EA7;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 168, 204, 0.35);
}

.text-btn {
  background-color: transparent;
  color: #00A8CC;
  padding: 10px 15px;
}

.text-btn:hover:not(:disabled) { color: #007EA7; }
</style>