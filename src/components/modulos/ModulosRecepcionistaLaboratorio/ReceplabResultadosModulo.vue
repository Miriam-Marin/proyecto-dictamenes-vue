<template>
  <!-- Barra de acciones -->
  <section class="modulo-acciones">
    <span class="modulo-acciones-titulo">Acciones disponibles</span>

    <div class="modulo-acciones-botones">
      <button
        v-for="accion in acciones"
        :key="accion.id"
        type="button"
        class="sistpec-btn-accion"
        :class="{ active: selectedAction === accion.id }"
        @click="cambiarAccion(accion.id)"
      >
        {{ accion.label }}
      </button>
    </div>

    <div class="sistpec-info-box">
      <p class="sistpec-info-text">{{ descripcionAccionActual }}</p>
    </div>
  </section>

  <section class="modulo-contenido" ref="moduloContenidoRef">
    <!-- ALERTAS -->
    <div v-if="errores.length" class="modulo-alert modulo-alert--error">
      <ul>
        <li v-for="(e, i) in errores" :key="i">{{ e }}</li>
      </ul>
    </div>

    <div v-if="mensajeExito" class="modulo-alert modulo-alert--success">
      {{ mensajeExito }}
    </div>

    <!-- ===================== REGISTRAR ===================== -->
    <div v-if="selectedAction === 'registrar'">
      <h3 class="subtitulo">Registrar resultados</h3>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>Folio de muestra</label>
          <input v-model="formReg.id_muestra" type="text" placeholder="Ej. M-0004-2025" />
        </div>

        <div class="sistpec-form-group">
          <label>Número de arete</label>
          <input v-model="formReg.arete" type="text" placeholder="Ej. 3011520044" />
        </div>

        <div class="sistpec-form-group">
          <label>Resultado</label>
          <select v-model="formReg.resultado">
            <option value="">Seleccione</option>
            <option value="Negativo">Negativo</option>
            <option value="Positivo">Positivo</option>
          </select>
        </div>

        <div class="sistpec-form-group">
          <label>Comentarios</label>
          <input v-model="formReg.comentarios" type="text" placeholder="Observaciones (opcional)" />
        </div>
      </div>

      <div class="sistpec-search-bar fechas-bar">
        <div class="sistpec-form-group">
          <label>Fecha de carga (demo)</label>
          <input v-model="formReg.fecha_carga" type="date" />
        </div>

        <div class="sistpec-form-group sistpec-search-actions right">
          <button class="sistpec-btn-primary" type="button" @click="registrarResultado">
            GUARDAR RESULTADO
          </button>
          <button class="sistpec-btn-secondary" type="button" @click="limpiarRegistro">
            LIMPIAR
          </button>
        </div>
      </div>

      <div class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Folio muestra</th>
              <th>Arete</th>
              <th>UPP</th>
              <th>MVZ</th>
              <th>Propietario</th>
              <th>Resultado</th>
              <th>Estado</th>
              <th>Fecha registro</th>
              <th>Fecha carga</th>
              <th>Comentarios</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="r in resultadosDemo" :key="r.id">
              <td>{{ r.id_muestra }}</td>
              <td>{{ r.arete }}</td>
              <td>{{ r.upp }}</td>
              <td>{{ r.mvz }}</td>
              <td>{{ r.propietario }}</td>
              <td>{{ r.resultado }}</td>
              <td>
                <span class="badge" :class="badgeEstadoClase(r.estado)">
                  {{ r.estado }}
                </span>
              </td>
              <td>{{ r.fecha_registro }}</td>
              <td>{{ r.fecha_carga }}</td>
              <td class="comentarios-cell">
                <div class="comentarios-wrap">
                  <span v-if="r.comentarios" class="comentarios-text">{{ r.comentarios }}</span>
                  <span v-else class="comentarios-vacio">-</span>
                </div>
              </td>
            </tr>

            <tr v-if="resultadosDemo.length === 0">
              <td colspan="10" class="sin-resultados">Sin registros (demo).</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ===================== CONSULTAR ===================== -->
    <div v-else-if="selectedAction === 'consultar'">
      <header class="modulo-header">
        <h3 class="subtitulo">Consultar resultados</h3>
        <p class="modulo-subtitle">Capture al menos un criterio de búsqueda.</p>
      </header>

      <div v-if="mostrarAlertaConsultar" class="modulo-alert modulo-alert--error">
        Debe capturar <strong>al menos un criterio de búsqueda</strong> para realizar la consulta.
      </div>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>Número de caso</label>
          <input v-model="filtros.caso" type="text" placeholder="Ej. BR25-001" />
        </div>

        <div class="sistpec-form-group">
          <label>Folio de muestra</label>
          <input v-model="filtros.id_muestra" type="text" placeholder="Ej. M-0001-2025" />
        </div>

        <div class="sistpec-form-group">
          <label>Arete</label>
          <input v-model="filtros.arete" type="text" placeholder="Ej. 3011520051" />
        </div>

        <div class="sistpec-form-group">
          <label>UPP</label>
          <input v-model="filtros.upp" type="text" placeholder="Clave o nombre UPP" />
        </div>
      </div>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>MVZ</label>
          <input v-model="filtros.mvz" type="text" placeholder="Nombre MVZ" />
        </div>

        <div class="sistpec-form-group">
          <label>Propietario</label>
          <input v-model="filtros.propietario" type="text" placeholder="Nombre del propietario" />
        </div>

        <div class="sistpec-form-group">
          <label>Resultado</label>
          <select v-model="filtros.resultado">
            <option value="">Todos</option>
            <option value="Negativo">Negativo</option>
            <option value="Positivo">Positivo</option>
          </select>
        </div>

        <div class="sistpec-form-group">
          <label>Estado</label>
          <select v-model="filtros.estado">
            <option value="">Todos</option>
            <option value="Pendiente">Pendiente</option>
            <option value="Entregado">Entregado</option>
          </select>
        </div>
      </div>

      <div class="sistpec-search-bar fechas-bar">
        <div class="sistpec-form-group sistpec-form-group-inline">
          <label>Fecha de carga</label>
          <div class="sistpec-form-inline-inputs">
            <input v-model="filtros.fecha_inicio" type="date" />
            <span class="vigencia-sep">a</span>
            <input v-model="filtros.fecha_fin" type="date" />
          </div>
        </div>

        <div class="sistpec-form-group sistpec-search-actions right">
          <button class="sistpec-btn-primary" type="button" @click="buscar">
            BUSCAR
          </button>
          <button class="sistpec-btn-secondary" type="button" @click="limpiarFiltros">
            LIMPIAR FILTROS
          </button>
        </div>
      </div>

      <div v-if="consultado" class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Caso</th>
              <th>Folio muestra</th>
              <th>Arete</th>
              <th>UPP</th>
              <th>MVZ</th>
              <th>Propietario</th>
              <th>Resultado</th>
              <th>Estado</th>
              <th>Fecha carga</th>
              <th>Comentarios</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="r in resultadosFiltrados" :key="r.id">
              <td>{{ r.caso }}</td>
              <td>{{ r.id_muestra }}</td>
              <td>{{ r.arete }}</td>
              <td>{{ r.upp }}</td>
              <td>{{ r.mvz }}</td>
              <td>{{ r.propietario }}</td>
              <td>{{ r.resultado }}</td>
              <td>
                <span class="badge" :class="badgeEstadoClase(r.estado)">
                  {{ r.estado }}
                </span>
              </td>
              <td>{{ r.fecha_carga }}</td>
              <td class="comentarios-cell">
                <div class="comentarios-wrap">
                  <span v-if="r.comentarios" class="comentarios-text">{{ r.comentarios }}</span>
                  <span v-else class="comentarios-vacio">-</span>
                </div>
              </td>
            </tr>

            <tr v-if="resultadosFiltrados.length === 0">
              <td colspan="10" class="sin-resultados">
                No se encontraron resultados con los criterios capturados.
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- ===================== EDITAR (PENDIENTE) ===================== -->
    <div v-else-if="selectedAction === 'editar'">
      <h3 class="subtitulo">Editar resultados (solo estado Pendiente)</h3>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>Folio muestra</label>
          <input v-model="filtrosEditar.id_muestra" type="text" placeholder="Ej. M-0001-2025" />
        </div>

        <div class="sistpec-form-group">
          <label>Arete</label>
          <input v-model="filtrosEditar.arete" type="text" placeholder="Ej. 3011520051" />
        </div>

        <div class="sistpec-form-group">
          <label>Resultado</label>
          <select v-model="filtrosEditar.resultado">
            <option value="">Todos</option>
            <option value="Negativo">Negativo</option>
            <option value="Positivo">Positivo</option>
          </select>
        </div>

        <div class="sistpec-form-group sistpec-search-actions right">
          <button class="sistpec-btn-primary" type="button" @click="buscarEditar">BUSCAR</button>
          <button class="sistpec-btn-secondary" type="button" @click="limpiarEditar">LIMPIAR</button>
        </div>
      </div>

      <div v-if="editarBuscado" class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Folio</th>
              <th>Arete</th>
              <th>Resultado</th>
              <th>Estado</th>
              <th>Comentarios</th>
              <th>Acción</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="r in resultadosParaEditar" :key="r.id">
              <td>{{ r.id_muestra }}</td>
              <td>{{ r.arete }}</td>
              <td>{{ r.resultado }}</td>
              <td>
                <span class="badge" :class="badgeEstadoClase(r.estado)">
                  {{ r.estado }}
                </span>
              </td>
              <td class="comentarios-cell">
                <div class="comentarios-wrap">
                  <span v-if="r.comentarios" class="comentarios-text">{{ r.comentarios }}</span>
                  <span v-else class="comentarios-vacio">-</span>
                </div>
              </td>
              <td>
                <button
                  type="button"
                  class="sistpec-btn-secondary sistpec-btn-sm"
                  :disabled="r.estado !== 'Pendiente'"
                  @click="seleccionarEdicion(r)"
                >
                  EDITAR
                </button>
              </td>
            </tr>

            <tr v-if="resultadosParaEditar.length === 0">
              <td colspan="6" class="sin-resultados">Sin coincidencias.</td>
            </tr>
          </tbody>
        </table>
      </div>

      <div v-if="editando" class="sistpec-edit-panel">
        <h4 class="subtitulo-secundario">Editando: {{ editando.id_muestra }}</h4>

        <form class="sistpec-form" @submit.prevent="guardarEdicion">
          <div class="sistpec-form-row">
            <div class="sistpec-form-group">
              <label>Resultado</label>
              <select v-model="editando.resultado">
                <option value="Negativo">Negativo</option>
                <option value="Positivo">Positivo</option>
              </select>
            </div>

            <div class="sistpec-form-group">
              <label>Fecha de carga</label>
              <input v-model="editando.fecha_carga" type="date" />
            </div>
          </div>

          <div class="sistpec-form-row">
            <div class="sistpec-form-group">
              <label>Comentarios</label>
              <textarea v-model="editando.comentarios" rows="3" placeholder="Observaciones / incidencias"></textarea>
            </div>
            <div></div>
          </div>

          <div class="sistpec-form-actions">
            <button class="sistpec-btn-primary" type="submit">GUARDAR</button>
            <button class="sistpec-btn-secondary" type="button" @click="cancelarEdicion">
              CANCELAR
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- ===================== ELIMINAR (PENDIENTE) ===================== -->
    <div v-else-if="selectedAction === 'eliminar'">
      <h3 class="subtitulo">Eliminar resultados (solo estado Pendiente)</h3>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>Folio muestra</label>
          <input v-model="filtrosEliminar.id_muestra" type="text" placeholder="Ej. M-0001-2025" />
        </div>

        <div class="sistpec-form-group">
          <label>Arete</label>
          <input v-model="filtrosEliminar.arete" type="text" placeholder="Ej. 3011520051" />
        </div>

        <div class="sistpec-form-group">
          <label>Estado</label>
          <select v-model="filtrosEliminar.estado">
            <option value="">Todos</option>
            <option value="Pendiente">Pendiente</option>
            <option value="Entregado">Entregado</option>
          </select>
        </div>

        <div class="sistpec-form-group sistpec-search-actions right">
          <button class="sistpec-btn-primary" type="button" @click="buscarEliminar">BUSCAR</button>
          <button class="sistpec-btn-secondary" type="button" @click="limpiarEliminar">LIMPIAR</button>
        </div>
      </div>

      <div v-if="eliminarBuscado" class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Folio</th>
              <th>Arete</th>
              <th>Resultado</th>
              <th>Estado</th>
              <th>Comentarios</th>
              <th>Acción</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="r in resultadosParaEliminar" :key="r.id">
              <td>{{ r.id_muestra }}</td>
              <td>{{ r.arete }}</td>
              <td>{{ r.resultado }}</td>
              <td>
                <span class="badge" :class="badgeEstadoClase(r.estado)">
                  {{ r.estado }}
                </span>
              </td>
              <td class="comentarios-cell">
                <div class="comentarios-wrap">
                  <span v-if="r.comentarios" class="comentarios-text">{{ r.comentarios }}</span>
                  <span v-else class="comentarios-vacio">-</span>
                </div>
              </td>
              <td>
                <button
                  type="button"
                  class="sistpec-btn-danger sistpec-btn-sm"
                  :disabled="r.estado !== 'Pendiente'"
                  @click="eliminarResultado(r)"
                >
                  ELIMINAR
                </button>
              </td>
            </tr>

            <tr v-if="resultadosParaEliminar.length === 0">
              <td colspan="6" class="sin-resultados">Sin coincidencias.</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-else>
      <h3 class="subtitulo">Acción no disponible</h3>
    </div>
  </section>
</template>

<script setup>
import { computed, ref, nextTick } from 'vue';

defineProps({
  codigo: { type: String, required: false, default: '' },
  rol:    { type: String, required: false, default: '' }
});

const moduloContenidoRef = ref(null);
function scrollAlContenido() {
  nextTick(() => {
    if (!moduloContenidoRef.value) return;
    const rect = moduloContenidoRef.value.getBoundingClientRect();
    const offset = 90;
    window.scrollTo({ top: rect.top + window.scrollY - offset, behavior: 'smooth' });
  });
}

const acciones = [
  { id: 'registrar', label: 'REGISTRAR' },
  { id: 'consultar', label: 'CONSULTAR' },
  { id: 'editar', label: 'EDITAR (PENDIENTE)' },
  { id: 'eliminar', label: 'ELIMINAR (PENDIENTE)' }
];

const selectedAction = ref('registrar');
const errores = ref([]);
const mensajeExito = ref('');

function cambiarAccion(id) {
  selectedAction.value = id;
  errores.value = [];
  mensajeExito.value = '';
  scrollAlContenido();
}

const descripcionAccionActual = computed(() => {
  switch (selectedAction.value) {
    case 'registrar': return 'Registra resultados por muestra (demo).';
    case 'consultar': return 'Consulta resultados (requiere al menos un criterio).';
    case 'editar':    return 'Edita únicamente resultados con estado Pendiente.';
    case 'eliminar':  return 'Elimina únicamente resultados con estado Pendiente.';
    default: return '';
  }
});

// ===== DEMO DATA =====
const resultadosDemo = ref([
  {
    id: 1,
    caso: 'BR25-001',
    id_muestra: 'M-0001-2025',
    arete: '3011520051',
    upp: 'UPP-VER-001',
    mvz: 'MVZ Juan Pérez',
    propietario: 'José López Ramírez',
    resultado: 'Negativo',
    estado: 'Entregado',
    fecha_registro: '2025-12-10',
    fecha_carga: '2025-12-12',
    comentarios: 'Sin observaciones.'
  },
  {
    id: 2,
    caso: 'BR25-002',
    id_muestra: 'M-0002-2025',
    arete: '3011520062',
    upp: 'UPP-VER-014',
    mvz: 'MVZ Ana López',
    propietario: 'María Hernández Torres',
    resultado: 'Positivo',
    estado: 'Pendiente',
    fecha_registro: '2025-12-15',
    fecha_carga: '2025-12-16',
    comentarios: ''
  }
]);

function badgeEstadoClase(estado) {
  return estado === 'Entregado' ? 'badge--activo' : 'badge--pendiente';
}

// ===== REGISTRAR =====
function hoyISO() {
  const d = new Date();
  const yyyy = d.getFullYear();
  const mm = String(d.getMonth() + 1).padStart(2, '0');
  const dd = String(d.getDate()).padStart(2, '0');
  return `${yyyy}-${mm}-${dd}`;
}

const formReg = ref({
  id_muestra: '',
  arete: '',
  resultado: '',
  comentarios: '',
  fecha_carga: hoyISO()
});

function limpiarRegistro() {
  formReg.value = { id_muestra: '', arete: '', resultado: '', comentarios: '', fecha_carga: hoyISO() };
  errores.value = [];
  mensajeExito.value = '';
}

function registrarResultado() {
  errores.value = [];
  mensajeExito.value = '';

  if (!formReg.value.id_muestra.trim()) errores.value.push('Debe capturar el folio de muestra.');
  if (!formReg.value.arete.trim()) errores.value.push('Debe capturar el número de arete.');
  if (!formReg.value.resultado) errores.value.push('Debe seleccionar un resultado.');
  if (errores.value.length) return;

  const nuevo = {
    id: Date.now(),
    caso: 'BR25-XXX', // demo
    id_muestra: formReg.value.id_muestra.trim(),
    arete: formReg.value.arete.trim(),
    upp: 'UPP-VER-XXX',
    mvz: 'MVZ (demo)',
    propietario: 'Propietario (demo)',
    resultado: formReg.value.resultado,
    estado: 'Pendiente',
    fecha_registro: hoyISO(),
    fecha_carga: formReg.value.fecha_carga || hoyISO(),
    comentarios: formReg.value.comentarios.trim()
  };

  resultadosDemo.value = [nuevo, ...resultadosDemo.value];
  mensajeExito.value = 'Resultado registrado correctamente (demo).';
  limpiarRegistro();
}

// ===== CONSULTAR =====
const filtros = ref({
  caso: '',
  id_muestra: '',
  arete: '',
  upp: '',
  mvz: '',
  propietario: '',
  resultado: '',
  estado: '',
  fecha_inicio: '',
  fecha_fin: ''
});

const consultado = ref(false);
const mostrarAlertaConsultar = ref(false);

function hayAlMenosUnFiltro() {
  const f = filtros.value;
  return (
    f.caso.trim() ||
    f.id_muestra.trim() ||
    f.arete.trim() ||
    f.upp.trim() ||
    f.mvz.trim() ||
    f.propietario.trim() ||
    f.resultado ||
    f.estado ||
    f.fecha_inicio ||
    f.fecha_fin
  );
}

function buscar() {
  mostrarAlertaConsultar.value = false;
  if (!hayAlMenosUnFiltro()) {
    consultado.value = false;
    mostrarAlertaConsultar.value = true;
    return;
  }
  consultado.value = true;
}

function limpiarFiltros() {
  filtros.value = {
    caso: '',
    id_muestra: '',
    arete: '',
    upp: '',
    mvz: '',
    propietario: '',
    resultado: '',
    estado: '',
    fecha_inicio: '',
    fecha_fin: ''
  };
  consultado.value = false;
  mostrarAlertaConsultar.value = false;
}

const resultadosFiltrados = computed(() => {
  if (!consultado.value) return [];

  const f = filtros.value;
  const caso = f.caso.trim().toLowerCase();
  const idm = f.id_muestra.trim().toLowerCase();
  const are = f.arete.trim().toLowerCase();
  const upp = f.upp.trim().toLowerCase();
  const mvz = f.mvz.trim().toLowerCase();
  const pro = f.propietario.trim().toLowerCase();
  const res = f.resultado;
  const est = f.estado;
  const fi = f.fecha_inicio;
  const ff = f.fecha_fin;

  return resultadosDemo.value.filter(r => {
    const okCaso = caso ? (r.caso || '').toLowerCase().includes(caso) : true;
    const okId   = idm ? (r.id_muestra || '').toLowerCase().includes(idm) : true;
    const okAre  = are ? (r.arete || '').toLowerCase().includes(are) : true;
    const okUpp  = upp ? (r.upp || '').toLowerCase().includes(upp) : true;
    const okMvz  = mvz ? (r.mvz || '').toLowerCase().includes(mvz) : true;
    const okPro  = pro ? (r.propietario || '').toLowerCase().includes(pro) : true;
    const okRes  = res ? r.resultado === res : true;
    const okEst  = est ? r.estado === est : true;

    let okF = true;
    if (fi) okF = okF && (r.fecha_carga >= fi);
    if (ff) okF = okF && (r.fecha_carga <= ff);

    return okCaso && okId && okAre && okUpp && okMvz && okPro && okRes && okEst && okF;
  });
});

// ===== EDITAR =====
const filtrosEditar = ref({ id_muestra: '', arete: '', resultado: '' });
const editarBuscado = ref(false);
const editando = ref(null);

function buscarEditar() {
  editarBuscado.value = true;
  editando.value = null;
}

function limpiarEditar() {
  filtrosEditar.value = { id_muestra: '', arete: '', resultado: '' };
  editarBuscado.value = false;
  editando.value = null;
}

const resultadosParaEditar = computed(() => {
  if (!editarBuscado.value) return [];
  const f = filtrosEditar.value;
  const idm = f.id_muestra.trim().toLowerCase();
  const are = f.arete.trim().toLowerCase();
  const res = f.resultado;

  return resultadosDemo.value.filter(r => {
    const okId = idm ? r.id_muestra.toLowerCase().includes(idm) : true;
    const okA  = are ? r.arete.toLowerCase().includes(are) : true;
    const okR  = res ? r.resultado === res : true;
    return okId && okA && okR;
  });
});

function seleccionarEdicion(r) {
  errores.value = [];
  mensajeExito.value = '';

  if (r.estado !== 'Pendiente') {
    errores.value.push('Solo puede editar resultados con estado Pendiente.');
    return;
  }

  editando.value = {
    id: r.id,
    id_muestra: r.id_muestra,
    resultado: r.resultado,
    fecha_carga: r.fecha_carga,
    comentarios: r.comentarios || ''
  };
}

function guardarEdicion() {
  errores.value = [];
  mensajeExito.value = '';

  if (!editando.value) return;
  if (!editando.value.resultado) {
    errores.value.push('Debe seleccionar un resultado.');
    return;
  }

  const idx = resultadosDemo.value.findIndex(x => x.id === editando.value.id);
  if (idx === -1) {
    errores.value.push('No se encontró el resultado.');
    return;
  }

  resultadosDemo.value[idx] = {
    ...resultadosDemo.value[idx],
    resultado: editando.value.resultado,
    fecha_carga: editando.value.fecha_carga || resultadosDemo.value[idx].fecha_carga,
    comentarios: editando.value.comentarios
  };

  mensajeExito.value = 'Resultado actualizado (demo).';
  editando.value = null;
}

function cancelarEdicion() {
  editando.value = null;
}

// ===== ELIMINAR =====
const filtrosEliminar = ref({ id_muestra: '', arete: '', estado: '' });
const eliminarBuscado = ref(false);

function buscarEliminar() {
  eliminarBuscado.value = true;
}

function limpiarEliminar() {
  filtrosEliminar.value = { id_muestra: '', arete: '', estado: '' };
  eliminarBuscado.value = false;
}

const resultadosParaEliminar = computed(() => {
  if (!eliminarBuscado.value) return [];
  const f = filtrosEliminar.value;
  const idm = f.id_muestra.trim().toLowerCase();
  const are = f.arete.trim().toLowerCase();
  const est = f.estado;

  return resultadosDemo.value.filter(r => {
    const okId = idm ? r.id_muestra.toLowerCase().includes(idm) : true;
    const okA  = are ? r.arete.toLowerCase().includes(are) : true;
    const okE  = est ? r.estado === est : true;
    return okId && okA && okE;
  });
});

function eliminarResultado(r) {
  errores.value = [];
  mensajeExito.value = '';

  if (r.estado !== 'Pendiente') {
    errores.value.push('Solo se puede eliminar si el estado es Pendiente.');
    return;
  }

  const ok = window.confirm(`¿Desea eliminar el resultado de "${r.id_muestra}"?`);
  if (!ok) return;

  resultadosDemo.value = resultadosDemo.value.filter(x => x.id !== r.id);
  mensajeExito.value = 'Resultado eliminado (demo).';
}
</script>

<style scoped>
/* ==== base ==== */
.modulo-acciones { margin-bottom: 18px; }
.modulo-acciones-titulo { display:block; font-size:14px; margin-bottom:8px; color:#333; font-weight:600; }
.modulo-acciones-botones { display:flex; flex-wrap:wrap; gap:4px; }
.sistpec-btn-accion { border:none; padding:8px 16px; font-size:12px; font-weight:600; text-transform:uppercase; border-radius:3px; cursor:pointer; background:#2f6b32; color:#fff; letter-spacing:.5px; }
.sistpec-btn-accion.active { background:#244e26; }

.sistpec-info-box { margin-top:10px; padding:10px 14px; border-radius:4px; background:#e1f3e1; border:1px solid #c3e6c3; }
.sistpec-info-text { margin:0; font-size:13px; color:#225522; }

.modulo-contenido { margin-top:10px; }
.subtitulo { font-size:18px; margin:10px 0 15px; color:#333; }
.subtitulo-secundario { font-size:16px; margin:16px 0 10px; color:#333; }
.modulo-header { margin-bottom: 12px; text-align:left; }
.modulo-subtitle { margin:0; font-size:13px; color:#555; }

.modulo-alert { margin-bottom: 12px; padding: 10px 14px; border-radius: 4px; font-size: 13px; }
.modulo-alert--error { background:#fbeaea; border:1px solid #f5c2c2; color:#7a1f1f; }
.modulo-alert--success { background:#e1f3e1; border:1px solid #c3e6c3; color:#225522; }

/* filtros */
.sistpec-search-bar { display:grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap:12px; margin-bottom:16px; }
.fechas-bar { grid-template-columns: repeat(2, minmax(0, 1fr)); }

.sistpec-form-group { display:flex; flex-direction:column; gap:4px; }
.sistpec-form-group label { font-size:13px; font-weight:600; color:#444; }

.sistpec-form-group input,
.sistpec-form-group select,
.sistpec-form-group textarea {
  padding:8px 10px;
  border-radius:4px;
  border:1px solid #ccc;
  font-size:14px;
  outline:none;
}

.sistpec-form-group input:focus,
.sistpec-form-group select:focus,
.sistpec-form-group textarea:focus {
  border-color:#2f6b32;
  box-shadow:0 0 0 1px rgba(47,107,50,.15);
}

.sistpec-form-group-inline .sistpec-form-inline-inputs { display:flex; align-items:center; gap:6px; }
.vigencia-sep { font-size:14px; color:#666; }

.sistpec-search-actions { display:flex; align-items:flex-end; gap:8px; }
.sistpec-search-actions.right { justify-content:flex-end; }

.sistpec-btn-primary { background:#2f6b32; color:#fff; border:none; padding:8px 18px; border-radius:4px; font-size:13px; font-weight:600; cursor:pointer; }
.sistpec-btn-primary:hover { background:#244e26; }
.sistpec-btn-secondary { background:#e0e0e0; color:#333; border:none; padding:8px 18px; border-radius:4px; font-size:13px; font-weight:600; cursor:pointer; }
.sistpec-btn-secondary:hover { background:#d0d0d0; }
.sistpec-btn-sm { padding:4px 10px; font-size:11px; }

.sistpec-btn-danger { background:#7a061e; color:#fff; border:none; padding:6px 14px; border-radius:4px; font-size:12px; font-weight:600; cursor:pointer; }
.sistpec-btn-danger:hover { background:#5a0416; }
.sistpec-btn-danger[disabled],
.sistpec-btn-secondary[disabled] { opacity:.5; cursor:default; }

/* tabla */
.sistpec-table-wrapper { width:100%; overflow-x:auto; }
.sistpec-table { width:100%; border-collapse:collapse; font-size:13px; }
.sistpec-table thead { background:#7a061e; color:#fff; }
.sistpec-table th, .sistpec-table td { padding:8px 10px; border:1px solid #ddd; text-align:left; vertical-align:top; }
.sistpec-table tbody tr:nth-child(even) { background:#fafafa; }
.sin-resultados { text-align:center; color:#777; }

/* badges */
.badge { display:inline-block; padding:2px 8px; border-radius:10px; font-size:11px; font-weight:600; }
.badge--activo { background:#e1f3e1; color:#225522; }     /* Entregado */
.badge--pendiente { background:#e8f0ff; color:#1a4a9f; } /* Pendiente */

/* comentarios */
.comentarios-cell { min-width: 220px; }
.comentarios-wrap {
  border: 1px solid #e2e2e2;
  background: #ffffff;
  border-radius: 6px;
  padding: 6px 8px;
  line-height: 1.25;
}
.comentarios-text { display:block; color:#333; white-space:pre-wrap; word-break:break-word; }
.comentarios-vacio { color:#888; }

/* panel edición */
.sistpec-edit-panel { margin-top:20px; padding-top:10px; border-top:1px dashed #ccc; }
.sistpec-form { display:flex; flex-direction:column; gap:16px; }
.sistpec-form-row { display:grid; grid-template-columns: repeat(2, minmax(0, 1fr)); gap:16px; }
.sistpec-form-actions { display:flex; justify-content:flex-end; gap:8px; }

@media (max-width: 768px) {
  .sistpec-search-bar { grid-template-columns: 1fr; }
  .fechas-bar { grid-template-columns: 1fr; }
  .sistpec-form-row { grid-template-columns: 1fr; }
}
</style>

