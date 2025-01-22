<script setup>
import { ref, watch, defineProps } from 'vue';
import axios from 'axios';

const props = defineProps({
  curso: Number,
  etapa: String
});

const asignaturas = ref([]);
const columnasGrupos = ref([]);
const asignaturasSeleccionadas = ref([]);

const cargarAsignaturas = async () => {
  if (!props.curso || !props.etapa) {
    asignaturas.value = [];
    columnasGrupos.value = [];
    return;
  }
  try {
    const response = await axios.get('http://localhost:8081/direccionVentana3/asignaturas', {
      params: {
        curso: props.curso,
        etapa: props.etapa,
      },
    });
    asignaturas.value = response.data;

    const gruposSet = new Set();
    asignaturas.value.forEach(asignatura => {
      Object.keys(asignatura.numeroAlumnosEnGrupo).forEach(grupo => {
        gruposSet.add(grupo);
      })
    })
    columnasGrupos.value = Array.from(gruposSet);
  } catch (error) {
    console.error('Error al cargar asignaturas', error);
  }

}; 


const addBloque = async () => {
  if (asignaturasSeleccionadas.value.length < 2) {
    console.error("Debe seleccionar al menos dos asignaturas para crear un bloque.");
    return;
  }

  const nombresSeleccionados = asignaturasSeleccionadas.value.map(a => a.nombre)

  try {
    const response = await axios.post("http://localhost:8081/direccionVentana3/bloques", {
      curso: props.curso,
      etapa: props.etapa, // Envía solo los nombres de las asignaturas
      asignaturas: nombresSeleccionados
    });

    console.log("Bloque creado exitosamente:", response.data);
    // Puedes mostrar una notificación o actualizar el estado después de crear el bloque
  } catch (error) {
    console.error("Error al crear el bloque:", error.response?.data || error);
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
          <th class="border border-gray-400 px-4 py-2"></th>
          <th class="border border-gray-400 px-4 py-2">Nombre</th>
          <th class="border border-gray-400 px-4 py-2">Grupo</th>
          <th class="border border-gray-400 px-4 py-2">Número Total de Alumnos</th>
          <!-- Renderiza dinámicamente las columnas de grupos -->
          <th 
            v-for="grupo in columnasGrupos" 
            :key="grupo" 
            class="border border-gray-400 px-4 py-2"
          >
            Grupo {{ grupo }}
          </th>
          <th>Bloque</th>
        </tr>
      </thead>
      <tbody>
        <tr 
          v-for="asignatura in asignaturas" 
          :key="`${asignatura.curso}-${asignatura.etapa}-${asignatura.nombre}`"
        >
          <td>
            <input type="checkbox" v-model="asignaturasSeleccionadas" :value="asignatura">
          </td>
          <td class="border border-gray-400 px-4 py-2">{{ asignatura.nombre }}</td>
          <td class="border border-gray-400 px-4 py-2">{{ asignatura.grupo }}</td>
          <td class="border border-gray-400 px-4 py-2">{{ asignatura.numeroDeAlumnos }}</td>
          <!-- Renderiza dinámicamente los valores de cada grupo -->
          <td 
            v-for="grupo in columnasGrupos" 
            :key="grupo" 
            class="border border-gray-400 px-4 py-2"
          >
            {{ asignatura.numeroAlumnosEnGrupo[grupo] || 0 }}
          </td>
        </tr>
      </tbody>
    </table>

    <button @click="addBloque" class="mt-4 bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
      Crear Bloque
    </button>
  </div>
  <div v-else class="mt-6 text-center">
    <p class="text-gray-500">No hay asignaturas disponibles para el curso y etapa seleccionados.</p>
  </div>
</template>


<style scoped>
/* Opcional: Agregar estilos personalizados */
</style>
