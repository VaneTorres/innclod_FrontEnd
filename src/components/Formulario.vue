<template>
  <div class="formulario">
    <div
      class="modal fade "
      :id="'FormularioDocumento' + id_document"
      tabindex="-1"
      role="dialog"
      aria-labelledby="FormularioDocumentoLabel"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-lg" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h3 class="modal-title fs-5">{{ title }}</h3>
            <button
              type="button"
              class="close"
              data-dismiss="modal"
              aria-label="Close"
            >
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <form v-on:submit.prevent="Guardar()">
            <div class="modal-body">
              <div class="alert alert-danger" role="alert" v-if="errors">
                <ul>
                  <li v-if="errors.tip_id">{{ errors.tip_id }}</li>
                  <li v-if="errors.pro_id">{{ errors.pro_id }}</li>
                  <li v-if="errors.doc_nombre">{{ errors.doc_nombre }}</li>
                  <li v-if="errors.doc_contenido">{{ errors.doc_contenido }}</li>
                </ul>
              </div>
              <div class="row g-3">
                <div class="col-md-4">
                  <label>Tipo de documento</label>
                  <select class="form-control" v-model="form.tip_id">
                    <option
                      v-for="tipo_documento in tipo_documentos"
                      :key="tipo_documento.id"
                      :value="tipo_documento.id"
                    >
                      {{
                        tipo_documento.tip_prefijo +
                        "-" +
                        tipo_documento.tip_nombre
                      }}
                    </option>
                  </select>
                </div>
                <div class="col-md-4">
                  <label>Proceso</label>
                  <select class="form-control" v-model="form.pro_id">
                    <option
                      v-for="proceso in procesos"
                      :key="proceso.id"
                      :value="proceso.id"
                    >
                      {{ proceso.pro_prefijo + "-" + proceso.pro_nombre }}
                    </option>
                  </select>
                </div>
                <div class="col-md-4">
                  <label>Nombre</label>
                  <input
                    type="text"
                    class="form-control"
                    v-model="form.doc_nombre"
                  />
                </div>
                <div class="col-md-12">
                  <label>Contenido</label>
                  <textarea
                    class="form-control"
                    v-model="form.doc_contenido"
                  ></textarea>
                </div>
              </div>
            </div>
            <div class="modal-footer">
              <button type="submit" class="btn btn-success">Guardar</button>
            </div>
          </form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import Swal from "sweetalert2";
/* Alertas */
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
  name: "Formulario",
  data() {
    return {
      form: { tip_id: "", pro_id: "", doc_nombre: "", doc_contenido: "" },
      errors: null,
    };
  },
  props: {
    id_document: Number,
    title: String,
    procesos: Array,
    tipo_documentos: Array,
  },
  methods: {
    /* Alerta de error */
    alertError(error) {
      if (error.response.status == 422) {
        this.errors = error.response.data.errors;
        Toast.fire({
          icon: "error",
          title: "Error en el formulario",
        });
      } else {
        Toast.fire({
          icon: "error",
          title: "Error en el servidor",
        });
      }
    },
    /* Alerta de exito */
    alertSuccess() {
      Toast.fire({
        icon: "success",
        title: "Documento guadado correctamente",
      });
      this.$emit("editarTabla");
      $("#FormularioDocumento" + this.id_document).modal("hide");
    },
    /* Función para crear un documento */
    crearDocumento() {
      axios
        .post(`http://127.0.0.1:8000/api/documentos`, this.form, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          this.alertSuccess();
        })
        .catch((error) => {
          this.alertError(error);
        });
    },
    /* Función para editar un documento */  
    editarDocumento(id) {
      axios
        .put(`http://127.0.0.1:8000/api/documentos/${id}`, this.form, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          this.alertSuccess();
        })
        .catch((error) => {
          this.alertError(error);
        });
    },
    /* Función para guardar un documento, valida si tiene id para enviarlo a crear o editar según corresponda*/
    Guardar() {
      if (this.id_document == 0) {
        this.crearDocumento();
      } else {
        this.editarDocumento(this.id_document);
      }
    },
  },
  mounted(){
    /* Si el formulario es utilizado para editar me trae la información del documento */
    if (this.id_document != 0) {
     axios
        .get(`http://127.0.0.1:8000/api/documentos/${this.id_document}`, {
          headers: {
            Authorization: `Bearer ${localStorage.getItem("token")}`,
          },
        })
        .then((response) => {
          this.form = {
            tip_id: response.data.tip_id,
            pro_id: response.data.pro_id,
            doc_nombre: response.data.doc_nombre,
            doc_contenido: response.data.doc_contenido,
          };
        })
        .catch((error) => {
          this.alertError(error);
        });
    }
  }
};
</script>
<style>
.modal {
  color: black !important;
}
</style>
