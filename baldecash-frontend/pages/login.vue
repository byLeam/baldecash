<template>
    <div class="login-container">
      <h2>Iniciar Sesión</h2>
      <form @submit.prevent="loginUser">
        <div>
          <label for="email">Correo Electrónico:</label>
          <input type="email" v-model="credentials.email" required />
        </div>
        <div>
          <label for="password">Contraseña:</label>
          <input type="password" v-model="credentials.password" required />
        </div>
        <button type="submit" :disabled="isLoading">Iniciar Sesión</button>
        <div v-if="isLoading" class="spinner">Cargando...</div>
        <div v-if="errorMessage" class="error">{{ errorMessage }}</div>
      </form>
    </div>
  </template>
  
  <script setup lang="ts">
  import { ref } from 'vue'
  import axios from 'axios'
  import { useRouter } from 'vue-router'
  
  const router = useRouter()
  const credentials = ref({
    email: '',
    password: ''
  })
  const isLoading = ref(false)
  const errorMessage = ref('')
  
  // Función para iniciar sesión
  const loginUser = async () => {
    isLoading.value = true
    errorMessage.value = ''
  
    try {
      const response = await axios.post('http://127.0.0.1:8000/api/login', credentials.value)
      // Suponiendo que la respuesta incluye un token
      localStorage.setItem('token', response.data.token) // Guarda el token en localStorage
      // Redirige al usuario a la página principal después del inicio de sesión
      router.push('/')
    } catch (error) {
      errorMessage.value = 'Credenciales incorrectas. Intenta nuevamente.'
      console.error('Error de inicio de sesión:', error)
    } finally {
      isLoading.value = false
    }
  }
  </script>
  
  <style>
  .login-container {
    max-width: 400px;
    margin: 0 auto;
    padding: 20px;
    border: 1px solid #ccc;
    border-radius: 5px;
    background-color: #fff;
  }
  
  .error {
    color: red;
  }
  </style>
  