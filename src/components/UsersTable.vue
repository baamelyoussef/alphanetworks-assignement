<template>
  <div class="container">
    <div class="table-container">
      <table class="custom-table">
        <thead>
          <tr>
            <th>
              <div class="header-cell">
                <template v-if="!nameSearchActive">
                  <span>Name</span>
                  <i @click="toggleSearch('name')" class="magnify-icon">
                    <svg
                      class="w-6 h-6 text-gray-800 dark:text-white"
                      aria-hidden="true"
                      xmlns="http://www.w3.org/2000/svg"
                      width="24"
                      height="24"
                      fill="none"
                      viewBox="0 0 24 24"
                    >
                      <path
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-width="2"
                        d="m21 21-3.5-3.5M17 10a7 7 0 1 1-14 0 7 7 0 0 1 14 0Z"
                      />
                    </svg>
                  </i>
                </template>
                <template v-else>
                  <input v-model="nameQuery" placeholder="Search by name" />
                  <i @click="toggleSearch('name')" class="close-icon">
                    <svg
                      class="w-6 h-6 text-gray-800 dark:text-white"
                      aria-hidden="true"
                      xmlns="http://www.w3.org/2000/svg"
                      width="24"
                      height="24"
                      fill="none"
                      viewBox="0 0 24 24"
                    >
                      <path
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M6 18 17.94 6M18 18 6.06 6"
                      />
                    </svg>
                  </i>
                </template>
              </div>
            </th>
            <th>
              <div class="header-cell">
                <template v-if="!usernameSearchActive">
                  <span>Username</span>
                  <i @click="toggleSearch('username')" class="magnify-icon">
                    <svg
                      class="w-6 h-6 text-gray-800 dark:text-white"
                      aria-hidden="true"
                      xmlns="http://www.w3.org/2000/svg"
                      width="24"
                      height="24"
                      fill="none"
                      viewBox="0 0 24 24"
                    >
                      <path
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-width="2"
                        d="m21 21-3.5-3.5M17 10a7 7 0 1 1-14 0 7 7 0 0 1 14 0Z"
                      />
                    </svg>
                  </i>
                </template>
                <template v-else>
                  <input v-model="usernameQuery" placeholder="Search by username" />
                  <i @click="toggleSearch('username')" class="close-icon">
                    <svg
                      class="w-6 h-6 text-gray-800 dark:text-white"
                      aria-hidden="true"
                      xmlns="http://www.w3.org/2000/svg"
                      width="24"
                      height="24"
                      fill="none"
                      viewBox="0 0 24 24"
                    >
                      <path
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M6 18 17.94 6M18 18 6.06 6"
                      />
                    </svg>
                  </i>
                </template>
              </div>
            </th>
            <th>
              <div class="header-cell">
                <template v-if="!emailSearchActive">
                  <span>Email</span>
                  <i @click="toggleSearch('email')" class="magnify-icon">
                    <svg
                      class="w-6 h-6 text-gray-800 dark:text-white"
                      aria-hidden="true"
                      xmlns="http://www.w3.org/2000/svg"
                      width="24"
                      height="24"
                      fill="none"
                      viewBox="0 0 24 24"
                    >
                      <path
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-width="2"
                        d="m21 21-3.5-3.5M17 10a7 7 0 1 1-14 0 7 7 0 0 1 14 0Z"
                      />
                    </svg>
                  </i>
                </template>
                <template v-else>
                  <input v-model="emailQuery" placeholder="Search by email" />
                  <i @click="toggleSearch('email')" class="close-icon">
                    <svg
                      class="w-6 h-6 text-gray-800 dark:text-white"
                      aria-hidden="true"
                      xmlns="http://www.w3.org/2000/svg"
                      width="24"
                      height="24"
                      fill="none"
                      viewBox="0 0 24 24"
                    >
                      <path
                        stroke="currentColor"
                        stroke-linecap="round"
                        stroke-linejoin="round"
                        stroke-width="2"
                        d="M6 18 17.94 6M18 18 6.06 6"
                      />
                    </svg>
                  </i>
                </template>
              </div>
            </th>
            <th>Company</th>
            <th>City</th>
            <th>Phone</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="user in filteredUsers"
            :key="user.id"
            @click="viewPosts(user.id)"
            class="clickable-row"
          >
            <td>{{ user.name }}</td>
            <td>{{ user.username }}</td>
            <td>{{ user.email }}</td>
            <td>{{ user.company.name }}</td>
            <td>{{ user.address.city }}</td>
            <td>{{ user.phone }}</td>
          </tr>
        </tbody>
      </table>
    </div>

    <div v-if="loading" class="loading">Loading...</div>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

interface User {
  id: number
  name: string
  username: string
  email: string
  company: {
    name: string
  }
  address: {
    city: string
  }
  phone: string
}

const router = useRouter()

const nameSearchActive = ref<boolean>(false)
const usernameSearchActive = ref<boolean>(false)
const emailSearchActive = ref<boolean>(false)

const toggleSearch = (column: 'name' | 'username' | 'email') => {
  if (column === 'name') nameSearchActive.value = !nameSearchActive.value
  if (column === 'username') usernameSearchActive.value = !usernameSearchActive.value
  if (column === 'email') emailSearchActive.value = !emailSearchActive.value
}

const users = ref<User[]>([])
const loading = ref<boolean>(true)
const error = ref<string | null>(null)

const nameQuery = ref<string>('')
const usernameQuery = ref<string>('')
const emailQuery = ref<string>('')

const filteredUsers = computed<User[]>(() => {
  return users.value.filter((user) => {
    return (
      user.name.toLowerCase().includes(nameQuery.value.toLowerCase()) &&
      user.username.toLowerCase().includes(usernameQuery.value.toLowerCase()) &&
      user.email.toLowerCase().includes(emailQuery.value.toLowerCase())
    )
  })
})

const fetchUsers = async () => {
  try {
    const response = await axios.get<User[]>(import.meta.env.VITE_API_URL + 'users')
    users.value = response.data
  } catch (err) {
    error.value = err instanceof Error ? err.message : 'An unexpected error occurred'
  } finally {
    loading.value = false
  }
}

const viewPosts = (userId: number) => {
  router.push(`/posts?userId=${userId}`)
}

onMounted(fetchUsers)
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
}

.search-container {
  display: flex;
  gap: 20px;
  margin-bottom: 20px;
}

.search-item {
  flex: 1;
}

.search-item input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.table-container {
  overflow-x: auto;
}

.custom-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
  overflow: hidden;
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

.custom-table th,
.custom-table td {
  border: 1px solid #ccc;
}

.custom-table td button {
  padding: 4px 8px;
  background-color: #407df6;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.header-cell {
  display: flex;
  align-items: center;
  justify-content: space-between;
  align-items: center;
  align-content: center;
  gap: 8px;
}

.magnify-icon,
.close-icon {
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

.pagination-btn {
  padding: 8px 12px;
  background-color: #407df6;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
}

.pagination-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}
.loading,
.error {
  text-align: center;
  margin-top: 20px;
}
.custom-table tr.clickable-row {
  cursor: pointer;
}

.custom-table tr.clickable-row:hover {
  background-color: #f1f1f1;
}
</style>
