<script setup>
const props = defineProps({
  encabezados: {
    type: Array,
    required: true,
    validator: value => value.every(item => item.column_name)
  },
  contenido: {
    type: Array,
    default: () => []
  },
  tabla: {
    type: String,
    validator: value => ['producto', 'proveedor'].includes(value)
  }
});

const emit = defineEmits(['mostrar-editar', 'mostrar-eliminar']);

const getRowId = (row) => {
  return props.tabla === 'producto' ? row.id_producto : row.id_proveedor;
};

const getRowName = (row) => {
  return props.tabla === 'producto' ? row.nombre_producto : row.nombre_proveedor;
};
</script>

<template>
  <div class="table-container">
    <div class="table-body">
      <table class="table table-dark table-sm">
        <thead>
          <tr>
            <th v-for="(encabezado, i) in encabezados" :key="i" class="text-center">
              {{ encabezado.column_name }}
            </th>
            <th class="text-center">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(fila) in contenido" :key="getRowId(fila)">
            <td v-for="(encabezado, i) in encabezados" :key="i" class="text-center">
              {{ fila[encabezado.column_name] }}
            </td>
            <td class="text-center">
              <button 
                @click="emit('mostrar-editar', fila)" 
                class="btn btn-warning btn-sm"
                title="Editar"
              >
                <i class="bi bi-pencil"></i>
              </button>
              <button 
                @click="emit('mostrar-eliminar', fila)" 
                class="btn btn-danger btn-sm"
                title="Eliminar"
              >
                <i class="bi bi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.table-container {
  overflow-x: auto;
}

.table-body {
  max-height: 500px;
  overflow-y: auto;
}

.table {
  width: 100%;
  margin-bottom: 1rem;
  color: #fff;
}

.table th, .table td {
  padding: 0.75rem;
  vertical-align: middle;
  border-top: 1px solid #454d55;
}

.table thead th {
  vertical-align: bottom;
  border-bottom: 2px solid #454d55;
}

.table-dark {
  background-color: #343a40;
}

.table-dark th, .table-dark td, .table-dark thead th {
  border-color: #454d55;
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
  line-height: 1.5;
  border-radius: 0.2rem;
  margin: 0 2px;
}
</style>