<template>
  <div>
    <h1>Usuarios</h1>
    <button @click="openModal">Crear Usuario</button>
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>Nombres</th>
          <th>Apellidos</th>
          <th>Correo Electrónico</th>
          <th>Rol</th>
          <th>Fecha de Registro</th>
          <th>Acciones</th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="users.length === 0">
          <td colspan="7">No hay usuarios disponibles.</td>
        </tr>
        <tr v-for="user in users" :key="user.id">
          <td>{{ user.id }}</td>
          <td>{{ user.name }}</td>
          <td>{{ user.lastname }}</td>
          <td>{{ user.email }}</td>
          <td>{{ user.role }}</td>
          <td>{{ formatDate(user.created_at) }}</td>
          <td>
            <button @click="editUser(user.id)">Editar</button>
            <button @click="deleteUser(user.id)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Modal para crear usuario -->
    <div v-if="isModalOpen" class="modal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <h2>Crear Usuario</h2>
        <form @submit.prevent="createUser">
          <div>
            <label for="name">Nombre:</label>
            <input type="text" v-model="newUser.name" required />
          </div>
          <div>
            <label for="lastname">Apellido:</label>
            <input type="text" v-model="newUser.lastname" required />
          </div>
          <div>
            <label for="email">Correo Electrónico:</label>
            <input type="email" v-model="newUser.email" required />
          </div>
          <div>
            <label for="role">Rol:</label>
            <select v-model="newUser.role" required>
              <option value="" disabled selected>Selecciona un rol</option>
              <option value="Admin">Admin</option>
              <option value="Revisor">Revisor</option>
            </select>
          </div>
          <div>
            <label for="password">Contraseña:</label>
            <input type="text" v-model="newUser.password" required />
          </div>
          <button type="submit" :disabled="isLoading">Guardar</button>
          <button type="button" @click="closeModal">Cancelar</button>
        </form>
        <div v-if="isLoading" class="spinner">
          <span>Cargando...</span> <!-- Aquí puedes agregar un spinner gráfico si deseas -->
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import axios from 'axios'
import dayjs from 'dayjs'
import { useHead } from '@vueuse/head'

const users = ref([])
const isModalOpen = ref(false)
const newUser = ref({
  name: '',
  lastname: '',
  email: '',
  role: '',
  password: ''
})
const isLoading = ref(false) // Estado para el spinner

useHead({
  title: 'Usuarios'
})

// Función para obtener los usuarios
const fetchUsers = async () => {
  try {
    const response = await axios.get('http://127.0.0.1:8000/api/users')
    users.value = response.data 
  } catch (error) {
    console.error('Error al obtener los usuarios:', error)
  }
}

const formatDate = (dateString: string) => {
  return dayjs(dateString).format('DD-MM-YYYY') 
}

// Funciones para abrir y cerrar el modal
const openModal = () => {
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
  resetNewUser() // Resetea el formulario
}

// Función para crear un nuevo usuario
const createUser = async () => {
  isLoading.value = true // Activa el spinner
  try {
    const response = await axios.post('http://127.0.0.1:8000/api/users', newUser.value)
    users.value.push(response.data) // Agrega el nuevo usuario a la lista
    closeModal() // Cierra el modal después de guardar
    location.reload(); // Recarga la página para ver el nuevo usuario
  } catch (error) {
    console.error('Error al crear el usuario:', error)
  } finally {
    isLoading.value = false; // Desactiva el spinner
  }
}

// Función para reiniciar el objeto newUser
const resetNewUser = () => {
  newUser.value = {
    name: '',
    lastname: '',
    email: '',
    role: '',
    password: ''
  }
}

// Llama a la función cuando el componente se monte
onMounted(fetchUsers)

const editUser = (id: number) => {
  console.log('Editar usuario con ID:', id)
}

const deleteUser = (id: number) => {
  console.log('Eliminar usuario con ID:', id)
}
</script>

<style>
  /* Estilos básicos para la tabla */
  table {
    width: 100%;
    border-collapse: collapse;
  }

  th, td {
    border: 1px solid #ccc;
    padding: 8px;
    text-align: left;
  }

  th {
    background-color: #f4f4f4;
  }

  /* Estilos para el modal */
  .modal {
    display: flex;
    justify-content: center;
    align-items: center;
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5);
    z-index: 1000;
  }

  .modal-content {
    background-color: #fff;
    padding: 20px;
    border-radius: 5px;
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
  }

  .close {
    color: #aaa;
    float: right;
    font-size: 28px;
    font-weight: bold;
  }

  .close:hover,
  .close:focus {
    color: black;
    text-decoration: none;
    cursor: pointer;
  }

  /* Estilo para el spinner */
  .spinner {
    display: flex;
    justify-content: center;
    align-items: center;
    margin-top: 10px;
  }
</style>
