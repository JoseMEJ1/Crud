<script setup>
const props = defineProps({
  encabezados: Array,
  contenido: Array,
  tabla: Array
});
</script>

<template>
  <div class="card">
  <div class="card-header" v-for="(contenido, i) in tabla" :key="i">
    Tabla de {{ contenido }}
  </div>
  <div class="card-body">
    <table class="table table-Dark">
      <thead>
        <tr>
          <th v-for="(encabezado, i) in encabezados" :key="i">{{ encabezado }}</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(fila, index) in contenido" :key="index">
          <td v-for="(valor, i) in Object.values(fila)" :key="i">{{ valor }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</div>
</template>
