<template>
  <div>
    <h1>Usuarios</h1>
    <button class="btn create" @click="openModal">Crear Usuario</button>
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
            <button class="btn edit" @click="editUser(user.id)">Editar</button>
            <button class="btn delete" @click="deleteUser(user.id)">Eliminar</button>
          </td>
        </tr>
      </tbody>
    </table>

    <!-- Modal para crear/editar usuario -->
    <div v-if="isModalOpen" class="modal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <h2>{{ newUser.id ? 'Editar Usuario' : 'Crear Usuario' }}</h2>
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
          <button class="btn submit" type="submit" :disabled="isLoading">Guardar</button>
          <button class="btn cancel" type="button" @click="closeModal">Cancelar</button>
        </form>
        <div v-if="isLoading" class="spinner">
          <span>Cargando...</span>
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
  id: null, 
  name: '',
  lastname: '',
  email: '',
  role: '',
  password: ''
})
const isLoading = ref(false)

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
  resetNewUser();
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
}

// Función para crear o actualizar un usuario
const createUser = async () => {
  isLoading.value = true
  try {
    let response;
    if (newUser.value.id) {
      response = await axios.put(`http://127.0.0.1:8000/api/users/${newUser.value.id}`, newUser.value);
      const index = users.value.findIndex(user => user.id === newUser.value.id);
      if (index !== -1) {
        users.value[index] = response.data;
      }
    } else {
      response = await axios.post('http://127.0.0.1:8000/api/users', newUser.value);
      users.value.push(response.data);
    }
    
    closeModal();
    location.reload();
  } catch (error) {
    console.error('Error al guardar el usuario:', error);
  } finally {
    isLoading.value = false;
  }
}

const resetNewUser = () => {
  newUser.value = {
    id: null,
    name: '',
    lastname: '',
    email: '',
    role: '',
    password: ''
  }
}

onMounted(fetchUsers)

const editUser = (id: number) => {
  const userToEdit = users.value.find(user => user.id === id);
  if (userToEdit) {
    newUser.value = { ...userToEdit }; 
    isModalOpen.value = true; 
  }
}

const deleteUser = async (id: number) => {
  const confirmDelete = confirm('¿Estás seguro de que deseas eliminar este usuario?');
  if (!confirmDelete) return; 

  try {
    await axios.delete(`http://127.0.0.1:8000/api/users/${id}`);
    location.reload();
  } catch (error) {
    console.error('Error al eliminar el usuario:', error);
  }
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

/* Estilos para los botones */
.btn {
  padding: 10px 15px;
  margin: 5px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s, transform 0.3s;
}

.btn:hover {
  transform: scale(1.05);
}

.btn.create {
  background-color: #0969DA;
  color: white;
}

.btn.edit {
  background-color: #007bff;
  color: white;
}

.btn.edit:hover {
  background-color: #0056b3;
}

.btn.delete {
  background-color: #dc3545;
  color: white;
}

.btn.delete:hover {
  background-color: #c82333;
}

.btn.submit {
  background-color: #28a745;
  color: white;
}

.btn.submit:hover {
  background-color: #218838;
}

.btn.cancel {
  background-color: #6c757d;
  color: white;
}

.btn.cancel:hover {
  background-color: #5a6268;
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
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.2);
  width: 400px; /* Ancho fijo para el modal */
  position: relative; /* Para el posicionamiento del botón de cierre */
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
