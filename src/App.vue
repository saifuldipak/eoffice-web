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
        <button class="btn-add" @click="showAddUserForm = true">
          <i class="fas fa-plus"></i> Add
        </button>
        <button class="btn-logout" @click="handleLogout">
          <i class="fas fa-sign-out-alt"></i> Logout
        </button>
      </div>
    </div>
    <AddUserForm v-if="showAddUserForm" @close="showAddUserForm = false" />
    <ModifyUserForm v-if="showModifyUserForm" :user="selectedUser" @close="showModifyUserForm = false" @update="handleUserUpdate" />
    <div class="search-results" v-if="searchResults.length > 0">
      <table>
        <thead>
          <tr>
            <th>Username</th>
            <th>Email</th>
            <th>Full Name</th>
            <th>Role</th>
            <th>Active</th>
            <th>Created At</th>
            <th>Updated At</th>
            <th>Action</th> <!-- Added Action column -->
          </tr>
        </thead>
        <tbody>
          <tr v-for="user in searchResults" :key="user.id">
            <td>{{ user.username }}</td>
            <td>{{ user.email }}</td>
            <td>{{ user.first_name }} {{ user.last_name }}</td>
            <td>{{ user.role }}</td>
            <td>{{ user.is_active ? 'Yes' : 'No' }}</td>
            <td>{{ new Date(user.created_at).toLocaleString() }}</td>
            <td>{{ new Date(user.updated_at).toLocaleString() }}</td>
            <td>
              <i class="fas fa-edit" @click="editUser(user)"></i>
              <i class="fas fa-trash" @click="deleteUser(user)"></i>
            </td> <!-- Added icons for edit and delete -->
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
import AddUserForm from './components/AddUserForm.vue';
import ModifyUserForm from './components/ModifyUserForm.vue';

const isAuthenticated = ref(false);
const searchQuery = ref('');
const searchResults = ref([]);
const noResults = ref(false);
const showAddUserForm = ref(false);
const showModifyUserForm = ref(false);
const selectedUser = ref(null);

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
    searchResults.value = [];
    noResults.value = false;
    return;
  }

  const jwt = localStorage.getItem('jwt');
  if (!jwt) {
    handleLogout();
    return;
  }

  try {
    const response = await fetch(`${import.meta.env.VITE_API_BASE_URL}/users/${searchQuery.value}`, {
      headers: {
        'Authorization': `Bearer ${jwt}`,
        'Content-Type': 'application/json'
      }
    });
    if (response.status === 401) {
      handleLogout();
      return;
    }
    if (response.status === 404) {
      searchResults.value = [];
      noResults.value = true;
    } else {
      const data = await response.json();
      searchResults.value = data; // Assuming the API returns an array of users
      noResults.value = data.length === 0;
    }
  } catch (error) {
    console.error('Error fetching search results:', error);
    searchResults.value = [];
    noResults.value = true;
  }
};

const editUser = (user) => {
  selectedUser.value = user;
  showModifyUserForm.value = true;
};

const handleUserUpdate = (updatedUser) => {
  const index = searchResults.value.findIndex(u => u.username === updatedUser.username);
  if (index !== -1) {
    searchResults.value[index] = updatedUser;
  }
};

const deleteUser = async (user) => {
  const confirmed = confirm(`Are you sure you want to delete user ${user.username}?`);
  if (!confirmed) {
    return;
  }

  const jwt = localStorage.getItem('jwt');
  if (!jwt) {
    handleLogout();
    return;
  }

  try {
    const response = await fetch(`http://127.0.0.1:8000/users/${user.username}`, {
      method: 'DELETE',
      headers: {
        'Authorization': `Bearer ${jwt}`,
        'Content-Type': 'application/json'
      }
    });
    if (response.status === 401) {
      handleLogout();
      return;
    }
    if (response.ok) {
      searchResults.value = searchResults.value.filter(u => u.username !== user.username);
    } else {
      console.error('Failed to delete user:', response.statusText);
    }
  } catch (error) {
    console.error('Error deleting user:', error);
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

td i {
  cursor: pointer;
  margin-right: 10px;
}

td i:hover {
  color: #007bff;
}

.no-results {
  margin-top: 60px; /* Adjust based on your layout */
  padding: 20px;
  text-align: center;
  color: red;
}
</style>
