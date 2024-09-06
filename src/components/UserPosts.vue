<template>
  <div class="container">
    <button @click="goBack" class="back-btn">
      <svg
        class="w-6 h-6 text-gray-800 dark:text-white"
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
        width="18"
        height="18"
        fill="none"
        viewBox="0 0 24 24"
      >
        <path
          stroke="currentColor"
          stroke-linecap="round"
          stroke-linejoin="round"
          stroke-width="2"
          d="m15 19-7-7 7-7"
        />
      </svg>
      Back to Users
    </button>
    <div class="divider"></div>
    <div class="card">
      <div class="card-title">Posts by User {{ userId }}</div>
      <div v-if="loading" class="loading">Loading...</div>
      <div v-else-if="error" class="error">{{ error }}</div>

      <table class="custom-table">
        <thead>
          <tr>
            <th>Title</th>
            <th>Body</th>
            <th>Comments Count</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="post in paginatedPosts" :key="post.id">
            <td>{{ post.title }}</td>
            <td>{{ post.body }}</td>
            <td>{{ post.comments.length }}</td>
          </tr>
        </tbody>
      </table>

      <div v-if="!loading && posts.length === 0" class="info">No posts found for this user.</div>

      <div class="pagination">
        <button @click="prevPage" :disabled="currentPage === 1" class="pagination-btn">Prev</button>
        <span>Page {{ currentPage }} of {{ totalPages }}</span>
        <button @click="nextPage" :disabled="currentPage >= totalPages" class="pagination-btn">
          Next
        </button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import axios, { type AxiosResponse } from 'axios'
import { useRoute, useRouter } from 'vue-router'

type Comment = {
  postId: number
  id: number
  name: string
  email: string
  body: string
}

const route = useRoute()
const router = useRouter()
const userId = ref(route.query.userId as string)
const posts = ref<any[]>([])
const loading = ref(true)
const error = ref<string | null>(null)
const currentPage = ref(1)
const postsPerPage = ref(5)

// Fetch posts and comments
const fetchPosts = async () => {
  loading.value = true
  try {
    // Fetch all posts
    const postsResponse = await axios.get(
      import.meta.env.VITE_API_URL + `users/${userId.value}/posts`
    )
    const postsData = postsResponse.data

    // Fetch all comments for the posts in parallel
    const commentsRequests: [AxiosResponse] = postsData.map((post: any) =>
      axios.get(import.meta.env.VITE_API_URL + `posts/${post.id}/comments`)
    )
    const commentsResponses = await axios.all(commentsRequests)

    // Combine posts with their comments
    const postsWithComments = postsData.map((post: any, index: number) => ({
      ...post,
      comments: commentsResponses[index].data
    }))

    posts.value = postsWithComments
  } catch (err) {
    error.value = (err as Error).message
  } finally {
    loading.value = false
  }
}

// Navigate back to users page
const goBack = () => {
  router.push('/')
}

// Pagination logic
const prevPage = () => {
  if (currentPage.value > 1) {
    currentPage.value--
  }
}

const nextPage = () => {
  if (currentPage.value < totalPages.value) {
    currentPage.value++
  }
}

// Total pages based on posts count and posts per page
const totalPages = computed(() => {
  return Math.ceil(posts.value.length / postsPerPage.value)
})

// Paginated posts
const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * postsPerPage.value
  const end = start + postsPerPage.value
  return posts.value.slice(start, end)
})

// Initial data fetch
onMounted(fetchPosts)
</script>

<style scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

.back-btn {
  margin-bottom: 20px;
  padding: 8px 12px;
  background-color: #407df6;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 6;
}

.divider {
  margin: 20px 0;
  border-top: 1px solid #ddd;
}

.card {
  padding: 20px;
  border-radius: 10px;
  background-color: #f9f9f9;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.1);
}

.card-title {
  font-size: 1.5em;
  margin-bottom: 10px;
}

.custom-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.custom-table th,
.custom-table td {
  border: 1px solid #ddd;
  padding: 12px;
  text-align: left;
}

.custom-table th {
  background-color: #f4f4f4;
}

.custom-table tr:nth-child(even) {
  background-color: #f9f9f9;
}

.custom-table td button {
  padding: 4px 8px;
  background-color: #407df6;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}

.pagination button {
  padding: 8px 12px;
  background-color: #407df6;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}

.pagination button:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

.loading,
.error,
.info {
  text-align: center;
  margin-top: 20px;
}
</style>
