<template>
  <div class="modal-overlay">
    <div class="add-user-form">
      <button class="close-button" @click="$emit('close')">&times;</button>
      <h2>Add User</h2>
      <form @submit.prevent="handleSubmit">
        <div>
          <label for="username">Username:</label>
          <input type="text" id="username" v-model="username" required />
        </div>
        <div>
          <label for="password">Password:</label>
          <input type="password" id="password" v-model="password" required />
        </div>
        <div>
          <label for="email">Email:</label>
          <input type="email" id="email" v-model="email" required />
        </div>
        <div>
          <label for="firstName">First Name:</label>
          <input type="text" id="firstName" v-model="firstName" required />
        </div>
        <div>
          <label for="lastName">Last Name:</label>
          <input type="text" id="lastName" v-model="lastName" required />
        </div>
        <div>
          <label for="role">Role:</label>
          <select id="role" v-model="role" required>
            <option value="ticket_manager">Ticket Manager</option>
            <option value="ticket_updater">Ticket Updater</option>
            <option value="user_admin">User Admin</option>
          </select>
        </div>
        <button class="add-button" type="submit">Add User</button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const username = ref('');
const email = ref('');
const firstName = ref('');
const lastName = ref('');
const role = ref('admin'); // Default role
const password = ref('');
const emit = defineEmits(['close']);

const handleSubmit = async () => {
  const jwt = localStorage.getItem('jwt');
  if (!jwt) {
    alert('You are not authenticated');
    return;
  }

  const newUser = {
    username: username.value,
    email: email.value,
    first_name: firstName.value,
    last_name: lastName.value,
    role: role.value, // Include role in the newUser object
    password: password.value, // Include password in the newUser object
  };

  try {
    const response = await fetch('http://127.0.0.1:8000/users/', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${jwt}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(newUser),
    });

    if (response.ok) {
      alert('User added successfully');
      // Clear the form
      username.value = '';
      email.value = '';
      firstName.value = '';
      lastName.value = '';
      role.value = ''; // Reset to default role
      password.value = '';
      emit('close');
    } else {
      const errorData = await response.json();
      console.error(errorData);
      alert(`Error: ${errorData.detail}`);
    }
  } catch (error) {
    console.error('Error adding user:', error);
    alert('An error occurred while adding the user');
  }
};
</script>

<style scoped>
.modal-overlay {
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

.add-user-form {
  position: relative;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f9f9f9;
  max-width: 400px;
  width: 100%;
  margin: 20px auto;
}

.close-button {
  position: absolute;
  top: 10px;
  right: 10px;
  background: none;
  border: none;
  font-size: 24px;
  cursor: pointer;
  background-color: rgb(231, 2, 2);
  color: white;
  border-radius: 10%;
  padding: 2px 6px;
}

.add-user-form h2 {
  margin-bottom: 20px;
}

.add-user-form div {
  margin-bottom: 10px;
}

.add-user-form label {
  display: block;
  margin-bottom: 5px;
}

.add-user-form input,
.add-user-form select {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
}

.add-button {
  padding: 10px 20px;
  background-color: #017b1d;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  width: 100%;
}

.add-user-form button:hover {
  opacity: 0.8;
}
</style>