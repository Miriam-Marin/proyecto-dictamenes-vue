<template>
  <section
    v-if="panel.id"
    :id="panel.id"
  >
    <img
      src="/img/logo.png"
      alt="CEFPP Veracruz"
      class="logo-panel"
    />
    <h2>{{ panel.titulo }}</h2>
    <h3>Seleccione la actividad que desea realizar</h3>
  </section>
</template>

<script setup>
import { computed } from 'vue';

const props = defineProps({
  rol: { type: String, required: true }
});

const panel = computed(() => {
  switch (props.rol) {
    case 'administrador':
      return {
        id: 'adminPanel',
        titulo: 'Bienvenido a SISTPEC, Administrador'
      };
    case 'responsableLaboratorio':
      return {
        id: 'responsablelabPanel',
        titulo: 'Bienvenido a SISTPEC, Responsable de laboratorio'
      };
    case 'recepcionista':
      return {
        id: 'recepcionistaPanel',
        titulo: 'Bienvenido a SISTPEC, Recepcionista'
      };
    case 'coordinador':
      return {
        id: 'coordinadorPanel',
        titulo: 'Bienvenido a SISTPEC, Coordinador'
      };
    case 'mvzAutorizado':
      return {
        id: 'mvzPanel',
        titulo: 'Bienvenido a SISTPEC, MVZ'
      };
    default:
      return {
        id: '',
        titulo: ''
      };
  }
});
</script>
