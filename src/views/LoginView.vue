<template>
  <div class="login container">
    <div class="card">
      <div class="card-body">
        <h2>Login</h2>
        <form v-on:submit.prevent="login">
          <input
            type="text"
            placeholder="Correo electrónico"
            class="form-control my-2"
            v-model="email"
          />
          <input
            type="password"
            placeholder="Contraseña"
            v-model="password"
            class="form-control my-2"
          />
          <button type="submit" class="btn btn-primary">Iniciar sesión</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";
const Toast = Swal.mixin({
  toast: true,
  position: "top-end",
  showConfirmButton: false,
  timer: 3000,
  timerProgressBar: true,
  didOpen: (toast) => {
    toast.onmouseenter = Swal.stopTimer;
    toast.onmouseleave = Swal.resumeTimer;
  },
});
export default {
  name: "LoginView",
  data() {
    return {
      email: "",
      password: "",
    };
  },
  methods: {
    login() {
      /* Loguea al usuario */
      axios
        .post("http://127.0.0.1:8000/api/login", {
          email: this.email,
          password: this.password,
        })
        .then((response) => {
          /* Si todo es correcto, guarda el token en el localstorage y redirecciona al home */
          Toast.fire({
            icon: "success",
            title: "Bienvenido",
          });
          localStorage.setItem('token', response.data.token);
          this.$router.push({name: 'Home'});
        })
        .catch((error) => {
          /* Si hay un error muestra una alerta, dependiendo de si son errores del sevidor o de credenciales */
          if (error.response.data.errors) {
            Toast.fire({
              icon: "error",
              title: "credenciales incorrectas",
            });
          } else {
            Toast.fire({
              icon: "error",
              title: "Error en el servidor",
            });
          }
        });
    },
  },
};
</script>
<style>
body,
.card {
  background: #081b29 !important;
}
body {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 100vh;
}
.card {
  width: 750px;
  height: 250px;
  border: 2px solid #0ef !important;
  color: white !important;
  box-shadow: 0 0 25px #0ef !important;
}
.btn-primary {
  background: transparent !important;
}

.bienvenido {
  text-align: right;
  min-height: 10vh;
}
</style>
