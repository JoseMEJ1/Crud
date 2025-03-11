<script setup>
import { ref } from 'vue';
import Tabla from './components/tabla.vue';
import Formulario from './components/formulario.vue';
import Eliminar from './components/formulario_eliminar.vue';

const encabezados3 = ['id', 'nombre', 'edad', 'pais', 'oficio'];
const encabezados2 = ['id', 'nombre', 'edad', 'pais', 'oficio'];
const formularios = ["nombre","edad","pais_origen","oficio"];

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
      <div class="col-sm-5">
        <Tabla
          :encabezados="encabezados2"
          :contenido="contenido1"
          :tabla="['Personajes']"
        />
      </div>
      <div class="col-sm-2"></div>
      <div class="col-sm-5">
        <Tabla
          :encabezados="encabezados3"
          :contenido="contenido2"
          :tabla="['Produccion']"
        />
      </div>
    </div>
    <div class="row">
      <div class="col-sm-5">
        <Formulario :labels="['nombre', 'edad', 'pais_origen', 'oficio']" tabla="personajes" />
      </div>
      <div class="col-sm-2"></div>
      <div class="col-sm-5">
        <Formulario :labels="['nombre', 'edad', 'pais_origen', 'oficio']" tabla="produccion" />
      </div>
    </div>
    <div class="row">
        <div class="col-sm-5">
          <Eliminar :tabla="'personajes'" />
        </div>
        <div class="col-sm-2"></div>
        <div class="col-sm-5">
          <Eliminar :tabla="'produccion'" />
        </div>
      </div>
    </div>
  
</template>
