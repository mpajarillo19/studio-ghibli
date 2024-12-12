<script setup>
import { ref, onMounted, computed } from 'vue'

// State Management
const movies = ref([])          // Stores the list of all movies
const loading = ref(true)       // Tracks loading state
const error = ref(null)         // Stores any error messages
const searchQuery = ref('')     // Stores the current search input
const currentPage = ref(1)      // Tracks current page number
const moviesPerPage = 6         // Number of movies to display per page
const sortBy = ref('title')     // Current sorting criteria
const selectedMovie = ref(null) // Currently selected movie for modal
const isModalOpen = ref(false)  // Modal visibility state

// API Integration
const fetchMovies = async () => {
    try {
        const response = await fetch('https://ghibliapi.vercel.app/films')
        movies.value = await response.json()
    } catch (e) {
        error.value = 'Error fetching movies'
    } finally {
        loading.value = false
    }
}

// Modal Control Functions
const openModal = (movie) => {
    selectedMovie.value = movie
    isModalOpen.value = true
}

const closeModal = () => {
    isModalOpen.value = false
    selectedMovie.value = null
}

// Search and Sort Functionality
const filteredMovies = computed(() => {
    // Filter movies based on search query
    let filtered = movies.value.filter(movie =>
        movie.title.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
        movie.original_title.toLowerCase().includes(searchQuery.value.toLowerCase())
    )

    // Sort filtered movies based on selected criteria
    return filtered.sort((a, b) => {
        switch (sortBy.value) {
            case 'title':
                return a.title.localeCompare(b.title)
            case 'title-desc':
                return b.title.localeCompare(a.title)
            case 'date':
                return a.release_date - b.release_date
            case 'date-desc':
                return b.release_date - a.release_date
            case 'director':
                return a.director.localeCompare(b.director)
            case 'director-desc':
                return b.director.localeCompare(a.director)
            default:
                return 0
        }
    })
})

// Pagination Logic
const totalPages = computed(() =>
    Math.ceil(filteredMovies.value.length / moviesPerPage)
)

const paginatedMovies = computed(() => {
    const start = (currentPage.value - 1) * moviesPerPage
    const end = start + moviesPerPage
    return filteredMovies.value.slice(start, end)
})

// Pagination Navigation Functions
const nextPage = () => {
    if (currentPage.value < totalPages.value) {
        currentPage.value++
    }
}

const prevPage = () => {
    if (currentPage.value > 1) {
        currentPage.value--
    }
}

const goToPage = (page) => {
    currentPage.value = page
}

// Lifecycle Hook
onMounted(() => {
    fetchMovies() // Fetch movies when component mounts
})
</script>


<template>
    <div class="movies-container">
        <div class="controls">
            <div class="search-bar">
                <input type="text" v-model="searchQuery" placeholder="Search movies..." @input="currentPage = 1">
            </div>

            <div class="sort-control">
                <select v-model="sortBy">
                    <option value="title">Title (A-Z)</option>
                    <option value="title-desc">Title (Z-A)</option>
                    <option value="date">Release Date (Old-New)</option>
                    <option value="date-desc">Release Date (New-Old)</option>
                    <option value="director">Director (A-Z)</option>
                    <option value="director-desc">Director (Z-A)</option>
                </select>
            </div>
        </div>

        <div v-if="loading">Loading...</div>
        <div v-else-if="error">{{ error }}</div>
        <div v-else>
            <div class="movie-grid">
                <div v-for="movie in paginatedMovies" :key="movie.id" class="movie-card" @click="openModal(movie)">
                    <h2>{{ movie.title }}</h2>
                    <h3>{{ movie.original_title }}</h3>
                    <img :src="movie.image" :alt="movie.title" class="movie-image">
                </div>
            </div>

            <div v-if="isModalOpen" class="modal-overlay" @click="closeModal">
                <div class="modal-content" @click.stop>
                    <button class="modal-close" @click="closeModal">&times;</button>
                    <div v-if="selectedMovie" class="modal-details">
                        <h2>{{ selectedMovie.title }}</h2>
                        <h3>{{ selectedMovie.original_title }}</h3>
                        <img :src="selectedMovie.image" :alt="selectedMovie.title" class="modal-image">
                        <p class="director">Director: {{ selectedMovie.director }}</p>
                        <p class="release-date">Release Date: {{ selectedMovie.release_date }}</p>
                        <p class="description">{{ selectedMovie.description }}</p>
                    </div>
                </div>
            </div>

            <div class="pagination" v-if="totalPages > 1">
                <button @click="prevPage" :disabled="currentPage === 1" class="pagination-button">
                    Previous
                </button>

                <button v-for="page in totalPages" :key="page" @click="goToPage(page)"
                    :class="['page-number', { active: currentPage === page }]">
                    {{ page }}
                </button>

                <button @click="nextPage" :disabled="currentPage === totalPages" class="pagination-button">
                    Next
                </button>
            </div>
        </div>
    </div>
</template>

<style lang="scss" scoped>
.controls {
    display: flex;
    gap: 1rem;
    margin-bottom: 2rem;
}

.search-bar {
    flex: 1;

    input {
        width: 100%;
        padding: 0.8rem;
        font-size: 1rem;
        border: 1px solid #ddd;
        border-radius: 4px;
        box-sizing: border-box;
    }
}

.sort-control {
    select {
        padding: 0.8rem;
        font-size: 1rem;
        border: 1px solid #ddd;
        border-radius: 4px;
        background-color: white;
        cursor: pointer;
        min-width: 200px;

        &:hover {
            border-color: #999;
        }
    }
}

.movie-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
    gap: 2rem;
    margin-bottom: 2rem;
}

.movie-card {
    border: 1px solid #ddd;
    border-radius: 8px;
    padding: 1rem;
    background: white;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    cursor: pointer;
    transition: transform 0.2s;

    &:hover {
        transform: scale(1.02);
    }

    .movie-image {
        width: 100%;
        height: auto;
        border-radius: 4px;
    }

    h2 {
        margin: 0.5rem 0;
        font-size: 1.5rem;
    }

    h3 {
        margin: 0.5rem 0;
        font-size: 1rem;
        color: #666;
    }
}

.modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000;
}

.modal-content {
    background: white;
    padding: 2rem;
    border-radius: 8px;
    max-width: 800px;
    width: 90%;
    max-height: 90vh;
    overflow-y: auto;
    position: relative;
}

.modal-close {
    position: absolute;
    top: 1rem;
    right: 1rem;
    border: none;
    background: none;
    font-size: 1.5rem;
    cursor: pointer;
    padding: 0.5rem;
    width: 2rem;
    height: 2rem;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;

    &:hover {
        background: #f0f0f0;
    }
}

.modal-image {
    width: 100%;
    max-height: 400px;
    object-fit: contain;
    margin: 1rem 0;
}

.modal-details {
    h2 {
        margin-top: 0;
    }

    p {
        margin: 1rem 0;
    }

    .director,
    .release-date {
        font-weight: bold;
    }

    .description {
        line-height: 1.6;
    }
}

.pagination {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 0.5rem;
    margin-top: 2rem;
    padding: 1rem;

    button:hover:not(:disabled) {
        background: #f0f0f0;
    }
}

.pagination-button {
    padding: 0.5rem 1rem;
    border: 1px solid #ddd;
    background: white;
    cursor: pointer;
    border-radius: 4px;

    &:disabled {
        background: #f5f5f5;
        cursor: not-allowed;
    }
}

.page-number {
    padding: 0.5rem 1rem;
    border: 1px solid #ddd;
    background: white;
    cursor: pointer;
    border-radius: 4px;

    &.active {
        background: #4a4a4a;
        color: white;
        border-color: #4a4a4a;
    }
}
</style>
