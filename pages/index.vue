<template>
  <v-app>
    <v-main class="bg-grey-lighten-4">
      <v-container class="py-8">

        <!-- Login Section -->
        <v-card v-if="currentView === 'login'" class="mx-auto pa-4 mb-6" max-width="500">
          <v-card-title class="text-h5 text-center">Iniciar Sesión</v-card-title>

          <v-alert v-if="loginMessage" type="error" class="mb-4">{{ loginMessage }}</v-alert>

          <v-select
            v-model="loginData.rol"
            :items="['DOCENTE', 'ESTUDIANTE', 'ADMINISTRADOR', 'INVITADO']"
            label="Tipo de Usuario"
            class="mb-3"
          ></v-select>

          <v-text-field v-model="loginData.correo" label="Correo" class="mb-3"></v-text-field>
          <v-text-field v-model="loginData.contraseña" label="Contraseña" type="password" class="mb-3"></v-text-field>

          <v-btn color="primary" @click="login">Ingresar</v-btn>

          <v-divider class="my-4"></v-divider>

          <v-card-subtitle class="text-center">
            ¿No tienes una cuenta? <a href="#" @click.prevent="currentView = 'register'">Regístrate aquí</a>
          </v-card-subtitle>
        </v-card>

        <!-- Registro Section -->
        <v-card v-if="currentView === 'register'" class="mx-auto pa-4 mb-6" max-width="500">
          <v-card-title class="text-h5 text-center">Registrarse</v-card-title>

          <v-alert v-if="registerMessage" type="info" class="mb-4">{{ registerMessage }}</v-alert>

          <v-text-field v-model="registerData.nombre_usuario" label="Nombre de Usuario" class="mb-3"></v-text-field>
          <v-text-field v-model="registerData.correo" label="Correo" class="mb-3"></v-text-field>
          <v-text-field v-model="registerData.contraseña" label="Contraseña" type="password" class="mb-3"></v-text-field>

          <v-select
            v-model="registerData.rol"
            :items="['DOCENTE', 'ESTUDIANTE', 'ADMINISTRADOR', 'INVITADO']"
            label="Tipo de Usuario"
            class="mb-3"
          ></v-select>

          <v-btn color="secondary" @click="registerUser">Registrarse</v-btn>

          <v-card-subtitle class="text-center mt-3">
            <a href="#" @click.prevent="currentView = 'login'">Volver al Login</a>
          </v-card-subtitle>
        </v-card>

        <!-- Dashboard para DOCENTE -->
        <div v-if="currentView === 'dashboard' && userRole === 'DOCENTE'">
          <v-card class="pa-4 mb-6">
            <v-card-title>Subir REDA - DOCENTE</v-card-title>
            <v-text-field v-model="reda.title" label="Título del archivo" class="mb-3"></v-text-field>
            <v-textarea v-model="reda.description" label="Descripción" class="mb-3"></v-textarea>
            <v-text-field v-model="reda.author" label="Autor" class="mb-3"></v-text-field>
            <v-file-input
              v-model="reda.file"
              label="Seleccionar archivo"
              class="mb-3"
              accept="*/*"
              show-size
              prepend-icon="mdi-upload"
            ></v-file-input>
            <v-btn color="primary" @click="uploadReda">Subir REDA</v-btn>
          </v-card>
        </div>

        <!-- Mensaje para otros roles -->
        <div v-if="currentView === 'dashboard' && userRole !== 'DOCENTE'">
          <v-card class="pa-4">
            <v-card-title>Acceso restringido según el rol: {{ userRole }}</v-card-title>
          </v-card>
        </div>

      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const userRole = ref('');
const registerMessage = ref('');
const loginMessage = ref('');
const currentView = ref('login');

const loginData = ref({
  correo: '',
  contraseña: '',
  rol: 'INVITADO'
});

const registerData = ref({
  nombre_usuario: '',
  correo: '',
  contraseña: '',
  rol: 'INVITADO'
});

const reda = ref({
  title: '',
  description: '',
  author: '',
  file: null
});

const login = async () => {
  loginMessage.value = '';
  try {
    const res = await axios.post('http://localhost:3002/auth/login', loginData.value);
    userRole.value = res.data.rol;
    currentView.value = 'dashboard';
    loginMessage.value = 'Login exitoso. Bienvenido, ' + res.data.nombre_usuario;
  } catch (err) {
    loginMessage.value = err.response?.data.message || 'Error al iniciar sesión';
  }
};

const registerUser = async () => {
  registerMessage.value = '';
  try {
    await axios.post('http://localhost:3002/auth/register', registerData.value);
    registerMessage.value = 'Registro exitoso. Ahora puedes iniciar sesión.';

    // ✅ Limpiar formulario y redirigir a login
    registerData.value = { nombre_usuario: '', correo: '', contraseña: '', rol: 'INVITADO' };
    setTimeout(() => {
      registerMessage.value = '';
      currentView.value = 'login';
    }, 1500);
  } catch (err) {
    registerMessage.value = err.response?.data.message || 'Error al registrarse';
  }
};

const uploadReda = async () => {
  if (!reda.value.file) {
    console.log('Selecciona un archivo para subir.');
    return;
  }

  const formData = new FormData();
  formData.append('file', reda.value.file);
  formData.append('title', reda.value.title);
  formData.append('description', reda.value.description);
  formData.append('author', reda.value.author);

  try {
    const res = await axios.post('http://localhost:3002/ovas', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    });
    console.log('Archivo subido:', res.data);

    // Limpiar formulario
    reda.value = { title: '', description: '', author: '', file: null };
  } catch (err) {
    console.error('Error al subir el REDA:', err);
  }
};
</script>

<style scoped>
.v-card {
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
</style>
