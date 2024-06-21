<template>
  <div class="login-form">
    {{ token}}
    <h2>Login</h2>
    <form @submit.prevent="handleSubmit">
      <div class="form-group">
        <label for="username">Usuario:</label>
        <input type="text" id="username" v-model="email" required />
      </div>
      <div class="form-group">
        <label for="password">Contraseña:</label>
        <input type="password" id="password" v-model="password" required />
      </div>
      <button type="submit">Iniciar Sesión</button>
    </form>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      email: '',
      password: '',
      token: '' // Añadir propiedad para el token
    };
  },
  methods: {
    async handleSubmit() {
      try {
        const response = await axios.post('http://192.168.1.133:8081/api/login', {
          email: this.email,
          password: this.password
        });
        
        if (response.data.success) {
          const token = response.data.token;
          // Guarda el token en el local storage
          localStorage.setItem('authToken', token);

          // Actualiza la propiedad del token
          this.token = token;
          this.$emit('refreshData')

          console.log('Login exitoso:', response.data);
        } else {
          console.error('Error en el login:', response.data.message);
        }
      } catch (error) {
        console.error('Error durante el login:', error);
      }
    },
    getTokenFromLocalStorage() {
      this.token = localStorage.getItem('authToken');
    }
  },
  mounted() {
    // Obtener el token cuando el componente se monte
    this.getTokenFromLocalStorage();
  }
};
</script>

<style scoped>
.login-form {
  max-width: 300px;
  margin: 0 auto;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.form-group {
  margin-bottom: 1rem;
}

label {
  display: block;
  margin-bottom: 0.5rem;
}

input {
  width: 100%;
  padding: 0.5rem;
  margin-top: 0.2rem;
  border: 1px solid #ccc;
  border-radius: 5px;
}

button {
  width: 100%;
  padding: 0.5rem;
  background-color: #007BFF;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}
</style>
