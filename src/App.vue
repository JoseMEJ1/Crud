<script setup>
import { ref } from 'vue';
import Tabla from './components/tabla.vue';
import Formulario from './components/formulario.vue';

const encabezados3 = ['id', 'nombre', 'edad', 'pais', 'oficio'];
const encabezados2 = ['id', 'nombre', 'edad', 'pais', 'oficio'];
const formularios = ['nombre', 'edad', 'pais', 'oficio'];

const contenido1 = ref([]);
const contenido2 = ref([]);

fetch('http://localhost:3000/api/selectodo/personajes')
  .then(response => response.json())
  .then(data => {
    contenido1.value = data;
    console.log("Personajes:", data);
  })
  .catch(error => {
    console.error("No se encontró el contenido de personajes", error);
  });

fetch('http://localhost:3000/api/selectodo/produccion')
  .then(response => response.json())
  .then(data => {
    contenido2.value = data;
    console.log("Producción:", data);
  })
  .catch(error => {
    console.error("No se encontró el contenido de produccion", error);
  });

const manejarNuevoDato = (nuevoDato) => {
  console.log('Nuevo dato recibido desde el formulario:', nuevoDato);
  fetch('http://localhost:3000/api/selectodo/personajes')
    .then(response => response.json())
    .then(data => {
      contenido1.value = data;
    })
    .catch(error => {
      console.error("Error al actualizar la tabla de personajes", error);
    });
};
</script>

<template>
  <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css">
  <div class="container text-center">
    <div class="row">
      <Tabla
        :encabezados="encabezados3"
        :contenido="contenido2"
      />
      <Formulario
        :formulario="formularios"
        @agregar-dato="manejarNuevoDato"
      />
      <Tabla
        :encabezados="encabezados2"
        :contenido="contenido1"
      />
    </div>
  </div>
</template>
