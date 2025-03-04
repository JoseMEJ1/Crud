<script setup>
import { ref, watch } from 'vue';

const props = defineProps({
  formulario: Array
});
const emit = defineEmits(['agregar-dato']);

const formData = ref({});

watch(
  () => props.formulario,
  (newForm) => {
    formData.value = Object.fromEntries(newForm.map(campo => [campo, '']));
  },
  { immediate: true }
);

const enviarFormulario = async () => {
  emit('agregar-dato', { ...formData.value });

  const payload = {
    nombre: formData.value.nombre,
    edad: formData.value.edad,
    pais_origen: formData.value.pais, 
    oficio: formData.value.oficio
  };

  try {
    const response = await fetch('http://localhost:3000/api/insertar/personajes', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    });
    if (!response.ok) {
      throw new Error('Error en la petición');
    }
    const result = await response.json();
    console.log('Respuesta de la API:', result);

    formData.value = Object.fromEntries(props.formulario.map(campo => [campo, '']));
  } catch (error) {
    console.error('Error al enviar los datos:', error);
  }
};
</script>

<template>
  <div class="col-4">
    <form @submit.prevent="enviarFormulario">
      <div v-for="(contenido, i) in formulario" :key="i">
        <label class="form-label">{{ contenido }}</label>
        <input 
          type="text" 
          v-model="formData[contenido]" 
          class="form-control" 
          :id="contenido"
        />
      </div>
      <button type="submit" class="btn btn-dark">Guardar</button>
    </form>
  </div>
</template>
