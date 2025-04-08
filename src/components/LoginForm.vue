<template>
  <div class="login-form">
    <h2>Login</h2>
    <form @submit.prevent="handleLogin">
      <div>
        <label for="username">Username:</label>
        <input type="text" id="username" v-model="username" required />
      </div>
      <div>
        <label for="password">Password:</label>
        <input type="password" id="password" v-model="password" required />
      </div>
      <button type="submit">Login</button>
    </form>
    <!-- Error message display -->
    <p v-if="errorMessage" class="error-message">{{ errorMessage }}</p>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const username = ref('');
const password = ref('');
const errorMessage = ref(''); // Reactive variable for error message

const handleLogin = async () => {
  errorMessage.value = ''; // Clear any previous error message
  try {
    const response = await fetch(`${import.meta.env.VITE_API_BASE_URL}/auth/token`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/x-www-form-urlencoded',
      },
      body: new URLSearchParams({
        username: username.value,
        password: password.value,
      }),
    });

    if (!response.ok) {
      const errorResponse = await response.json(); // Parse JSON response
      throw new Error(errorResponse.detail || 'An error occurred'); // Extract "detail" or use a fallback message
    }

    const data = await response.json();
    const jwt = data.access_token; // Assuming the token is in the 'access_token' field
    localStorage.setItem('jwt', jwt);
    window.location.reload();
  } catch (error) {
    console.error('There was a problem with the login request:', error);
    errorMessage.value = error.message; // Set the extracted error message
  }
};
</script>

<style scoped>
.login-form {
  max-width: 300px;
  margin: 100px auto;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 10px;
  background-color: #f9f9f9;
}

.login-form h2 {
  text-align: center;
}

.login-form div {
  margin-bottom: 15px;
}

.login-form label {
  display: block;
  margin-bottom: 5px;
}

.login-form input {
  width: 100%;
  padding: 8px;
  box-sizing: border-box;
}

.login-form button {
  width: 100%;
  padding: 10px;
  background-color: #017b1d;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.login-form button:hover {
  opacity: 0.8;
}

.error-message {
  color: red;
  margin-top: 10px;
  text-align: center;
}
</style>