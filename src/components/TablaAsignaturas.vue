<script setup>
import { ref, watch, defineProps } from 'vue';
import axios from 'axios';

const props = defineProps({
  curso: Number,
  etapa: String
});

const asignaturas = ref([]);

const cargarAsignaturas = async () => {
  if (!props.curso || !props.etapa) {
    asignaturas.value = [];
    return;
  }
  try {
    const response = await axios.get('http://localhost:8081/direccion/asignaturas', {
      params: {
        curso: props.curso,
        etapa: props.etapa,
      },
    });
    asignaturas.value = response.data;
  } catch (error) {
    console.error('Error al cargar asignaturas', error);
  }
};

watch([() => props.curso, () => props.etapa], () => {
  cargarAsignaturas();
}, { immediate: true });
</script>

<template>
  <div v-if="asignaturas.length > 0" class="mt-6">
    <table class="table-auto border-collapse border border-gray-400 w-full">
      <thead>
        <tr class="bg-gray-200">
          <th class="border border-gray-400 px-4 py-2">Nombre</th>
          <th class="border border-gray-400 px-4 py-2">Grupo</th>
          <th class="border border-gray-400 px-4 py-2">Número Total de Alumnos</th>
          <th class="border border-gray-400 px-4 py-2">Alumnos por Grupo</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="asignatura in asignaturas" :key="`${asignatura.curso}-${asignatura.etapa}-${asignatura.nombre}`">
          <td class="border border-gray-400 px-4 py-2">{{ asignatura.nombre }}</td>
          <td class="border border-gray-400 px-4 py-2">{{ asignatura.grupo }}</td>
          <td class="border border-gray-400 px-4 py-2">{{ asignatura.numeroDeAlumnos }}</td>
          <td class="border border-gray-400 px-4 py-2">
            <ul>
              <li v-for="(numAlumnos, grupo) in asignatura.numeroAlumnosEnGrupo" :key="grupo">
                {{ grupo }}: {{ numAlumnos }}
              </li>
            </ul>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
  <div v-else class="mt-6 text-center">
    <p class="text-gray-500">No hay asignaturas disponibles para el curso y etapa seleccionados.</p>
  </div>
</template>

<style scoped>
/* Opcional: Agregar estilos personalizados */
</style>
