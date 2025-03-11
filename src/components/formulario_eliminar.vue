<template>
    <div class="card">
      <div class="card-header">
        Eliminar {{ tabla }}
      </div>
      <div class="card-body">
        <form @submit.prevent="eliminar">
          <div class="mb-3">
            <input type="text" id="id" v-model="id" class="form-control form-control-lg" placeholder="Ingresa el ID"/>
          </div>
          <button type="submit" class="btn btn-danger">
            Eliminar
          </button>
        </form>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref } from 'vue';
  import axios from 'axios';
  
  const props = defineProps({
    tabla: {
      type: String,
      default: 'personajes'
    }
  });
  const id = ref('');
  
  const eliminar = async () => {
    try {
      const response = await axios.delete(`http://localhost:3000/api/delete/${props.tabla}/${id.value}`);
      window.location.reload();
    } catch (error) {
      console.error("Error al eliminar el registro:", error);
    }
  };
  </script>
  