<script setup>
    import { ref } from 'vue';
    import CursosEtapas from './CursosEtapas.vue';
    import TablaAsignaturas from './TablaAsignaturas.vue';

    const cursoSeleccionado = ref(null);
    const etapaSeleccionada = ref('');

    const actualizarSeleccion = ({ curso, etapa }) => {
      cursoSeleccionado.value = curso;
      etapaSeleccionada.value = etapa;
    };
</script>

<template>
  <div>
    <CursosEtapas @actualizar-select="actualizarSeleccion" />
    <TablaAsignaturas :curso="cursoSeleccionado" :etapa="etapaSeleccionada" />
  </div>
</template>
