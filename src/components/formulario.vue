<template>
  <div class="card">
    Agregar {{ tabla }}
  <div class="card-header">
  </div>
  <div class="card-body">
    <form @submit.prevent="Insertar">
    <div v-for="(campo, index) in labels" :key="index" class="mb-3">
      <label :for="campo" class="form-label">
        {{ campo }}
      </label>
      <input
        type="text"
        class="form-control form-control-lg"
        :id="campo"
        :placeholder="campo"
        v-model="json[campo]"
      />
    </div>
    <button type="submit" class="btn btn-success">
      Agregar
    </button>
  </form>
  </div>
</div>
</template>

<script setup>
import { reactive } from 'vue';
import axios from 'axios';

const props = defineProps({
  labels: {
    type: Array,
    required: true
  },
  tabla: {
    type: String,
    default: 'personajes'
  }
});
const json = reactive({});
props.labels.forEach((campo) => {
  json[campo] = "";
});

const Insertar = async () => {
  console.log(json);
  try {
    const response = await axios.post(
      `http://localhost:3000/api/insertar/${props.tabla}`,
      json,
      { headers: { 'Content-Type': 'application/json' } }
    );
    console.log("Datos insertados:", response.data);
    window.location.reload();
  } catch (error) {
    console.error("Error al insertar datos:", error);
  }
};
</script>
