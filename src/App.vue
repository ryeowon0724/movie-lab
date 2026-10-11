<script setup>
import { ref } from 'vue'
import { fetchMovies } from './api.js'
import MovieCard from './components/MovieCard.vue'

const draftKey = ref('')
const connected = ref(false)
let apiKey = ''

// 새롭게 추가할 상태 변수들 (영화 목록, 검색어, 에러 메시지)
const movies = ref([])
const searchQuery = ref('')
const errorMsg = ref('')

// API를 호출하여 영화 데이터를 가져오는 함수
async function loadMovies(query = '') {
  errorMsg.value = ''
  try {
    const data = await fetchMovies({ apiKey, query })
    movies.value = data.results
  } catch (error) {
    errorMsg.value = error.message
    movies.value = []
  }
}

// 1. API 키를 입력하고 'Store in memory' 버튼을 눌렀을 때 실행
async function connect() {
  apiKey = draftKey.value.trim()
  if (!apiKey) return
  
  connected.value = true // 연결 상태로 변경
  draftKey.value = ''    // 입력칸 초기화
  
  // 키가 연결되자마자 인기 영화 목록을 불러옵니다.
  await loadMovies()
}

// 2. 영화 검색 버튼을 눌렀을 때 실행
async function handleSearch() {
  await loadMovies(searchQuery.value)
}
</script> 

<template>
  <main class="catalog">
    <h1>🎬 Movie Lab</h1>

    <!-- API 키 입력 화면 (API 키가 없을 때만 보임) -->
    <form v-if="!connected" class="starter-form" @submit.prevent="connect">
      <label for="tmdb-key">TMDB API Key (v3)</label>
      <input id="tmdb-key" type="password" v-model="draftKey" autocomplete="off" required>
      <button>Store in memory</button>
    </form>

    <!-- 영화 목록 및 검색 화면 (API 키 입력 완료 시 보임) -->
    <div v-else>
      <form class="search-form" @submit.prevent="handleSearch">
        <input 
          type="text" 
          v-model="searchQuery" 
          placeholder="영화 제목을 검색해보세요..." 
          class="search-input"
        >
        <button type="submit" class="search-btn">Search</button>
      </form>

      <p v-if="errorMsg" class="error-msg">{{ errorMsg }}</p>

      <div class="movie-grid">
        <!-- 기존에 완성된 MovieCard 컴포넌트를 반복해서 출력 -->
        <MovieCard 
          v-for="movie in movies" 
          :key="movie.id" 
          :movie="movie" 
        />
      </div>
    </div>
  </main>
</template>

<style scoped>
/* 화면을 깔끔하게 보여주기 위한 기본 CSS */
.search-form {
  display: flex;
  justify-content: center;
  margin-bottom: 2rem;
  gap: 10px;
}
.search-input {
  padding: 8px 12px;
  width: 100%;
  max-width: 400px;
  border: 1px solid #ccc;
  border-radius: 4px;
}
.search-btn {
  padding: 8px 16px;
  background-color: #2c3e50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
.error-msg {
  color: red;
  text-align: center;
  font-weight: bold;
}
.movie-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
  gap: 20px;
  padding: 20px 0;
}
</style>
