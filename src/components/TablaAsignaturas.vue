<script setup>
import { ref, watch, defineProps } from 'vue';
import axios from 'axios';

const props = defineProps({
  curso: Number,
  etapa: String,
});

const asignaturas = ref([]);
const columnasGrupos = ref([]);
const asignaturasSeleccionadas = ref([]);
const loading = ref(false);
const errorMensaje = ref("");

// Cargar asignaturas
const cargarAsignaturas = async () => {
  if (!props.curso || !props.etapa) {
    asignaturas.value = [];
    columnasGrupos.value = [];
    return;
  }

  loading.value = true;
  errorMensaje.value = "";

  try {
    const response = await axios.get('http://localhost:8081/direccionVentana3/asignaturas', {
      params: { curso: props.curso, etapa: props.etapa },
    });

    console.log("Respuesta del servidor:", response.data);

    asignaturas.value = response.data;

    // Crear lista de grupos dinámicamente
    const gruposSet = new Set();
    asignaturas.value.forEach((asignatura) => {
      Object.keys(asignatura.numeroAlumnosEnGrupo).forEach((grupo) => {
        gruposSet.add(grupo);
      });
    });

    columnasGrupos.value = Array.from(gruposSet);
  } catch (error) {
    errorMensaje.value = "Error al cargar asignaturas. Por favor, inténtelo de nuevo.";
    console.error('Error al cargar asignaturas:', error);
  } finally {
    loading.value = false;
  }
};

// Crear bloque
const addBloque = async () => {
  if (asignaturasSeleccionadas.value.length < 2) {
    errorMensaje.value = "Debe seleccionar al menos dos asignaturas para crear un bloque.";
    return;
  }

  loading.value = true;
  errorMensaje.value = "";

  const nombresSeleccionados = asignaturasSeleccionadas.value.map((a) => a.nombre);

  try {
    const response = await axios.post("http://localhost:8081/direccionVentana3/bloques", null, {
      params: {
        curso: props.curso,
        etapa: props.etapa,
        asignaturas: nombresSeleccionados.join(","),
      },
    });

    console.log("Bloque creado exitosamente:", response.data);
    asignaturasSeleccionadas.value = []; 
    cargarAsignaturas(); 
  } catch (error) {
    errorMensaje.value = "Error al crear el bloque. Intente nuevamente.";
    console.error("Error al crear el bloque:", error.response?.data || error);
  } finally {
    loading.value = false;
  }
};

// Eliminar bloque
const eliminarBloque = async (asignatura) => {
  loading.value = true;
  errorMensaje.value = "";

  try {
    await axios.delete(`http://localhost:8081/direccionVentana3/eliminarBloque`, {
      params: {
        curso: props.curso,
        etapa: props.etapa,
        asignatura: asignatura.nombre,
        grupo: asignatura.grupo
      },
    });

    console.log(`Bloque eliminado para la asignatura: ${asignatura.nombre}`);
    cargarAsignaturas();
  } catch (error) {
    errorMensaje.value = "Error al eliminar el bloque. Intente nuevamente.";
    console.error("Error al eliminar el bloque:", error.response?.data || error);
  } finally {
    loading.value = false;
  }
};

watch([() => props.curso, () => props.etapa], cargarAsignaturas, { immediate: true });
</script>

<template>
  <div class="mt-6">
    <div v-if="errorMensaje" class="text-red-500 mb-4 text-center">
      {{ errorMensaje }}
    </div>

    <div v-if="loading" class="text-center text-gray-500">
      Cargando datos, por favor espere...
    </div>

    <div v-if="asignaturas.length > 0 && !loading">
      <table class="table-auto border-collapse border border-gray-400 w-full">
        <thead>
          <tr class="bg-gray-200">
            <th class="border border-gray-400 px-4 py-2"></th>
            <th class="border border-gray-400 px-4 py-2">Nombre</th>
            <th class="border border-gray-400 px-4 py-2">Grupo</th>
            <th class="border border-gray-400 px-4 py-2">Número Total de Alumnos</th>
            <th v-for="grupo in columnasGrupos" :key="grupo" class="border border-gray-400 px-4 py-2">
              Grupo {{ grupo }}
            </th>
            <th class="border border-gray-400 px-4 py-2">Bloque</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="asignatura in asignaturas" :key="`${asignatura.curso}-${asignatura.etapa}-${asignatura.nombre}`">
            <td>
              <input type="checkbox" v-model="asignaturasSeleccionadas" :value="asignatura" />
            </td>
            <td class="border border-gray-400 px-4 py-2">{{ asignatura.nombre }}</td>
            <td class="border border-gray-400 px-4 py-2">{{ asignatura.grupo }}</td>
            <td class="border border-gray-400 px-4 py-2">{{ asignatura.numeroDeAlumnos }}</td>
            <td v-for="grupo in columnasGrupos" :key="grupo" class="border border-gray-400 px-4 py-2">
              {{ asignatura.numeroAlumnosEnGrupo[grupo] || 0 }}
            </td>
            <td class="border border-gray-400 px-4 py-2">
              <div v-if="asignatura.bloqueId !== undefined && asignatura.bloqueId !== null">
                Bloque {{ asignatura.bloqueId }}
                <button
                  @click="eliminarBloque(asignatura)"
                  class="ml-2 text-red-500 hover:underline"
                >
                  X
                </button>
              </div>
              <div v-else class="text-gray-400">Sin bloque</div>
            </td>
          </tr>
        </tbody>
      </table>

      <button
        @click="addBloque"
        :disabled="asignaturasSeleccionadas.length < 2 || loading"
        class="mt-4 bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600 disabled:opacity-50"
      >
        {{ loading ? "Procesando..." : "Crear Bloque" }}
      </button>
    </div>

    <div v-else-if="!loading" class="mt-6 text-center">
      <p class="text-gray-500">No hay asignaturas disponibles para el curso y etapa seleccionados.</p>
    </div>
  </div>
</template>



<style scoped>

</style>
