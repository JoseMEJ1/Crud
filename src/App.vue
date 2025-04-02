<script setup>
import { ref } from 'vue';
import axios from 'axios';
import TablaImpresion from './components/tabla.vue';

const contenido = ref([]);
const encabezados = ref([]);
const tablaSeleccionada = ref('');
const mensajeError = ref("");
const registroAEditar = ref({});
const mostrarModalEditar = ref(false);
const mostrarModalEliminar = ref(false);
const idAEliminar = ref(null);
const nombreAEliminar = ref('');
const mostrarModalAgregar = ref(false);
const nuevoRegistro = ref({});

// Campos requeridos para cada tabla
const camposRequeridos = {
  producto: ['nombre', 'precio', 'existencia', 'id_proveedor'],
  proveedor: ['nombre', 'rfc']
};

const cerrarModal = () => {
  mostrarModalEliminar.value = false;
  mostrarModalAgregar.value = false;
  mostrarModalEditar.value = false;
  mensajeError.value = "";
};

const mostrarEditar = (registro) => {
  registroAEditar.value = { ...registro };
  nuevoRegistro.value = { ...registro };
  mostrarModalEditar.value = true;
};

const validarDatos = (datos) => {
  const errores = [];
  
  // Validar campos requeridos
  camposRequeridos[tablaSeleccionada.value].forEach(campo => {
    if (!datos[campo] && datos[campo] !== 0) {
      errores.push(`El campo ${campo} es requerido`);
    }
  });
  
  // Validaciones específicas para productos
  if (tablaSeleccionada.value === 'producto') {
    if (datos.precio <= 0) {
      errores.push('El precio debe ser mayor que 0');
    }
    if (datos.existencia < 0) {
      errores.push('La existencia no puede ser negativa');
    }
  }
  
  // Validaciones específicas para proveedores
  if (tablaSeleccionada.value === 'proveedor' && datos.rfc && datos.rfc.length !== 13) {
    errores.push('El RFC debe tener 13 caracteres');
  }
  
  return errores;
};

const editarRegistro = async () => {
  try {
    const idKey = tablaSeleccionada.value === "producto" ? "id_producto" : "id_proveedor";
    const id = registroAEditar.value[idKey];
    
    if (!id) throw new Error("ID no válido");
    
    // Validar datos antes de enviar
    const errores = validarDatos(nuevoRegistro.value);
    if (errores.length > 0) {
      throw new Error(errores.join('\n'));
    }
    
    const url = `http://localhost:3000/api/editar/${tablaSeleccionada.value}/${id}`;
    const { [idKey]: _, ...datosActualizados } = nuevoRegistro.value;
    
    await axios.put(url, datosActualizados);
    await mostrarTabla(tablaSeleccionada.value);
    cerrarModal();
  } catch (error) {
    mensajeError.value = error.response?.data?.error || error.message;
  }
};

const mostrarEliminar = (registro) => {
  const idKey = tablaSeleccionada.value === "producto" ? "id_producto" : "id_proveedor";
  const nombreKey = "nombre"; // Corregido: ahora usa simplemente 'nombre'
  
  idAEliminar.value = registro[idKey];
  nombreAEliminar.value = registro[nombreKey] || `Registro ${registro[idKey]}`;
  mostrarModalEliminar.value = true;
};

const confirmarEliminacion = async () => {
  try {
    if (!idAEliminar.value) throw new Error("ID no especificado");
    
    const url = `http://localhost:3000/api/eliminar/${tablaSeleccionada.value}/${idAEliminar.value}`;
    await axios.delete(url);
    await mostrarTabla(tablaSeleccionada.value);
    cerrarModal();
  } catch (error) {
    mensajeError.value = error.response?.data?.error || error.message;
  }
};

const mostrarModalAgregarRegistro = () => {
  nuevoRegistro.value = {};
  encabezados.value.forEach(header => {
    if (!header.column_name.includes('id_') || 
        (tablaSeleccionada.value === 'producto' && header.column_name === 'id_proveedor')) {
      nuevoRegistro.value[header.column_name] = '';
    }
  });
  mostrarModalAgregar.value = true;
};

const agregarRegistro = async () => {
  try {
    const datos = {};
    
    encabezados.value.forEach(columna => {
      const valor = nuevoRegistro.value[columna.column_name];
      if (valor !== undefined && valor !== null && valor !== '') {
        // Convertir a número los campos numéricos
        if (['precio', 'existencia', 'id_proveedor'].includes(columna.column_name)) {
          datos[columna.column_name] = Number(valor);
        } else {
          datos[columna.column_name] = valor;
        }
      }
    });

    // Validar datos antes de enviar
    const errores = validarDatos(datos);
    if (errores.length > 0) {
      throw new Error(errores.join('\n'));
    }
    
    const url = `http://localhost:3000/api/agregar/${tablaSeleccionada.value}`;
    await axios.post(url, datos);
    await mostrarTabla(tablaSeleccionada.value);
    cerrarModal();
  } catch (error) {
    mensajeError.value = error.response?.data?.error || error.message;
  }
};

const mostrarTabla = async (tabla) => {
  try {
    contenido.value = [];
    encabezados.value = [];
    mensajeError.value = "";
    
    const [contenidoRes, encabezadosRes] = await Promise.all([
      axios.get(`http://localhost:3000/api/seleccionarcontenido/${tabla}`),
      axios.get(`http://localhost:3000/api/seleccionarencabezado/${tabla}`)
    ]);
    
    tablaSeleccionada.value = tabla;
    contenido.value = contenidoRes.data || [];
    encabezados.value = encabezadosRes.data || [];
  } catch (error) {
    mensajeError.value = error.response?.data?.error || error.message;
  }
};
</script>

<template>
  <div class="sidebar">
    <h4 class="Titulo_Panel">Panel Tablas <i class="bi bi-folder"></i></h4>
    <button class="btn btn-secondary" @click="mostrarTabla('producto')">
      <i class="bi bi-box"></i> Productos
    </button>
    <button class="btn btn-secondary" @click="mostrarTabla('proveedor')">
      <i class="bi bi-person-lines-fill"></i> Proveedor
    </button>
  </div>

  <div class="content">
    <nav class="navbar navbar-dark bg-dark mb-4">
      <div class="container-fluid">
        <span class="navbar-brand">Crud Inventario <i class="bi bi-cart-check"></i></span>
      </div>
    </nav>

    <div v-if="mensajeError" class="alert alert-danger" role="alert">
      <i class="bi bi-exclamation-triangle"></i> {{ mensajeError }}
    </div>

    <div v-if="!tablaSeleccionada">
      <div class="alert alert-primary" role="alert">
        <i class="bi bi-info-circle"></i> Selecciona una tabla
      </div>
    </div>

    <div v-else-if="encabezados.length > 0">
      <div class="table-title">
        <div class="row">
          <div class="col-9">
            <h5 style="color: white;">
              <i class="bi bi-bag-check"></i> Tabla de {{ tablaSeleccionada }}
            </h5>
          </div>
          <div class="col-3">
            <button type="button" class="btn btn-success" @click="mostrarModalAgregarRegistro">
              <i class="bi bi-plus-circle"></i> Agregar
            </button>
          </div>
        </div>
      </div>

      <TablaImpresion 
        :encabezados="encabezados" 
        :contenido="contenido" 
        :tabla="tablaSeleccionada"
        @mostrar-eliminar="mostrarEliminar"
        @mostrar-editar="mostrarEditar"
      />
      
      <div v-if="contenido.length === 0" class="alert alert-warning" role="alert">
        <i class="bi bi-cloud-slash"></i> La tabla está vacía
      </div>
    </div>

    <div v-if="mostrarModalEliminar" class="custom-modal-overlay" @click.self="cerrarModal">
      <div class="modal-dialog custom-modal">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Eliminar {{ tablaSeleccionada }}</h5>
            <button type="button" class="btn" @click="cerrarModal"><i class="bi bi-x-circle"></i></button>
          </div>
          <div class="modal-body">
            <p>¿Eliminar <strong>{{ nombreAEliminar }}</strong>?</p>
            <div v-if="mensajeError" class="alert alert-danger" role="alert">
              <i class="bi bi-exclamation-triangle"></i> {{ mensajeError }}
            </div>
          </div>
          <div class="modal-footer">
            <button class="btn btn-secondary" @click="cerrarModal">Cancelar</button>
            <button class="btn btn-danger" @click="confirmarEliminacion">
              <i class="bi bi-trash"></i> Eliminar
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="mostrarModalAgregar" class="custom-modal-overlay" @click.self="cerrarModal">
      <div class="modal-dialog custom-modal">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Agregar {{ tablaSeleccionada }}</h5>
            <button type="button" class="btn" @click="cerrarModal"><i class="bi bi-x-circle"></i></button>
          </div>
          <div class="modal-body">
            <div v-for="(columna, index) in encabezados" :key="index">
              <template v-if="!columna.column_name.includes('id_') || 
                            (tablaSeleccionada === 'producto' && columna.column_name === 'id_proveedor')">
                <label>{{ columna.column_name }}</label>
                <input
                  v-model="nuevoRegistro[columna.column_name]"
                  :placeholder="columna.column_name"
                  class="form-control"
                  :required="!columna.column_name.includes('id_') || columna.column_name === 'id_proveedor'"
                />
              </template>
            </div>
            <div v-if="mensajeError" class="alert alert-danger" role="alert">
              <i class="bi bi-exclamation-triangle"></i> {{ mensajeError }}
            </div>
          </div>
          <div class="modal-footer">
            <button class="btn btn-secondary" @click="cerrarModal">
              <i class="bi bi-x-circle"></i> Cancelar
            </button>
            <button class="btn btn-primary" @click="agregarRegistro">
              <i class="bi bi-plus-circle"></i> Agregar
            </button>
          </div>
        </div>
      </div>
    </div>

    <div v-if="mostrarModalEditar" class="custom-modal-overlay" @click.self="cerrarModal">
      <div class="modal-dialog custom-modal">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">Editar {{ tablaSeleccionada }}</h5>
            <button type="button" class="btn" @click="cerrarModal"><i class="bi bi-x-circle"></i></button>
          </div>
          <div class="modal-body">
            <div v-for="(valor, campo) in registroAEditar" :key="campo">
              <template v-if="!campo.includes('id_') || 
                            (tablaSeleccionada === 'producto' && campo === 'id_proveedor')">
                <label>{{ campo }}</label>
                <input
                  v-model="nuevoRegistro[campo]"
                  :placeholder="campo"
                  class="form-control"
                  :required="!campo.includes('id_') || campo === 'id_proveedor'"
                />
              </template>
            </div>
            <div v-if="mensajeError" class="alert alert-danger" role="alert">
              <i class="bi bi-exclamation-triangle"></i> {{ mensajeError }}
            </div>
          </div>
          <div class="modal-footer">
            <button class="btn btn-secondary" @click="cerrarModal">
              <i class="bi bi-x-circle"></i> Cancelar
            </button>
            <button class="btn btn-primary" @click="editarRegistro">
              <i class="bi bi-pencil"></i> Guardar
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
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
  cursor: pointer;
}

.sidebar button:hover {
  background-color: #444;
}

.content {
  margin-left: 260px;
  padding: 20px;
}

.custom-modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
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
  color: white;
}

.form-control {
  background: none;
  border: none;
  border-bottom: 2px solid #888;
  color: #fff;
  padding: 8px;
  margin-bottom: 15px;
}

.form-control:focus {
  outline: none;
  border-bottom: 2px solid #00bcd4;
}

.Titulo_Panel {
  color: #fff;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 1px solid #444;
}

.table-title {
  background: #343a40;
  padding: 15px;
  border-radius: 5px;
  margin-bottom: 20px;
}

.alert {
  margin-top: 15px;
}
</style>