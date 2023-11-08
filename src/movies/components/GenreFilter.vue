<template>
  <div
    class="p-4 bg-gray-100 dark:bg-gray-900 text-gray-800 dark:text-gray-300 rounded dark:shadow-lg shadow-lg dark:overflow-hidden overflow-hidden"
  >
    <!-- Display  name filter-->
    <div class="mb-4">
      <label for="name" class="block text-gray-700 dark:text-gray-300 text-sm font-bold mb-2"
        >Name:</label
      >
      <input
        :disabled="hasError"
        id="name"
        v-model.trim="nameFilter"
        type="text"
        placeholder="Enter movie name"
        class="shadow appearance-none disabled:cursor-not-allowed border rounded w-full py-2 px-3 text-gray-700 dark:text-gray-300 leading-tight focus:outline-none focus:shadow-outline dark:focus:shadow-outline"
      />
    </div>

    <!-- Display description filter -->
    <div class="mb-4">
      <label for="description" class="block text-gray-700 dark:text-gray-300 text-sm font-bold mb-2"
        >Description:</label
      >
      <input
        :disabled="hasError"
        id="description"
        v-model.trim="descriptionFilter"
        type="text"
        placeholder="Enter movie description"
        class="shadow disabled:cursor-not-allowed appearance-none border rounded w-full py-2 px-3 text-gray-700 dark:text-gray-300 leading-tight focus:outline-none focus:shadow-outline dark:focus:shadow-outline"
      />
    </div>
    <!-- Display genre filter -->
    <div class="mb-4">
      <label for="genre" class="block text-gray-700 dark:text-gray-300 text-sm font-bold mb-2"
        >Genre:</label
      >
      <input
        id="genre"
        :disabled="hasError"
        v-model.trim="selectedGenre"
        @change="addGenreFilter"
        class="disabled:cursor-not-allowed shadow appearance-none border rounded w-full py-2 px-3 text-gray-700 dark:text-gray-300 leading-tight focus:outline-none focus:shadow-outline dark:focus:shadow-outline"
        list="genre-list"
        placeholder="Enter movie genre"
      />
      <datalist id="genre-list" class="w-full">
        <!-- Define la lista de géneros -->
        <option v-for="genre in genresApiData" :key="genre.id" :value="genre.name">
          {{ genre.name }}
        </option>
      </datalist>
      <span v-if="!isValidGenre" class="text-red-600 text-sm"
        >"{{ selectedGenre }}" is not a valid genre</span
      >
    </div>

    <!-- Display selected genres as labels -->
    <div class="mb-2">
      <span
        v-for="genre in genresFilter"
        :key="genre"
        class="inline-block bg-gray-300 text-gray-800 text-xs font-semibold rounded-full px-2 py-1 mr-1"
      >
        {{ genre }}
        <button @click="removeGenreFilter(genre)" class="ml-1 text-gray-600 text-xs">×</button>
      </span>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import type { MovieFilter, MovieGenre } from '@/movies/types'
import { useGetGenresApi } from '../composables/useGetGenresApi'

const emit = defineEmits<{
  (event: 'updateFilters', filters: MovieFilter): void
}>()

// Data from API
const genresApiData = ref<MovieGenre[]>([])

// Selected genre
const selectedGenre = ref<string>('')

// Constant to validate genre
//TODO: Validate genre
const isValidGenre = ref(true)

// Filters to send to the MovieList component
const genresFilter = ref<string[]>([])
const nameFilter = ref<string>('')
const descriptionFilter = ref<string>('')

watch([selectedGenre, nameFilter, descriptionFilter], () => {
  const newFilters: MovieFilter = {
    genre: genresFilter.value,
    name: nameFilter.value,
    description: descriptionFilter.value
  }
  emit('updateFilters', newFilters)
})

function addGenreFilter() {
  if (selectedGenre.value !== '' && !genresFilter.value.includes(selectedGenre.value)) {
    genresFilter.value.push(selectedGenre.value)
  }
  selectedGenre.value = ''
}

function removeGenreFilter(genre: string) {
  const index = genresFilter.value.indexOf(genre)
  if (index !== -1) {
    genresFilter.value.splice(index, 1)
  }
}

const hasError = ref<boolean>(true)

async function loadGenres() {
  hasError.value = true
  try {
    const data = await useGetGenresApi()
    genresApiData.value = data
    hasError.value = false
  } catch (error: any) {
    hasError.value = true
  }
}

onMounted(async () => {
  await loadGenres()
})
</script>

<style scoped></style>