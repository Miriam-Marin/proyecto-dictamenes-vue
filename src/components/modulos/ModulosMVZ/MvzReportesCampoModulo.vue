<template>
  <!-- Barra de acciones -->
  <section class="modulo-acciones">
    <span class="modulo-acciones-titulo">Administrar Reportes de Campo (MVZ)</span>

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

    <!-- ====================== 1) CAPTURAR HOJA ====================== -->
    <div v-if="selectedAction === 'capturar'">
      <h3 class="subtitulo">Capturar hoja de reporte de control de campo</h3>

      <div class="sistpec-info-box">
        <p class="sistpec-info-text">
          Capture la hoja. Posteriormente podrá capturar los <strong>aretes</strong> en la opción
          <strong>CAPTURAR ARETES</strong>.
        </p>
      </div>

      <form class="sistpec-form" @submit.prevent="guardarHoja">
        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Folio hoja control campo</label>
            <input v-model="formHoja.folio_hoja" type="text" placeholder="Ej. HCC-2025-010" />
          </div>

          <div class="sistpec-form-group">
            <label>Fecha de muestreo</label>
            <input v-model="formHoja.fecha_muestreo" type="date" />
          </div>

          <div class="sistpec-form-group">
            <label>Especie</label>
            <select v-model="formHoja.especie">
              <option value="" disabled>Seleccione</option>
              <option>Bovino</option>
              <option>Caprino</option>
              <option>Ovino</option>
              <option>Porcino</option>
              <option>Otro</option>
            </select>
          </div>

          <div class="sistpec-form-group">
            <label>Tipo de servicio</label>
            <select v-model="formHoja.tipo_servicio">
              <option value="" disabled>Seleccione</option>
              <option value="Campaña">Campaña</option>
              <option value="Particular">Particular</option>
            </select>
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Propietario</label>
            <select v-model="formHoja.propietario_id">
              <option value="" disabled>Seleccione</option>
              <option v-for="p in propietariosDisponibles" :key="p.id" :value="p.id">
                {{ nombreCompletoProp(p) }} ({{ p.estatus }})
              </option>
            </select>
          </div>

          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>UPP</label>
            <select v-model="formHoja.upp_id">
              <option value="" disabled>Seleccione</option>
              <option v-for="u in uppDisponibles" :key="u.id" :value="u.id">
                {{ u.clave_upp }} - {{ u.nombre_upp || 'Sin nombre' }}
              </option>
            </select>
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Municipio (UPP)</label>
            <input v-model="formHoja.municipio" type="text" placeholder="Ej. Ayahualulco" />
          </div>
          <div class="sistpec-form-group">
            <label>Localidad (UPP)</label>
            <input v-model="formHoja.localidad" type="text" placeholder="Ej. Los Altos" />
          </div>
          <div class="sistpec-form-group">
            <label>Estado (UPP)</label>
            <input v-model="formHoja.estado" type="text" placeholder="Ej. Veracruz" />
          </div>
          <div class="sistpec-form-group">
            <label>Código Postal</label>
            <input v-model="formHoja.codigo_postal" type="text" placeholder="Ej. 91260" />
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>MVZ (quien muestrea)</label>
            <input v-model="formHoja.mvz_nombre" type="text" placeholder="Ej. MVZ Juan Pérez" />
          </div>

          <div class="sistpec-form-group">
            <label>Tel. MVZ (opcional)</label>
            <input v-model="formHoja.mvz_telefono" type="text" placeholder="Ej. 228..." />
          </div>

          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Observaciones (opcional)</label>
            <input v-model="formHoja.observaciones" type="text" placeholder="Notas del muestreo" />
          </div>
        </div>

        <div class="sistpec-form-actions">
          <button type="submit" class="sistpec-btn-primary">GUARDAR HOJA</button>
          <button type="button" class="sistpec-btn-secondary" @click="limpiarFormHoja">LIMPIAR</button>
        </div>
      </form>

      <div class="sistpec-info-box" style="margin-top: 12px;">
        <p class="sistpec-info-text">
          Al guardar, la hoja queda en estatus <strong>Pendiente</strong> y aún puede editarse
          hasta que Recepción le asigne <strong>Número de Caso</strong>.
        </p>
      </div>
    </div>

    <!-- ====================== 2) CAPTURAR ARETES ====================== -->
    <div v-else-if="selectedAction === 'aretes'">
      <h3 class="subtitulo">Capturar números de aretes</h3>

      <div class="sistpec-info-box">
        <p class="sistpec-info-text">
          Seleccione una hoja y capture los aretes (uno por muestra). Puede agregar o quitar filas.
          La cantidad de aretes capturados será el total de muestras asociadas a la hoja.
        </p>
      </div>

      <div class="sistpec-search-bar fechas-bar">
        <div class="sistpec-form-group">
          <label>Buscar hoja (folio)</label>
          <input v-model="filtroAretesFolio" type="text" placeholder="Ej. HCC-2025-010" />
        </div>

        <div class="sistpec-form-group sistpec-search-actions">
          <button type="button" class="sistpec-btn-primary" @click="buscarHojasParaAretes">BUSCAR</button>
          <button type="button" class="sistpec-btn-secondary" @click="limpiarBusquedaAretes">LIMPIAR</button>
        </div>
      </div>

      <div v-if="buscadoAretes" class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Folio hoja</th>
              <th>Fecha</th>
              <th>UPP</th>
              <th>Especie</th>
              <th>Total aretes</th>
              <th>Número de caso</th>
              <th>Acciones</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="h in hojasParaAretes" :key="h.id">
              <td>{{ h.folio_hoja }}</td>
              <td>{{ h.fecha_muestreo }}</td>
              <td>{{ uppClave(h.upp_id) }}</td>
              <td>{{ h.especie }}</td>
              <td>{{ h.aretes.length }}</td>
              <td>{{ h.numero_caso || '-' }}</td>
              <td>
                <button
                  type="button"
                  class="sistpec-btn-secondary sistpec-btn-sm"
                  @click="seleccionarHojaAretes(h)"
                  :disabled="h.numero_caso_asignado"
                  :title="h.numero_caso_asignado ? 'Bloqueado: ya tiene número de caso.' : 'Capturar aretes'"
                >
                  SELECCIONAR
                </button>
              </td>
            </tr>

            <tr v-if="hojasParaAretes.length === 0">
              <td colspan="7" class="sin-resultados">No se encontraron hojas con ese criterio.</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- Panel captura aretes -->
      <div v-if="hojaAretesSeleccionada" class="sistpec-edit-panel">
        <h4 class="subtitulo-secundario">
          Hoja: {{ hojaAretesSeleccionada.folio_hoja }} | UPP: {{ uppClave(hojaAretesSeleccionada.upp_id) }}
        </h4>

        <div class="sistpec-form-actions" style="justify-content:flex-end;">
          <button type="button" class="sistpec-btn-secondary" @click="agregarFilaArete">+ Agregar arete</button>
        </div>

        <div class="sistpec-table-wrapper">
          <table class="sistpec-table">
            <thead>
              <tr>
                <th>#</th>
                <th>Número de arete</th>
                <th>Acciones</th>
              </tr>
            </thead>

            <tbody>
              <tr v-for="(a, idx) in hojaAretesSeleccionada.aretes" :key="a._key">
                <td>{{ idx + 1 }}</td>
                <td>
                  <input
                    v-model="hojaAretesSeleccionada.aretes[idx].arete"
                    type="text"
                    class="input-inline"
                    placeholder="Ej. 301152005..."
                  />
                </td>
                <td>
                  <button type="button" class="sistpec-btn-danger sistpec-btn-sm" @click="quitarFilaArete(idx)">
                    QUITAR
                  </button>
                </td>
              </tr>

              <tr v-if="hojaAretesSeleccionada.aretes.length === 0">
                <td colspan="3" class="sin-resultados">Agregue al menos un arete.</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="sistpec-info-box">
          <p class="sistpec-info-text">
            Total de aretes capturados: <strong>{{ hojaAretesSeleccionada.aretes.length }}</strong>
          </p>
        </div>

        <div class="sistpec-form-actions">
          <button type="button" class="sistpec-btn-primary" @click="guardarAretes">
            GUARDAR ARETES
          </button>
          <button type="button" class="sistpec-btn-secondary" @click="cancelarAretes">
            CANCELAR
          </button>
        </div>
      </div>
    </div>

    <!-- ====================== 3) CONSULTAR HOJAS ====================== -->
    <div v-else-if="selectedAction === 'consultar'">
      <h3 class="subtitulo">Consultar hojas de reporte</h3>

      <div class="sistpec-search-bar">
        <div class="sistpec-form-group">
          <label>Folio hoja</label>
          <input v-model="filtrosCons.folio_hoja" type="text" placeholder="Ej. HCC-2025-010" />
        </div>

        <div class="sistpec-form-group">
          <label>UPP (clave)</label>
          <input v-model="filtrosCons.upp" type="text" placeholder="Ej. 30-025..." />
        </div>

        <div class="sistpec-form-group">
          <label>Propietario (nombre)</label>
          <input v-model="filtrosCons.propietario" type="text" placeholder="Ej. Ruiz" />
        </div>

        <div class="sistpec-form-group">
          <label>Con número de caso</label>
          <select v-model="filtrosCons.con_caso">
            <option value="">Todos</option>
            <option value="si">Sí</option>
            <option value="no">No</option>
          </select>
        </div>
      </div>

      <div class="sistpec-search-bar fechas-bar">
        <div class="sistpec-form-group sistpec-form-group-inline">
          <label>Fecha muestreo</label>
          <div class="sistpec-form-inline-inputs">
            <input v-model="filtrosCons.fecha_inicio" type="date" />
            <span class="vigencia-sep">a</span>
            <input v-model="filtrosCons.fecha_fin" type="date" />
          </div>
        </div>

        <div class="sistpec-form-group sistpec-search-actions">
          <button type="button" class="sistpec-btn-primary" @click="buscarConsultar">BUSCAR</button>
          <button type="button" class="sistpec-btn-secondary" @click="limpiarConsultar">LIMPIAR</button>
        </div>
      </div>

      <div v-if="buscadoCons" class="sistpec-table-wrapper">
        <table class="sistpec-table">
          <thead>
            <tr>
              <th>Folio hoja</th>
              <th>Fecha</th>
              <th>Especie</th>
              <th>UPP</th>
              <th>Propietario</th>
              <th>Total aretes</th>
              <th>Número de caso</th>
              <th>Estatus edición</th>
              <th>Acciones</th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="h in hojasConsultadas" :key="h.id">
              <td>{{ h.folio_hoja }}</td>
              <td>{{ h.fecha_muestreo }}</td>
              <td>{{ h.especie }}</td>
              <td>{{ uppClave(h.upp_id) }}</td>
              <td>{{ propietarioNombre(h.propietario_id) }}</td>
              <td>{{ h.aretes.length }}</td>
              <td>{{ h.numero_caso || '-' }}</td>
              <td>
                <span class="badge" :class="h.numero_caso_asignado ? 'badge--inactivo' : 'badge--activo'">
                  {{ h.numero_caso_asignado ? 'BLOQUEADA' : 'EDITABLE' }}
                </span>
              </td>
              <td>
                <div class="acciones">
                  <button type="button" class="sistpec-btn-secondary sistpec-btn-sm" @click="verDetalle(h)">
                    VER
                  </button>

                  <button
                    type="button"
                    class="sistpec-btn-secondary sistpec-btn-sm"
                    :disabled="h.numero_caso_asignado"
                    @click="abrirEdicionHoja(h)"
                    title="Solo antes de asignar número de caso."
                  >
                    EDITAR
                  </button>
                </div>
              </td>
            </tr>

            <tr v-if="hojasConsultadas.length === 0">
              <td colspan="9" class="sin-resultados">No se encontraron hojas con esos criterios.</td>
            </tr>
          </tbody>
        </table>
      </div>

      <!-- detalle -->
      <div v-if="detalleHoja" class="sistpec-edit-panel">
        <h4 class="subtitulo-secundario">Detalle: {{ detalleHoja.folio_hoja }}</h4>

        <div class="detalle-grid">
          <div><span class="lbl">Propietario:</span> {{ propietarioNombre(detalleHoja.propietario_id) }}</div>
          <div><span class="lbl">UPP:</span> {{ uppClave(detalleHoja.upp_id) }}</div>
          <div><span class="lbl">Fecha:</span> {{ detalleHoja.fecha_muestreo }}</div>
          <div><span class="lbl">Especie:</span> {{ detalleHoja.especie }}</div>
          <div><span class="lbl">Servicio:</span> {{ detalleHoja.tipo_servicio }}</div>
          <div><span class="lbl">MVZ:</span> {{ detalleHoja.mvz_nombre || '-' }}</div>
          <div style="grid-column: span 2;"><span class="lbl">Obs:</span> {{ detalleHoja.observaciones || '-' }}</div>
          <div style="grid-column: span 2;">
            <span class="lbl">Aretes ({{ detalleHoja.aretes.length }}):</span>
            <span>{{ detalleHoja.aretes.map(x => x.arete).filter(Boolean).join(', ') || '-' }}</span>
          </div>
        </div>

        <div class="sistpec-form-actions">
          <button type="button" class="sistpec-btn-secondary" @click="detalleHoja = null">CERRAR</button>
        </div>
      </div>
    </div>

    <!-- ====================== 4) EDITAR HOJA (ANTES DE CASO) ====================== -->
    <div v-else-if="selectedAction === 'editar'">
      <h3 class="subtitulo">Editar hoja de reporte (solo antes de asignar número de caso)</h3>

      <div class="sistpec-info-box">
        <p class="sistpec-info-text">
          Si la hoja ya tiene <strong>Número de Caso</strong>, queda bloqueada para edición.
        </p>
      </div>

      <div v-if="!hojaEditando" class="modulo-alert modulo-alert--error">
        Seleccione una hoja desde <strong>CONSULTAR</strong> para editar.
      </div>

      <form v-else class="sistpec-form" @submit.prevent="guardarEdicionHoja">
        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Folio hoja</label>
            <input v-model="hojaEditando.folio_hoja" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Fecha de muestreo</label>
            <input v-model="hojaEditando.fecha_muestreo" type="date" />
          </div>
          <div class="sistpec-form-group">
            <label>Especie</label>
            <select v-model="hojaEditando.especie">
              <option value="" disabled>Seleccione</option>
              <option>Bovino</option>
              <option>Caprino</option>
              <option>Ovino</option>
              <option>Porcino</option>
              <option>Otro</option>
            </select>
          </div>
          <div class="sistpec-form-group">
            <label>Tipo de servicio</label>
            <select v-model="hojaEditando.tipo_servicio">
              <option value="" disabled>Seleccione</option>
              <option value="Campaña">Campaña</option>
              <option value="Particular">Particular</option>
            </select>
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Propietario</label>
            <select v-model="hojaEditando.propietario_id">
              <option value="" disabled>Seleccione</option>
              <option v-for="p in propietariosDisponibles" :key="p.id" :value="p.id">
                {{ nombreCompletoProp(p) }} ({{ p.estatus }})
              </option>
            </select>
          </div>

          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>UPP</label>
            <select v-model="hojaEditando.upp_id">
              <option value="" disabled>Seleccione</option>
              <option v-for="u in uppDisponibles" :key="u.id" :value="u.id">
                {{ u.clave_upp }} - {{ u.nombre_upp || 'Sin nombre' }}
              </option>
            </select>
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>Municipio</label>
            <input v-model="hojaEditando.municipio" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Localidad</label>
            <input v-model="hojaEditando.localidad" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Estado</label>
            <input v-model="hojaEditando.estado" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Código Postal</label>
            <input v-model="hojaEditando.codigo_postal" type="text" />
          </div>
        </div>

        <div class="sistpec-form-row">
          <div class="sistpec-form-group">
            <label>MVZ</label>
            <input v-model="hojaEditando.mvz_nombre" type="text" />
          </div>
          <div class="sistpec-form-group">
            <label>Tel. MVZ</label>
            <input v-model="hojaEditando.mvz_telefono" type="text" />
          </div>
          <div class="sistpec-form-group" style="grid-column: span 2;">
            <label>Observaciones</label>
            <input v-model="hojaEditando.observaciones" type="text" />
          </div>
        </div>

        <div class="sistpec-form-actions">
          <button type="submit" class="sistpec-btn-primary">GUARDAR CAMBIOS</button>
          <button type="button" class="sistpec-btn-secondary" @click="cancelarEdicionHoja">CANCELAR</button>
        </div>
      </form>
    </div>

    <!-- fallback -->
    <div v-else>
      <h3 class="subtitulo">Acción no disponible</h3>
      <p>El contenido para esta acción aún está en desarrollo.</p>
    </div>
  </section>
</template>

<script setup>
import { computed, nextTick, ref, watch } from 'vue';

defineProps({
  codigo: { type: String, required: false, default: '' },
  rol: { type: String, required: false, default: 'MVZ' }
});

/* ===================== Scroll ===================== */
const moduloContenidoRef = ref(null);
function scrollAlContenido() {
  nextTick(() => {
    if (!moduloContenidoRef.value) return;
    const rect = moduloContenidoRef.value.getBoundingClientRect();
    const offset = 90;
    const top = rect.top + window.scrollY - offset;
    window.scrollTo({ top, behavior: 'smooth' });
  });
}

/* ===================== Acciones ===================== */
const acciones = [
  { id: 'capturar', label: 'CAPTURAR HOJA' },
  { id: 'aretes', label: 'CAPTURAR ARETES' },
  { id: 'consultar', label: 'CONSULTAR HOJAS' },
  { id: 'editar', label: 'EDITAR HOJA' }
];

const selectedAction = ref('capturar');
const errores = ref([]);
const mensajeExito = ref('');

function cambiarAccion(id) {
  selectedAction.value = id;
  scrollAlContenido();
}

const descripcionAccionActual = computed(() => {
  switch (selectedAction.value) {
    case 'capturar':
      return 'Capture los datos generales de la hoja de control de campo.';
    case 'aretes':
      return 'Seleccione una hoja sin número de caso y capture los aretes.';
    case 'consultar':
      return 'Consulte hojas por folio, UPP, propietario y fechas.';
    case 'editar':
      return 'Edite hoja solo si NO tiene número de caso asignado.';
    default:
      return '';
  }
});

watch(
  () => selectedAction.value,
  () => {
    errores.value = [];
    mensajeExito.value = '';
  }
);

/* ===================== DEMO: MVZ actual ===================== */
const mvzUserId = 10;

/* ===================== DEMO DATA: Propietarios (solo del MVZ) ===================== */
const propietariosDemo = ref([
  { id: 1, mvz_user_id: 10, apellido_paterno: 'Ruiz', apellido_materno: 'Mendoza', nombres: 'Miguel Ángel', estatus: 'Activo' },
  { id: 2, mvz_user_id: 10, apellido_paterno: 'García', apellido_materno: 'López', nombres: 'Juan', estatus: 'Finado' },
  { id: 3, mvz_user_id: 99, apellido_paterno: 'Otro', apellido_materno: 'MVZ', nombres: 'Ajeno', estatus: 'Activo' }
]);

const propietariosDisponibles = computed(() => propietariosDemo.value.filter(p => p.mvz_user_id === mvzUserId));

function nombreCompletoProp(p) {
  return `${p.apellido_paterno || ''} ${p.apellido_materno || ''} ${p.nombres || ''}`.trim();
}
function propietarioNombre(id) {
  const p = propietariosDemo.value.find(x => x.id === id);
  return p ? nombreCompletoProp(p) : '—';
}

/* ===================== DEMO DATA: UPP (solo del MVZ) ===================== */
const uppDemo = ref([
  { id: 101, mvz_user_id: 10, propietario_id: 1, clave_upp: '30-025-2000-001', nombre_upp: 'El Encino' },
  { id: 102, mvz_user_id: 10, propietario_id: 2, clave_upp: '30-025-1055-001', nombre_upp: 'San Francisco' },
  { id: 103, mvz_user_id: 99, propietario_id: 99, clave_upp: 'XX-XXX-XXXX-XXX', nombre_upp: 'Ajena' }
]);

const uppDisponibles = computed(() => uppDemo.value.filter(u => u.mvz_user_id === mvzUserId));

function uppClave(uppId) {
  const u = uppDemo.value.find(x => x.id === uppId);
  return u ? u.clave_upp : '—';
}

/* ===================== DEMO DATA: hojas control campo (del MVZ) ===================== */
const hojasDemo = ref([
  {
    id: 5001,
    mvz_user_id: 10,
    folio_hoja: 'HCC-2025-010',
    fecha_muestreo: '2025-12-14',
    especie: 'Bovino',
    tipo_servicio: 'Campaña',
    propietario_id: 1,
    upp_id: 101,
    municipio: 'Ayahualulco',
    localidad: 'Los Altos',
    estado: 'Veracruz',
    codigo_postal: '91260',
    mvz_nombre: 'MVZ Juan Pérez',
    mvz_telefono: '',
    observaciones: '',
    numero_caso_asignado: false,
    numero_caso: '',
    aretes: [
      { _key: 'a1', arete: '301152005' },
      { _key: 'a2', arete: '301152006' }
    ]
  },
  {
    id: 5002,
    mvz_user_id: 10,
    folio_hoja: 'HCC-2025-020',
    fecha_muestreo: '2025-12-10',
    especie: 'Bovino',
    tipo_servicio: 'Campaña',
    propietario_id: 1,
    upp_id: 101,
    municipio: 'Ayahualulco',
    localidad: 'Los Altos',
    estado: 'Veracruz',
    codigo_postal: '91260',
    mvz_nombre: 'MVZ Juan Pérez',
    mvz_telefono: '',
    observaciones: '',
    numero_caso_asignado: true,
    numero_caso: 'BR25-001',
    aretes: [{ _key: 'b1', arete: '301152010' }]
  }
]);

/* ===================== 1) Capturar hoja ===================== */
const formHoja = ref({
  folio_hoja: '',
  fecha_muestreo: '',
  especie: '',
  tipo_servicio: '',
  propietario_id: '',
  upp_id: '',
  municipio: '',
  localidad: '',
  estado: '',
  codigo_postal: '',
  mvz_nombre: '',
  mvz_telefono: '',
  observaciones: ''
});

function limpiarFormHoja() {
  formHoja.value = {
    folio_hoja: '',
    fecha_muestreo: '',
    especie: '',
    tipo_servicio: '',
    propietario_id: '',
    upp_id: '',
    municipio: '',
    localidad: '',
    estado: '',
    codigo_postal: '',
    mvz_nombre: '',
    mvz_telefono: '',
    observaciones: ''
  };
}

function guardarHoja() {
  errores.value = [];
  mensajeExito.value = '';

  const f = formHoja.value;

  if (!String(f.folio_hoja || '').trim()) errores.value.push('Capture el folio de hoja de control de campo.');
  if (!f.fecha_muestreo) errores.value.push('Seleccione la fecha de muestreo.');
  if (!String(f.especie || '').trim()) errores.value.push('Seleccione la especie.');
  if (!String(f.tipo_servicio || '').trim()) errores.value.push('Seleccione el tipo de servicio.');
  if (!String(f.propietario_id || '').trim()) errores.value.push('Seleccione el propietario.');
  if (!String(f.upp_id || '').trim()) errores.value.push('Seleccione la UPP.');

  if (errores.value.length) return;

  const folio = String(f.folio_hoja).trim();
  const existe = hojasDemo.value.some(h => h.mvz_user_id === mvzUserId && String(h.folio_hoja || '').trim().toLowerCase() === folio.toLowerCase());
  if (existe) return errores.value.push('Ese folio de hoja ya existe (registrado por usted).');

  hojasDemo.value.push({
    id: Date.now(),
    mvz_user_id: mvzUserId,
    folio_hoja: folio,
    fecha_muestreo: f.fecha_muestreo,
    especie: f.especie,
    tipo_servicio: f.tipo_servicio,
    propietario_id: Number(f.propietario_id),
    upp_id: Number(f.upp_id),
    municipio: String(f.municipio || '').trim(),
    localidad: String(f.localidad || '').trim(),
    estado: String(f.estado || '').trim(),
    codigo_postal: String(f.codigo_postal || '').trim(),
    mvz_nombre: String(f.mvz_nombre || '').trim(),
    mvz_telefono: String(f.mvz_telefono || '').trim(),
    observaciones: String(f.observaciones || '').trim(),
    numero_caso_asignado: false,
    numero_caso: '',
    aretes: []
  });

  mensajeExito.value = 'Hoja registrada (DEMO). Ahora puede capturar aretes.';
  limpiarFormHoja();
}

/* ===================== 2) Capturar aretes ===================== */
const filtroAretesFolio = ref('');
const buscadoAretes = ref(false);

function buscarHojasParaAretes() {
  buscadoAretes.value = true;
  errores.value = [];
  mensajeExito.value = '';
}

function limpiarBusquedaAretes() {
  filtroAretesFolio.value = '';
  buscadoAretes.value = false;
  hojaAretesSeleccionada.value = null;
  errores.value = [];
  mensajeExito.value = '';
}

const hojasParaAretes = computed(() => {
  if (!buscadoAretes.value) return [];
  const folio = filtroAretesFolio.value.trim().toLowerCase();

  return hojasDemo.value
    .filter(h => h.mvz_user_id === mvzUserId)
    .filter(h => (folio ? (h.folio_hoja || '').toLowerCase().includes(folio) : true));
});

const hojaAretesSeleccionada = ref(null);

function seleccionarHojaAretes(h) {
  errores.value = [];
  mensajeExito.value = '';
  if (h.numero_caso_asignado) return errores.value.push('La hoja ya tiene número de caso; no se pueden capturar aretes.');
  hojaAretesSeleccionada.value = JSON.parse(JSON.stringify(h));
}

function agregarFilaArete() {
  if (!hojaAretesSeleccionada.value) return;
  hojaAretesSeleccionada.value.aretes.push({
    _key: crypto?.randomUUID ? crypto.randomUUID() : String(Date.now() + Math.random()),
    arete: ''
  });
}

function quitarFilaArete(idx) {
  if (!hojaAretesSeleccionada.value) return;
  hojaAretesSeleccionada.value.aretes.splice(idx, 1);
}

function guardarAretes() {
  errores.value = [];
  mensajeExito.value = '';
  if (!hojaAretesSeleccionada.value) return errores.value.push('Seleccione una hoja.');

  const vacios = hojaAretesSeleccionada.value.aretes.some(a => !String(a.arete || '').trim());
  if (hojaAretesSeleccionada.value.aretes.length === 0) errores.value.push('Agregue al menos un arete.');
  if (vacios) errores.value.push('Hay aretes vacíos. Capture todos los aretes.');
  if (errores.value.length) return;

  // evitar duplicados dentro de la misma hoja
  const list = hojaAretesSeleccionada.value.aretes.map(a => String(a.arete).trim());
  const set = new Set(list.map(x => x.toLowerCase()));
  if (set.size !== list.length) return errores.value.push('Hay aretes duplicados dentro de la hoja.');

  const idx = hojasDemo.value.findIndex(h => h.id === hojaAretesSeleccionada.value.id);
  if (idx === -1) return errores.value.push('No se encontró la hoja para actualizar.');

  hojasDemo.value[idx] = { ...hojasDemo.value[idx], aretes: hojaAretesSeleccionada.value.aretes };
  mensajeExito.value = 'Aretes guardados (DEMO).';
  hojaAretesSeleccionada.value = null;
}

function cancelarAretes() {
  hojaAretesSeleccionada.value = null;
}

/* ===================== 3) Consultar hojas ===================== */
const filtrosCons = ref({
  folio_hoja: '',
  upp: '',
  propietario: '',
  con_caso: '',
  fecha_inicio: '',
  fecha_fin: ''
});

const buscadoCons = ref(false);
const detalleHoja = ref(null);

function buscarConsultar() {
  buscadoCons.value = true;
  errores.value = [];
  mensajeExito.value = '';
}

function limpiarConsultar() {
  filtrosCons.value = { folio_hoja: '', upp: '', propietario: '', con_caso: '', fecha_inicio: '', fecha_fin: '' };
  buscadoCons.value = false;
  detalleHoja.value = null;
  errores.value = [];
  mensajeExito.value = '';
}

function propietarioTexto(id) {
  return propietarioNombre(id).toLowerCase();
}

const hojasConsultadas = computed(() => {
  if (!buscadoCons.value) return [];
  const f = filtrosCons.value;

  const folio = f.folio_hoja.trim().toLowerCase();
  const upp = f.upp.trim().toLowerCase();
  const prop = f.propietario.trim().toLowerCase();
  const conCaso = f.con_caso;
  const ini = f.fecha_inicio;
  const fin = f.fecha_fin;

  return hojasDemo.value
    .filter(h => h.mvz_user_id === mvzUserId)
    .filter(h => (folio ? (h.folio_hoja || '').toLowerCase().includes(folio) : true))
    .filter(h => (upp ? uppClave(h.upp_id).toLowerCase().includes(upp) : true))
    .filter(h => (prop ? propietarioTexto(h.propietario_id).includes(prop) : true))
    .filter(h => {
      if (!conCaso) return true;
      return conCaso === 'si' ? h.numero_caso_asignado : !h.numero_caso_asignado;
    })
    .filter(h => {
      let ok = true;
      if (ini) ok = ok && h.fecha_muestreo >= ini;
      if (fin) ok = ok && h.fecha_muestreo <= fin;
      return ok;
    });
});

function verDetalle(h) {
  detalleHoja.value = JSON.parse(JSON.stringify(h));
}

/* ===================== 4) Editar hoja ===================== */
const hojaEditando = ref(null);

function abrirEdicionHoja(h) {
  errores.value = [];
  mensajeExito.value = '';
  if (h.numero_caso_asignado) return errores.value.push('La hoja ya tiene número de caso. No es editable.');
  hojaEditando.value = JSON.parse(JSON.stringify(h));
  selectedAction.value = 'editar';
}

function cancelarEdicionHoja() {
  hojaEditando.value = null;
  selectedAction.value = 'consultar';
}

function guardarEdicionHoja() {
  errores.value = [];
  mensajeExito.value = '';
  if (!hojaEditando.value) return errores.value.push('No hay hoja seleccionada.');

  if (hojaEditando.value.numero_caso_asignado) return errores.value.push('La hoja ya tiene número de caso. No es editable.');

  if (!String(hojaEditando.value.folio_hoja || '').trim()) errores.value.push('El folio es obligatorio.');
  if (!hojaEditando.value.fecha_muestreo) errores.value.push('La fecha es obligatoria.');
  if (!String(hojaEditando.value.especie || '').trim()) errores.value.push('La especie es obligatoria.');
  if (!String(hojaEditando.value.tipo_servicio || '').trim()) errores.value.push('El tipo de servicio es obligatorio.');
  if (!String(hojaEditando.value.propietario_id || '').trim()) errores.value.push('Seleccione propietario.');
  if (!String(hojaEditando.value.upp_id || '').trim()) errores.value.push('Seleccione UPP.');
  if (errores.value.length) return;

  // evitar duplicar folio con otro registro del mismo MVZ
  const folio = String(hojaEditando.value.folio_hoja).trim().toLowerCase();
  const existe = hojasDemo.value.some(h =>
    h.mvz_user_id === mvzUserId &&
    h.id !== hojaEditando.value.id &&
    String(h.folio_hoja || '').trim().toLowerCase() === folio
  );
  if (existe) return errores.value.push('Ya existe otra hoja con ese folio (registrado por usted).');

  const idx = hojasDemo.value.findIndex(x => x.id === hojaEditando.value.id);
  if (idx === -1) return errores.value.push('No se encontró la hoja para actualizar.');

  hojasDemo.value[idx] = { ...hojasDemo.value[idx], ...hojaEditando.value, propietario_id: Number(hojaEditando.value.propietario_id), upp_id: Number(hojaEditando.value.upp_id) };
  mensajeExito.value = 'Hoja actualizada (DEMO).';
  hojaEditando.value = null;
  selectedAction.value = 'consultar';
}
</script>

<style scoped>
/* Header acciones */
.modulo-acciones { margin-bottom: 20px; }
.modulo-acciones-titulo { display:block; font-size:14px; margin-bottom:8px; color:#333; font-weight:900; }
.modulo-acciones-botones { display:flex; flex-wrap:wrap; gap:6px; }

/* Botones */
.sistpec-btn-accion{
  border:none; padding:8px 14px; font-size:12px; font-weight:900;
  text-transform:uppercase; border-radius:4px; cursor:pointer;
  background:#2f6b32; color:#fff; letter-spacing:0.4px;
}
.sistpec-btn-accion.active{ background:#244e26; }

/* Contenido */
.modulo-contenido { margin-top: 10px; }
.subtitulo { font-size:18px; margin:10px 0 15px; color:#333; }
.subtitulo-secundario { font-size:16px; margin:16px 0 10px; color:#333; }

/* Info */
.sistpec-info-box{
  margin-top:10px; padding:10px 14px; border-radius:4px;
  background:#e1f3e1; border:1px solid #c3e6c3;
}
.sistpec-info-text{ margin:0; font-size:13px; color:#225522; }

/* Alertas */
.modulo-alert{ margin-bottom:12px; padding:10px 14px; border-radius:4px; font-size:13px; }
.modulo-alert--error{ background:#fbeaea; border:1px solid #f5c2c2; color:#7a1f1f; }
.modulo-alert--success{ background:#e1f3e1; border:1px solid #c3e6c3; color:#225522; }

/* Form */
.sistpec-form{ display:flex; flex-direction:column; gap:16px; }
.sistpec-form-row{ display:grid; grid-template-columns: repeat(4, minmax(0, 1fr)); gap:12px; }

.sistpec-form-group{ display:flex; flex-direction:column; gap:4px; }
.sistpec-form-group label{ font-size:13px; font-weight:900; color:#444; }
.sistpec-form-group input,
.sistpec-form-group select{
  padding:8px 10px; border-radius:4px; border:1px solid #ccc;
  font-size:14px; outline:none;
}
.sistpec-form-group input:focus,
.sistpec-form-group select:focus{
  border-color:#2f6b32;
  box-shadow:0 0 0 1px rgba(47, 107, 50, 0.15);
}

/* Actions */
.sistpec-form-actions{
  display:flex; justify-content:flex-end; gap:8px;
}

.sistpec-btn-primary{
  background:#2f6b32; color:#fff; border:none;
  padding:8px 18px; border-radius:4px;
  font-size:13px; font-weight:900; cursor:pointer;
}
.sistpec-btn-primary:hover{ background:#244e26; }
.sistpec-btn-primary:disabled{ opacity:.55; cursor:not-allowed; }

.sistpec-btn-secondary{
  background:#e0e0e0; color:#333; border:none;
  padding:8px 18px; border-radius:4px;
  font-size:13px; font-weight:900; cursor:pointer;
}
.sistpec-btn-secondary:hover{ background:#d0d0d0; }

.sistpec-btn-danger{
  background:#7a061e; color:#fff; border:none;
  padding:6px 14px; border-radius:4px;
  font-size:12px; font-weight:900; cursor:pointer;
}
.sistpec-btn-danger:hover{ background:#5a0416; }

.sistpec-btn-sm{ padding:5px 10px; font-size:11px; }

/* filtros grid */
.sistpec-search-bar{
  display:grid; grid-template-columns: repeat(4, minmax(0, 1fr));
  gap:12px; margin-bottom:16px;
}
.fechas-bar{ grid-template-columns: repeat(2, minmax(0, 1fr)); }
.sistpec-search-actions{
  display:flex; align-items:flex-end; gap:8px; justify-content:flex-end;
}

/* tabla */
.sistpec-table-wrapper{ width:100%; overflow-x:auto; }
.sistpec-table{ width:100%; border-collapse:collapse; font-size:13px; }
.sistpec-table thead{ background:#7a061e; color:#fff; }
.sistpec-table th, .sistpec-table td{ padding:8px 10px; border:1px solid #ddd; text-align:left; }
.sistpec-table tbody tr:nth-child(even){ background:#fafafa; }
.sin-resultados{ text-align:center; color:#777; }

.sistpec-edit-panel{
  margin-top:20px; padding-top:10px; border-top:1px dashed #ccc;
}

.badge{ display:inline-block; padding:2px 8px; border-radius:10px; font-size:11px; font-weight:900; }
.badge--activo{ background:#e1f3e1; color:#225522; border:1px solid #c3e6c3; }
.badge--inactivo{ background:#fbeaea; color:#7a1f1f; border:1px solid #f5c2c2; }

.detalle-grid{
  display:grid;
  grid-template-columns: repeat(2, minmax(0, 1fr));
  gap:8px 12px;
  font-size:13px;
}
.lbl{ font-weight:900; color:#444; }

.input-inline{
  width:100%;
  padding:8px 10px;
  border-radius:4px;
  border:1px solid #ccc;
  font-size:14px;
  outline:none;
}

.vigencia-sep{ font-size:14px; color:#666; }

.acciones{ display:flex; gap:6px; flex-wrap:wrap; }

/* responsive */
@media (max-width: 768px) {
  .sistpec-search-bar { grid-template-columns: 1fr; }
  .fechas-bar { grid-template-columns: 1fr; }
  .sistpec-form-row { grid-template-columns: 1fr; }
  .detalle-grid { grid-template-columns: 1fr; }
}
</style>
