<template>
  <div class="min-h-screen bg-gray-100" style="background-image: url('tile.jpg');">
    <div class="container py-6 mx-auto">
      <div class="bg-white p-4 shadow rounded">
        <div class="flex flex-col md:flex-row mb-5 shadow-lg py-5">
          <!-- Surah selection dropdown -->
          <div class="flex flex-1 md:w-1/3 items-center px-4 mb-4 md:mb-0">
            <select @change="getSpecificSurah" class="quran-input text-center bg-gray-200 rounded-md py-2 px-4 w-full">
              <option value="">Select Surah</option>
              <option v-for="surah in surahs" :value="surah.number" :key="surah.number">
                {{ surah.number }}. {{ surah.name }} - {{ surah.englishName }}
              </option>
            </select>
          </div>

          <!-- Surah information -->
          <div class="flex-1 md:w-1/3 px-4 py-6">
            <h3 class="font-bold text-lg mb-1 text-gray-800 text-center">
              {{ currentSurah.number }}. {{ currentSurah.name }} - {{ currentSurah.englishName }}
            </h3>
            <p class="text-sm text-gray-600 text-center">
              Total Ayahs: {{ currentSurah.numberOfAyahs }} - {{ currentSurah.englishNameTranslation }}
            </p>
          </div>

          <!-- Loop toggle button -->
          <div class="flex flex-1 md:w-1/3 px-4 items-center">
            <button @click="toggleLoop" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded w-full" :class="{ 'bg-yellow-500': isLooping }">
              {{ isLooping ? 'Disable Loop' : 'Enable Loop' }}
            </button>
          </div>
        </div>

        <!-- Loading spinner -->
        <div v-if="loading" class="flex items-center justify-center">
          <svg class="animate-spin h-5 w-5 mr-3 text-gray-800" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
            <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
            <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 4.418 3.582 8 8 8v-4c-2.657 0-4.92-1.292-6.324-3.209l2.472-1.413zM12 20c-4.418 0-8-3.582-8-8H0c0 6.627 5.373 12 12 12v-4c-2.657 0-4.92-1.292-6.324-3.209l2.472-1.413A7.957 7.957 0 0112 16v4zm6.324-3.209A7.957 7.957 0 0120 12h-4c0 4.418 3.582 8 8 8v-4c-2.657 0-4.92-1.292-6.324-3.209l2.472-1.413zM16 4.709V9.5h4A7.962 7.962 0 0116 4.709z"></path>
          </svg>
          <span class="text-gray-800">Loading...</span>
        </div>

        <!-- Ayahs display -->
        <div v-else>
          <!-- Render Bismillah if not the first surah -->
          <div v-if="currentSurah.number !== 1" class="single-ayah rtl bg-gray-100 rounded-lg mb-5 divide-y divide-gray-300 hidden">
            <div class="py-8 px-10 text-center">
              <p class="arbic-text text-3xl font-bold mb-2 leading-[3rem]">
                بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ
              </p>
            </div>
          </div>
          
          <!-- Ayahs loop -->
          <div class="single-ayah rtl bg-gray-100 rounded-lg mb-5 divide-y divide-gray-300">
            <div v-for="(ayah, index) in currentSurah.ayahs" :key="index" :class="{ 'bg-yellow-200': index === highlightedAyahIndex }">
              <div class="shadow p-4 mb-4">
                <!-- Play button -->
                <button @click="playAudio(currentSurahAudio.ayahs[index].audio, index)" class="bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-2 rounded-full">
                  <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor" class="w-6 h-6">
                    <path stroke-linecap="round" stroke-linejoin="round" d="M21 7.5V18M15 7.5V18M3 16.811V8.69c0-.864.933-1.406 1.683-.977l7.108 4.061a1.125 1.125 0 010 1.954l-7.108 4.061A1.125 1.125 0 013 16.811z"/>
                  </svg>
                </button>
                <div>
                  <!-- Arabic text -->
                  <p ref="ayahText" class="arabic-text text-2xl text-right mb-2" style="line-height: 3.5rem">
                    <span class="text-gray-600 border-gray-900 text-sm mr-2 border rounded-full px-3 py-2">{{ ayah.numberInSurah }}</span>
                    {{ currentSurahArabic.ayahs[index].text }}
                  </p>
                  <!-- Ayah text -->
                  <p class="text-sm sm:text-xl bangla-text-nato text-gray-700 text-left mt-4">{{ ayah.text }}</p>
                  <!-- English translation -->
                  <p class="text-sm sm:text-base text-gray-500 mt-2 text-left">{{ currentSurahTranslations[index] }}</p>
                </div>
              </div>
            </div>
          </div>
          <!-- Default audio player (hidden) -->
          <audio ref="audioPlayer" controls controlsList="nodownload" class="fixed bottom-0 left-1/2 transform -translate-x-1/2 w-80 text-white py-2" v-show="showCustomControls"></audio>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "App",
  data() {
    return {
      surahs: [],
      currentSurah: [],
      currentSurahArabic: [],
      currentSurahAudio: [],
      currentSurahTranslations: [],
      loading: true,
      isLooping: false,
      showCustomControls: false,
      highlightedAyahIndex: -1
    };
  },
  mounted() {
    axios.get("https://api.alquran.cloud/v1/surah").then((response) => {
      this.surahs = response.data.data;
      this.querySpecificsurah(1);
    });
  },

  methods: {
    getSpecificSurah(e) {
      this.querySpecificsurah(e.target.value);
    },

    playAudio(audioUrl, index) {
  const audioPlayer = this.$refs.audioPlayer;

  // Play the audio of the current ayah
  audioPlayer.src = audioUrl;
  audioPlayer.play();
  this.highlightedAyahIndex = index;

  // Show custom controls when audio is played
  this.showCustomControls = true;

  // Scroll to the highlighted ayah
  this.scrollToHighlightedAyah();

  // Listen for the audio to finish playing
  audioPlayer.onended = () => {
    if (this.isLooping) {
      // If looping is enabled, play the next ayah
      const nextIndex = (index + 1) % this.currentSurahAudio.ayahs.length;
      this.playAudio(this.currentSurahAudio.ayahs[nextIndex].audio, nextIndex);
    } else if (index < this.currentSurahAudio.ayahs.length - 1) {
      // If looping is disabled and there are more ayahs in the surah, play the next ayah
      this.playAudio(this.currentSurahAudio.ayahs[index + 1].audio, index + 1);
    } else {
      // If it's the last ayah and looping is disabled, proceed to the next surah
      const nextSurahNumber = this.currentSurah.number + 1;
      if (nextSurahNumber <= this.surahs.length) {
        this.querySpecificsurah(nextSurahNumber);
      }
    }
  };
},



    querySpecificsurah(surahNumber) {
      this.loading = true;
      const banglaPromise = axios.get(`https://api.alquran.cloud/v1/surah/${surahNumber}/bn.bengali`);
      const audioPromise = axios.get(`https://api.alquran.cloud/v1/surah/${surahNumber}/ar.alafasy`);
      const englishTranslationPromise = axios.get(`https://api.alquran.cloud/v1/surah/${surahNumber}/en.asad`);

      Promise.all([banglaPromise, audioPromise, englishTranslationPromise]).then(([banglaResponse, audioResponse, englishTranslationResponse]) => {
        this.currentSurah = banglaResponse.data.data;
        this.currentSurahAudio = audioResponse.data.data;
        this.currentSurahTranslations = englishTranslationResponse.data.data.ayahs.map(ayah => ayah.text);

        axios.get(`https://api.alquran.cloud/v1/surah/${surahNumber}`)
          .then(response => {
            this.currentSurahArabic = response.data.data;
            this.loading = false;
          })
          .catch(error => {
            console.log(error);
            this.loading = false;
          });
      })
      .catch(error => {
        console.log(error);
        this.loading = false;
      });
    },

    toggleLoop() {
      this.isLooping = !this.isLooping;
    },

    scrollToHighlightedAyah() {
      // Scroll to the highlighted ayah
      const ayahElement = this.$refs.ayahText[this.highlightedAyahIndex];
      if (ayahElement) {
        ayahElement.scrollIntoView({ behavior: "smooth", block: "center" });
      }
    }
  }
};
</script>

<style>
.container{
  max-width: 800px;
}
</style>
