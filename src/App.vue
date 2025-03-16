<template>
  
    <LoginForm v-if="!isAuthenticated" />
    <div v-else>
      <div class="toolbar">
        <div class="toolbar-left">
          <span>Users</span>
        </div>
        <div class="toolbar-center">
          <input type="text" placeholder="Search..." class="search-input" v-model="searchQuery" @keyup.enter="handleSearch" />
        </div>
        <div class="toolbar-right">
          <button class="btn-add">
            <i class="fas fa-plus"></i> Add
          </button>
          <button class="btn-modify">
            <i class="fas fa-edit"></i> Modify
          </button>
          <button class="btn-delete">
            <i class="fas fa-trash"></i> Delete
          </button>
          <button class="btn-logout" @click="handleLogout">
            <i class="fas fa-sign-out-alt"></i> Logout
          </button>
        </div>
      </div>
      <div class="search-results" v-if="searchResult">
        <table>
          <thead>
            <tr>
              <th>Username</th>
              <th>Email</th>
              <th>Full Name</th>
            </tr>
          </thead>
          <tbody>
            <tr>
              <td>{{ searchResult.username }}</td>
              <td>{{ searchResult.email }}</td>
              <td>{{ searchResult.first_name }} {{ searchResult.last_name }}</td>
            </tr>
          </tbody>
        </table>
      </div>
      <div class="no-results" v-else-if="noResults">
        No results found.
      </div>
    </div>

</template>

<script setup>
import { ref, onMounted } from 'vue';
import LoginForm from './components/LoginForm.vue';

const isAuthenticated = ref(false);
const searchQuery = ref('');
const searchResult = ref();
const noResults = ref(false);

onMounted(() => {
  const jwt = localStorage.getItem('jwt');
  isAuthenticated.value = !!jwt;
});

const handleLogout = () => {
  localStorage.removeItem('jwt');
  isAuthenticated.value = false;
  window.location.reload();
};

const handleSearch = async () => {
  if (searchQuery.value.trim() === '') {
    searchResult.value = '';
    noResults.value = false;
    return;
  }

  const jwt = localStorage.getItem('jwt');
  if (!jwt) {
    handleLogout();
    return;
  }

  try {
    const response = await fetch(`http://127.0.0.1:8000/users/${searchQuery.value}`, {
      headers: {
        'Authorization': `Bearer ${jwt}`,
        'Content-Type': 'application/json'
      }
    });
    if (response.status === 404) {
      searchResults.value = '';
      noResults.value = true;
    } else {
      const data = await response.json();
      searchResult.value = data; // Assuming the API returns an array of results
      noResults.value = false;
    }
  } catch (error) {
    console.error('Error fetching search results:', error);
    searchResult.value = '';
    noResults.value = true;
  }
};
</script>

<style scoped>
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  border: 1px solid #ccc;
  background-color: #d4edda; /* Updated background color */
  position: fixed;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 98%;
  margin-top: 10px;
  border-radius: 10px;
}

.toolbar-left {
  flex: 1;
}

.toolbar-left span {
  color: black;
  font-size: 30px;
}

.toolbar-center {
  flex: 2;
  display: flex;
  justify-content: center;
}

.search-input {
  width: 80%;
  max-width: 500px;
  padding: 10px;
  height: 15px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.toolbar-right {
  display: flex;
  gap: 10px;
}

button {
  display: flex;
  align-items: center;
  padding: 10px 20px;
  border: none;
  color: white;
  cursor: pointer;
  border-radius: 5px;
}

button i {
  margin-right: 8px;
}

button:hover {
  opacity: 0.8;
}

.btn-add {
  background-color: #017b1d;
}

.btn-modify {
  background-color: #cf9c04;
}

.btn-delete {
  background-color: #dc3545;
}

.btn-logout {
  background-color: #007bff;
}

.search-results {
  margin-top: 60px; /* Adjust based on your layout */
  padding: 20px;
}

table {
  width: 100%;
  border-collapse: collapse;
}

th, td {
  padding: 10px;
  border: 1px solid #ccc;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

.no-results {
  margin-top: 60px; /* Adjust based on your layout */
  padding: 20px;
  text-align: center;
  color: red;
}
</style>
