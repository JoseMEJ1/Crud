<template>
  <div class="table-container">
    <div class="table-body">
      <table class="table table-dark table-sm">
        <thead class="table table-dark">
          <tr>
            <th v-for="(encabezado, i) in encabezados" :key="i" class="text-center">
              {{ encabezado.column_name }}
            </th>
            <th class="text-center">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(fila, index) in contenido" :key="index">
            <td v-for="(valor, i) in Object.values(fila)" :key="i" class="text-center">
              {{ valor }}
            </td>
            <td class="text-center">
              <button 
                @click="$emit('mostrar-editar', fila)" 
                class="btn btn-warning btn-sm">
                <i class="bi bi-pencil"></i>
              </button>
              <button 
                @click="$emit('mostrar-eliminar', { id: Object.values(fila)[0], nombre: Object.values(fila)[1]})" 
                class="btn btn-danger btn-sm">
                <i class="bi bi-trash"></i>
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
const props = defineProps({
  encabezados: Array,
  contenido: Array,
  tabla: String
});
</script>