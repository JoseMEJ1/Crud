<script setup>
import { ref } from 'vue';
import axios from 'axios';
import Tabla_Impresion from './components/tabla.vue';

const contenido = ref([]);
const encabezados = ref([]);
const tablaSeleccionada = ref('');
const mensajeError = ref('');
const registroAEditar = ref({});
const mostrarModalEditar = ref(false);
const ErrorEditar = ref(false);
const mostrarModalEliminar = ref(false);
const idAEliminar = ref(null);
const nombreAEliminar = ref('');
const ErrorEliminar = ref(false);
const mostrarModalAgregar = ref(false);
const nuevoRegistro = ref({});
const ErrorAgrega = ref(false);

// Modal
const cerrarModal = () => {
  mostrarModalEliminar.value = false;
  mostrarModalAgregar.value = false;
  mostrarModalEditar.value = false;
};

// Editar
const mostrarEditar = (registro) => {
  registroAEditar.value = { ...registro };
  nuevoRegistro.value = { ...registro };
  mostrarModalEditar.value = true;
  ErrorEditar.value = false;
};

const EditarRegistro = async () => {
  try {
    const idKey = tablaSeleccionada.value === 'producto' ? 'id_producto' : 'id_proveedor';
    const id = registroAEditar.value[idKey];
    const url = `http://localhost:3000/api/editar/${tablaSeleccionada.value}/${id}`;
    const { [idKey]: _, ...datosActualizados } = nuevoRegistro.value;
    await axios.put(url, datosActualizados);
    await mostrarTabla(tablaSeleccionada.value);
    cerrarModal();
  } catch (error) {
    mensajeError.value = `Error al actualizar ${tablaSeleccionada.value}: ${error.response?.data?.message || error.message}`;
    ErrorEditar.value = true;
  }
};

// Eliminar
const mostrarEliminar = (data) => {
  idAEliminar.value = data.id;
  nombreAEliminar.value = data.nombre;
  mostrarModalEliminar.value = true;
  ErrorEliminar.value = false;
};

const confirmarEliminacion = async () => {
  const url = `http://localhost:3000/api/eliminar/${tablaSeleccionada.value}/${idAEliminar.value}`;
  try {
    await axios.delete(url);
    await mostrarTabla(tablaSeleccionada.value);
    cerrarModal();
  } catch (error) {
    mensajeError.value = `Error al eliminar ${tablaSeleccionada.value}: ${error.response?.data?.message || error.message}`;
    ErrorEliminar.value = true;
  }
};

// Agregar
const mostrarModalAgregarRegistro = () => {
  nuevoRegistro.value = {};
  encabezados.value.forEach((header) => {
    nuevoRegistro.value[header.column_name] = '';
  });
  mostrarModalAgregar.value = true;
};

const agregarRegistro = async () => {
  const datos = {};
  encabezados.value.forEach((columna) => {
    const valor = nuevoRegistro.value[columna.column_name];
    if (valor !== undefined && valor !== null && valor !== '') {
      datos[columna.column_name] = valor;
    }
  });
  try {
    const url = `http://localhost:3000/api/agregar/${tablaSeleccionada.value}`;
    await axios.post(url, datos);
    await mostrarTabla(tablaSeleccionada.value);
    mostrarModalAgregar.value = false;
  } catch (error) {
    ErrorAgrega.value = true;
    mensajeError.value = 'Error al agregar el registro.';
  }
};

// Leer
const mostrarTabla = async (tabla) => {
  tablaSeleccionada.value = tabla;
  try {
    const responseContenido = await axios.get(`http://localhost:3000/api/seleccionarcontenido/${tabla}`);
    contenido.value = Array.isArray(responseContenido.data) ? responseContenido.data : [];

    const responseEncabezados = await axios.get(`http://localhost:3000/api/seleccionarencabezado/${tabla}`);
    encabezados.value = Array.isArray(responseEncabezados.data) ? responseEncabezados.data : [];

    nuevoRegistro.value = {};
    encabezados.value.forEach((header) => {
      nuevoRegistro.value[header.nombre] = '';
    });
    if (tabla === 'producto') {
      nuevoRegistro.value['id_proveedor'] = '';
    }
  } catch (error) {
    console.error('Error al obtener los datos:', error);
  }
};

</script>

<template>
  <div class="sidebar">
    <h4 class="Titulo_Panel">Panel Tablas <i class="bi bi-folder"></i></h4>
    <button class="btn btn-secondary" @click="mostrarTabla('producto')">
      <i class="bi bi-box"></i> Productos <i class="bi bi-rocket-takeoff"></i>
    </button>
    <button class="btn btn-secondary" @click="mostrarTabla('proveedor')">
      <i class="bi bi-person-lines-fill"></i> Proveedor <i class="bi bi-truck"></i>
    </button>
  </div>

  <div class="content">
    <nav class="navbar navbar-dark bg-dark mb-4">
      <div class="container-fluid">
        <span class="navbar-brand">Crud Inventario <i class="bi bi-cart-check"></i></span>
      </div>
    </nav>

    <div v-if="!tablaSeleccionada">
      <div class="alert alert-primary" role="alert">
        <i class="bi bi-info-circle"></i> Selecciona una tabla para visualizar su contenido.
      </div>
    </div>

    <div v-else-if="encabezados.length > 0">
      <div class="table-title">
        <div class="row">
          <div class="col-9">
            <h5 style="color: white;">
              <i class="bi bi-bag-check"></i> Tabla de {{ tablaSeleccionada }} <i class="bi bi-fire"></i>
            </h5>
          </div>
          <div class="col-3">
            <button type="button" class="btn btn-success" @click="mostrarModalAgregarRegistro">
              <i class="bi bi-plus-circle"></i> Agregar {{ tablaSeleccionada }} <i class="bi bi-cloud-arrow-up-fill"></i>
            </button>
          </div>
        </div>
        <br>
      </div>

      <Tabla_Impresion 
        :encabezados="encabezados" 
        :contenido="contenido" 
        :tabla="tablaSeleccionada"
        @mostrar-eliminar="mostrarEliminar"
        @mostrar-editar="mostrarEditar"
      />
      <div v-if="contenido.length === 0" class="alert alert-danger" role="alert">
        <i class="bi bi-cloud-slash"></i> La tabla {{ tablaSeleccionada }} se encuentra vacía.
      </div>
    </div>

  </div>

  <!-- Modal Eliminar -->
  <div v-if="mostrarModalEliminar" class="custom-modal-overlay" @click.self="cerrarModal">
    <div class="modal-dialog custom-modal">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Eliminar {{ tablaSeleccionada }}</h5>
          <button type="button" class="btn" @click="cerrarModal"><i class="bi bi-x-circle"></i></button>
        </div>
        <div class="modal-body">
          <p>¿Seguro que deseas eliminar <strong>{{ nombreAEliminar }}</strong>?</p>
          <div v-if="ErrorEliminar">
            <div class="alert alert-danger" role="alert">
              <p>{{ mensajeError }}</p>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn btn-secondary" @click="cerrarModal">Cancelar</button>
          <button class="btn btn-danger" @click="confirmarEliminacion"><i class="bi bi-trash"></i> Eliminar</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal Agregar -->
  <div v-if="mostrarModalAgregar" class="custom-modal-overlay" @click.self="cerrarModal">
    <div class="modal-dialog custom-modal">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Agregar {{ tablaSeleccionada }}</h5>
          <button type="button" class="btn" @click="cerrarModal"><i class="bi bi-x-circle"></i></button>
        </div>
        <div class="modal-body">
          <div v-for="(columna, index) in encabezados" :key="index">
            <label>{{ columna.column_name }}</label>
            <input
              v-model="nuevoRegistro[columna.column_name]"
              :placeholder="columna.column_name"
              :id="columna.column_name"
              :name="columna.column_name"
              class="form-control"
            />
          </div>
          <div v-if="ErrorAgrega">
            <div class="alert alert-warning" role="alert">
              <p id="textoerror">{{ mensajeError }}</p>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn btn-secondary" @click="cerrarModal"><i class="bi bi-x-circle"></i> Cancelar</button>
          <button class="btn btn-primary" @click="agregarRegistro"><i class="bi bi-plus-circle"></i> Agregar</button>
        </div>
      </div>
    </div>
  </div>

  <!-- Modal Editar -->
  <div v-if="mostrarModalEditar" class="custom-modal-overlay" @click.self="cerrarModal">
    <div class="modal-dialog custom-modal">
      <div class="modal-content">
        <div class="modal-header">
          <h5 class="modal-title">Editar {{ tablaSeleccionada }}</h5>
          <button type="button" class="btn" @click="cerrarModal"><i class="bi bi-x-circle"></i></button>
        </div>
        <div class="modal-body">
          <div v-for="(columna, index) in encabezados" :key="index">
            <label>{{ columna.column_name }}</label>
            <input
              v-model="nuevoRegistro[columna.column_name]"
              :placeholder="columna.column_name"
              :id="columna.column_name"
              :name="columna.column_name"
              class="form-control"
            />
          </div>
          <div v-if="ErrorEditar">
            <div class="alert alert-danger" role="alert">
              <p>{{ mensajeError }}</p>
            </div>
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn btn-secondary" @click="cerrarModal">Cancelar</button>
          <button class="btn btn-warning" @click="EditarRegistro"><i class="bi bi-pencil"></i> Editar</button>
        </div>
      </div>
    </div>
  </div>

</template>


<style scoped>
.form-control {
  background: none;
  border: none;
  border-bottom: 2px solid #888;
  color: #fff; 
  padding: 8px;
  font-size: 16px;
  transition: border-color 0.3s ease, box-shadow 0.3s ease;
}

.form-control:focus {
  outline: none;
  border-bottom: 2px solid #00bcd4;
  box-shadow: 0 2px 5px rgba(0, 188, 212, 0.5);
}

.form-control::placeholder {
  color: #bbb;
  opacity: 0.7;
}

.custom-modal-overlay {
  background-color: black;
  color: white;
}

.sidebar {
  width: 250px;
  height: 100vh;
  background: #23272a;
  padding: 15px;
  position: fixed;
  color: white;
}

.sidebar button {
  display: block;
  margin-bottom: 10px;
  width: 100%;
  text-align: left;
  background: none;
  border: none;
  color: #fff;
  padding: 8px;
  font-size: 14px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.sidebar button:hover {
  background-color: #444;
}

.content {
  margin-left: 260px;
  padding: 20px;
  font-size: 14px;
}

.custom-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  z-index: 1050;
  display: flex;
  align-items: center;
  justify-content: center;
}

.custom-modal {
  background: #333;
  border-radius: 8px;
  width: 90%;
  max-width: 500px;
}

.modal-header, .modal-footer {
  padding: 1rem;
  border-bottom: 1px solid #444;
}

.modal-body {
  padding: 1rem;
}

.btn-close {
  font-size: 1.5rem;
  background: none;
  color: white;
  border: none;
  cursor: pointer;
}

.btn-close:hover {
  color: #00bcd4;
}
</style>
