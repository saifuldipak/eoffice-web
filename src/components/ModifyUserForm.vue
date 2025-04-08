<template>
  <div class="overlay">
    <div class="modify-user-form">
      <h2>Modify User</h2>
      <form @submit.prevent="handleSubmit">
        <div>
          <label for="username">Username:</label>
          <input type="text" id="username" v-model="user.username" disabled />
        </div>
        <div>
          <label for="email">Email:</label>
          <input type="email" id="email" v-model="user.email" required />
        </div>
        <div>
          <label for="first_name">First Name:</label>
          <input type="text" id="first_name" v-model="user.first_name" required />
        </div>
        <div>
          <label for="last_name">Last Name:</label>
          <input type="text" id="last_name" v-model="user.last_name" required />
        </div>
        <div>
          <label for="password">Password:</label>
          <input type="password" id="password" v-model="user.password" />
        </div>
        <div>
          <label for="role">Role:</label>
          <select id="role" v-model="user.role" required>
            <option value="ticket_manager">Ticket Manager</option>
            <option value="ticket_updater">Ticket Updater</option>
            <option value="user_admin">User Admin</option>
          </select>
        </div>
        <div>
          <label for="is_active">Active Status:</label>
          <div class="radio-group">
            <div class="radio-item">
              <input type="radio" id="active" value="true" v-model="user.is_active" />
              <label for="active">Active</label>
            </div>
            <div class="radio-item">
              <input type="radio" id="inactive" value="false" v-model="user.is_active" />
              <label for="inactive">Inactive</label>
            </div>
          </div>
        </div>
        <button type="submit">Save</button>
        <button type="button" @click="$emit('close')">Cancel</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const props = defineProps({
  user: Object
});

const emit = defineEmits(['close', 'update']);

const user = ref({ ...props.user });
const initialUser = JSON.parse(JSON.stringify(user.value)); // Deep copy of the initial user object

const handleSubmit = async () => {
  const jwt = localStorage.getItem('jwt');
  if (!jwt) {
    handleLogout();
    return;
  }

  const modifiedFields = {};
  for (const key in user.value) {
    if (user.value[key] !== initialUser[key]) {
      modifiedFields[key] = user.value[key];
    }
  }

  if (Object.keys(modifiedFields).length === 0) {
    alert('No changes made to the form.');
    return;
  }

  try {
    const response = await fetch(`${import.meta.env.VITE_API_BASE_URL}/users/${user.value.username}`, {
      method: 'PATCH',
      headers: {
        'Authorization': `Bearer ${jwt}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(modifiedFields)
    });
    if (response.status === 401) {
      handleLogout();
      return;
    }
    if (response.ok) {
      alert('User updated successfully');
      emit('update', user.value);
      emit('close');
    } else {
      console.error('Failed to update user:', response.statusText);
    }
  } catch (error) {
    console.error('Error updating user:', error);
  }
};
</script>

<style scoped>
.overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modify-user-form {
  background-color: #f9f9f9;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  max-width: 500px;
  width: 90%;
}

.modify-user-form h2 {
  margin-bottom: 20px;
  text-align: center;
}

.modify-user-form form div {
  margin-bottom: 15px;
}

.modify-user-form label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

.modify-user-form input,
.modify-user-form select {
  width: calc(100% - 20px);
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  box-sizing: border-box;
}

.modify-user-form button {
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-right: 10px;
}

.modify-user-form button[type="submit"] {
  background-color: #28a745;
  color: white;
}

.modify-user-form button[type="button"] {
  background-color: #dc3545;
  color: white;
}

.radio-group {
  display: flex;
  align-items: center;
}

.radio-item {
  display: flex;
  align-items: center;
  margin-right: 15px;
}

.radio-item input[type="radio"] {
  margin-right: 5px;
}
</style>