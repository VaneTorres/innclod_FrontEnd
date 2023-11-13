<template>
  <div class="documentos">
    <div class="row">
      <!-- Boton de cerrar sesión -->
      <div class="col-sm-3">
        <button type="button" class="btn btn-danger my-2" @click="logout()">
          Cerrar sesión
        </button>
      </div>
      <!-- Fin boton cerrar sesión -->
      <!-- Boton para agregar un nuevo documento -->
      <div class="col-sm-3">
        <button
          type="button"
          class="btn btn-primary my-2"
          data-toggle="modal"
          data-target="#FormularioDocumento0"
        >
          Nuevo documento
        </button>
      </div>
      <!-- Fin boton agregar -->
      <!-- Input para buscar en la tabla documentos -->
      <div class="col-sm-6">
        <input
          type="text"
          class="form-control"
          @keydown="Buscar()"
          placeholder="Buscar"
        />
      </div>
    </div>
    <!-- Fin de input -->
    <!-- Tabla de documentos -->
    <div class="table-responsive">
      <table class="table table-dark">
        <thead>
          <tr>
            <th scope="col">Código</th>
            <th scope="col">Tipo de documento</th>
            <th scope="col">Proceso de documento</th>
            <th scope="col">Nombre del documento</th>
            <th scope="col">Contenido</th>
            <th scope="col">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="documento in data" :key="documento.id">
            <th>{{ documento.codigo }}</th>
            <td>{{ documento.tip_tipo_doc.tip_nombre }}</td>
            <td>{{ documento.pro_proceso.pro_nombre }}</td>
            <td>{{ documento.doc_nombre }}</td>
            <td>{{ documento.doc_contenido }}</td>

            <td>
              <!-- Boton elliminar -->
              <button
                class="btn btn-danger"
                @click="deleteDocument(documento.id)"
              >
                Eliminar
              </button>
              <!-- Fin boton eliminar -->
              <!-- Boton editar -->
              <button
                type="button"
                class="btn btn-primary mx-2"
                data-toggle="modal"
                :data-target="'#FormularioDocumento' + documento.id"
                @click="id = documento.id"
              >
                Editar
              </button>
              <!-- Fin boton editar -->
              <!-- Llama el formulario de editar -->
              <Formulario
                title="Editar documento"
                v-on:editarTabla="ConsultarDocumentos"
                :id_document="documento.id"
                :procesos="procesos"
                :tipo_documentos="tipo_documentos"
              />
              <!-- Fin formulario editar -->
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <!-- Fin de tabla de documentos -->
  </div>
  <!-- LLama el formulario de agregar -->
  <Formulario
    title="Nuevo documento"
    :id_document="id"
    :procesos="procesos"
    v-on:editarTabla="ConsultarDocumentos"
    :tipo_documentos="tipo_documentos"
  />
</template>
<script>
import Formulario from "@/components/Formulario.vue";
import axios from "axios";
import Swal from "sweetalert2";
// Alertas
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
  name: "HomeView",
  data() {
    return {
      tipo_documentos: [],
      procesos: [],
      data: [],
      id: 0,
      contador: {},
      datos_originales: [],
    };
  },
  components: {
    Formulario,
  },
  methods: {
    /* Función para cerrar sesión */
    logout(){
      axios
        .post("http://127.0.0.1:8000/api/logout")
        .then((response) => {
          Toast.fire({
            icon: "success",
            title: "Sesión cerrada",
          });
          localStorage.removeItem('token');
          this.$router.push({name: 'Login'});
        })
        .catch((e) => {
          Toast.fire({
            icon: "error",
            title: "Error en el servidor",
          });
        });
    },
    /* Función para buscar en la tabla */
    Buscar() {
      this.data = this.datos_originales.filter((item) => {
        return item.doc_nombre
          .toLowerCase()
          .includes(event.target.value.toLowerCase());
      });
    },
    /* Función para obtener los datos parametricos de la base de datos */
    obtenerParametricas() {
      axios
        .get("http://127.0.0.1:8000/api/parametricas", {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          this.tipo_documentos = response.data.tipo_documento;
          this.procesos = response.data.proceso;
        })
        .catch((e) => {
          Toast.fire({
            icon: "error",
            title: "Error en el servidor",
          });
        });
    },
    /* Función para eliminar un documento */
    deleteDocument(id) {
      Swal.fire({
        title: "¿Estas seguro?",
        text: "¡No podrás revertir esto!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonColor: "#3085d6",
        cancelButtonColor: "#d33",
        cancelButtonText: "Cancelar",
        confirmButtonText: "Si, eliminar",
      }).then((result) => {
        if (result.value) {
          axios
            .delete(`http://127.0.0.1:8000/api/documentos/${id}`, {
              headers: {
                Authorization: `Bearer ${localStorage.getItem("token")}`,
              },
            })
            .then((response) => {
              this.ConsultarDocumentos();
              Toast.fire({
                icon: "success",
                title: "Documento eliminado",
              });
            })
            .catch((error) => {
              Toast.fire({
                icon: "error",
                title: "Error en el servidor",
              });
            });
        }
      });
    },
    /* Función para generar el codigo del documento */
    generarCodigo(item) {
      const key = `${item.tip_tipo_doc.tip_prefijo}-${item.pro_proceso.pro_prefijo}`;
      this.contador[key] = (this.contador[key] || 0) + 1;
      return `${key}-${this.contador[key]}`;
    },
    /* Función para consultar los documentos */
    ConsultarDocumentos() {
      this.contador = {};
      axios
        .get("http://127.0.0.1:8000/api/documentos", {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          response.data = response.data.map((item) => ({
            codigo: this.generarCodigo(item),
            ...item,
          }));
          this.datos_originales = response.data;
          this.data = response.data;
        })
        .catch((error) => {
          Toast.fire({
            icon: "error",
            title: "Error en el servidor",
          });
        });
    },
  },
  mounted() {
    this.ConsultarDocumentos();
    this.obtenerParametricas();
  },
};
</script>
