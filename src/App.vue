

<template>
  <div class="min-h-screen bg-gray-100">
    <div class="container py-6">
      <div class="bg-white p-4 shadow rounded">
        <div class="flex -mx-4 item-center mb-5 shadow-lg py-5">
          <div class="flex-1 px-4">
            <select @change="getSpecificSurah" name="" id="" class="quran-input">
              <option value="">Select Surah</option>
              <option v-for="surah in surahs" :value="surah.number">{{surah.name}} - {{ surah.englishName }}</option>
            </select>
          </div>

          <div class="flex-1 px-4">
            <h3 class="font-bold text-lg mb-1">{{currentSurah.name}} - {{ currentSurah.englishName }}</h3>
            <p>{{currentSurah.englishNameTranslation}}</p>
          </div>
        </div>

        <div v-if="loading" class="flex">
          <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="w-6 h-6 animate-spin">
            <path stroke-linecap="round" stroke-linejoin="round" d="M4.5 12a7.5 7.5 0 0015 0m-15 0a7.5 7.5 0 1115 0m-15 0H3m16.5 0H21m-1.5 0H12m-8.457 3.077l1.41-.513m14.095-5.13l1.41-.513M5.106 17.785l1.15-.964m11.49-9.642l1.149-.964M7.501 19.795l.75-1.3m7.5-12.99l.75-1.3m-6.063 16.658l.26-1.477m2.605-14.772l.26-1.477m0 17.726l-.26-1.477M10.698 4.614l-.26-1.477M16.5 19.794l-.75-1.299M7.5 4.205L12 12m6.894 5.785l-1.149-.964M6.256 7.178l-1.15-.964m15.352 8.864l-1.41-.513M4.954 9.435l-1.41-.514M12.002 12l-3.75 6.495" />
          </svg>

          Loading...
        </div>

        <div v-else class="text-4xl">
          <p v-for="ayah in currentSurah.ayahs" class="py-10 px-5 bg-gray-300 rounded-lg mb-5 leading-loose	">
            <span class="text-xs w-4 h-4 text-center border rounded-full inline-block">{{ ayah.numberInSurah }}</span> - 
            {{ ayah.text}}
          </p>
        </div>




      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data () {
    return {
      surahs: [],
      currentSurah: [],
      loading: true,
    }
  },

  mounted () {
    axios.get('https://api.alquran.cloud/v1/surah')
      .then(response =>{
        this.surahs = response.data.data
      })

      this.querySpecificsurah(1);
      
  },
  methods: {
    getSpecificSurah(e) {
      this.querySpecificsurah(e.target.value)
    },
    querySpecificsurah(surahNumber) {
      axios.get('https://api.alquran.cloud/v1/surah/' + surahNumber).then(response =>{
        console.log( response.data.data);
        this.currentSurah = response.data.data
        this.loading = false
      })
    }
  }

};

</script>