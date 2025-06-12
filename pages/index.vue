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

          <!-- Mostrar REDAs subidos y opción para editar -->
          <v-card class="pa-4 mt-6">
            <v-card-title>Editar REDAs</v-card-title>
            <v-row dense>
              <v-col cols="12" md="4" v-for="(reda, index) in redaList" :key="index">
                <v-card class="mb-4">
                  <v-card-title>{{ reda.nombre }}</v-card-title>
                  <v-card-subtitle>{{ reda.descripcion }}</v-card-subtitle>
                  <v-card-text>
                    <strong>Autor:</strong> {{ reda.id_autor }}<br />
                    <strong>Formato:</strong> {{ reda.formato }}
                  </v-card-text>

                  <!-- Botón para editar REDA -->
                  <v-btn color="primary" @click="editReda(reda)">Editar</v-btn>
                </v-card>
              </v-col>
            </v-row>
          </v-card>

          <!-- Formulario para editar el REDA -->
          <v-card v-if="editingReda" class="pa-4 mb-6">
            <v-card-title>Actualizar REDA</v-card-title>
            <v-text-field v-model="editingReda.title" label="Título del archivo" class="mb-3"></v-text-field>
            <v-textarea v-model="editingReda.description" label="Descripción" class="mb-3"></v-textarea>
            <v-text-field v-model="editingReda.author" label="Autor" class="mb-3"></v-text-field>
            <v-btn color="primary" @click="updateReda">Actualizar REDA</v-btn>
          </v-card>
        </div>

        <!-- Dashboard para ESTUDIANTE -->
        <div v-if="currentView === 'dashboard' && userRole === 'ESTUDIANTE'">
          <v-card class="pa-4 mb-6">
            <v-card-title>Archivos Subidos por Docentes</v-card-title>

            <v-alert v-if="redaMessage" type="info" class="mb-4">{{ redaMessage }}</v-alert>

            <v-row dense>
              <v-col cols="12" md="4" v-for="(reda, index) in redaList" :key="index">
                <v-card class="mb-4 pa-4">
                  <v-card-title>{{ reda.nombre }}</v-card-title>
                  <v-card-subtitle>{{ reda.descripcion }}</v-card-subtitle>
                  <v-card-text>
                    <strong>Autor:</strong> {{ reda.id_autor }}<br />
                    <strong>Formato:</strong> {{ reda.formato }}
                  </v-card-text>

                  <!-- Mostrar el archivo según el tipo MIME -->
                  <div v-if="reda.formato && reda.formato.startsWith('image/')">
                    <img :src="getFileUrl(reda.ruta)" alt="Imagen" style="max-width: 100%; height: auto;" />
                  </div>
                  <div v-else-if="reda.formato && reda.formato.startsWith('video/')">
                    <video :src="getFileUrl(reda.ruta)" controls style="max-width: 100%; height: auto;"></video>
                  </div>
                  <div v-else-if="reda.formato === 'application/pdf'">
                    <iframe :src="getFileUrl(reda.ruta)" style="width: 100%; height: 500px;" frameborder="0"></iframe>
                  </div>
                  <div v-else>
                    <!-- Para otros tipos, mostrar link para descargar o abrir -->
                    <a :href="getFileUrl(reda.ruta)" target="_blank" download>Descargar archivo</a>
                  </div>
                </v-card>
              </v-col>
            </v-row>

          </v-card>
        </div>

      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';

const currentView = ref('login');
const userRole = ref('');
const registerMessage = ref('');
const loginMessage = ref('');
const redaMessage = ref('');
const redaList = ref([]);
const editingReda = ref(null); // Nueva referencia para editar el REDA

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

    if (userRole.value === 'ESTUDIANTE') {
      fetchRedas();
    }

  } catch (err) {
    loginMessage.value = err.response?.data.message || 'Error al iniciar sesión';
  }
};

const registerUser = async () => {
  registerMessage.value = '';
  try {
    await axios.post('http://localhost:3002/auth/register', registerData.value);
    registerMessage.value = 'Registro exitoso. Ahora puedes iniciar sesión.';
    registerData.value = { nombre_usuario: '', correo: '', contraseña: '', rol: 'INVITADO' };

    setTimeout(() => {
      currentView.value = 'login';
      registerMessage.value = '';
    }, 1500);

  } catch (err) {
    registerMessage.value = err.response?.data.message || 'Error al registrarse';
  }
};

const uploadReda = async () => {
  if (!reda.value.file) return;

  const formData = new FormData();
  formData.append('file', reda.value.file);
  formData.append('title', reda.value.title);
  formData.append('description', reda.value.description);
  formData.append('author', reda.value.author);

  try {
    await axios.post('http://localhost:3002/ovas', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    });
    reda.value = { title: '', description: '', author: '', file: null };
    fetchRedas(); // Recargar la lista de REDAs después de subir un nuevo REDA
  } catch (err) {
    console.error('Error al subir el REDA:', err);
  }
};

const fetchRedas = async () => {
  redaMessage.value = '';
  try {
    const res = await axios.get('http://localhost:3002/ovas');
    redaList.value = res.data;
  } catch (err) {
    redaMessage.value = 'Error al obtener los REDAs.';
  }
};

const getFileUrl = (ruta) => {
  if (!ruta) return '';
  return ruta.startsWith('http') ? ruta : `http://localhost:3002/uploads/${ruta}`;
};

const editReda = (redaToEdit) => {
  // Setea los valores del REDA a editar
  editingReda.value = { ...redaToEdit };
};

const updateReda = async () => {
  if (!editingReda.value) return;

  try {
    await axios.put(`http://localhost:3002/ovas/${editingReda.value._id}`, editingReda.value);
    fetchRedas();  // Recargar la lista después de la actualización
    editingReda.value = null; // Limpiar el formulario de edición
  } catch (err) {
    console.error('Error al actualizar el REDA:', err);
  }
};
</script>

<style scoped>
.v-card {
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
</style>
