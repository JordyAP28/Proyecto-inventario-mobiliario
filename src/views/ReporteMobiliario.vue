<template>
  <div class="reporte-mobiliario">
    <div class="row">
      <div class="col-12">
        <h1 class="titulo-principal">Reporte de Daño Mobiliario</h1>
        <hr>
      </div>
    </div>

    <div class="card shadow-sm">
      <div class="card-header bg-danger text-white">
        <h5 class="mb-0">
          <i class="fas fa-exclamation-triangle me-2"></i>
          Nuevo Reporte de Daño
        </h5>
      </div>
      <div class="card-body">
        <form @submit.prevent="handleSubmit">
          <div class="row">
            <!-- Código del mobiliario -->
            <div class="col-md-6 mb-3">
              <label for="codigoMobiliario" class="form-label">
                Código del Mobiliario <span class="text-danger">*</span>
              </label>
              <div class="input-group">
                <input 
                  type="text" 
                  class="form-control" 
                  id="codigoMobiliario"
                  v-model="form.codigoMobiliario"
                  :class="{ 'is-invalid': errors.codigoMobiliario }"
                  placeholder="ULEAM-XXXX"
                  @blur="buscarMobiliario"
                  @input="clearMobiliarioData"
                />
                <button 
                  class="btn btn-outline-secondary" 
                  type="button"
                  @click="buscarMobiliario"
                  :disabled="!form.codigoMobiliario.trim()"
                >
                  <i class="fas fa-search"></i>
                </button>
                <div v-if="errors.codigoMobiliario" class="invalid-feedback">
                  {{ errors.codigoMobiliario }}
                </div>
              </div>
              
              <!-- Información del mobiliario encontrado -->
              <div v-if="mobiliarioEncontrado" class="mt-2 p-2 bg-light rounded">
                <small class="text-success">
                  <i class="fas fa-check-circle me-1"></i>
                  <strong>{{ mobiliarioEncontrado.tipo }}</strong> - 
                  {{ mobiliarioEncontrado.ubicacion.edificio }}, {{ mobiliarioEncontrado.ubicacion.aula }}
                </small>
              </div>
            </div>

            <!-- Tipo de daño -->
            <div class="col-md-6 mb-3">
              <label for="tipoDano" class="form-label">
                Tipo de Daño <span class="text-danger">*</span>
              </label>
              <select 
                class="form-select" 
                id="tipoDano"
                v-model="form.tipoDano"
                :class="{ 'is-invalid': errors.tipoDano }"
              >
                <option value="" disabled>Seleccione...</option>
                <option value="estructura">Daño estructural</option>
                <option value="superficie">Daño superficial</option>
                <option value="funcional">Falla funcional</option>
                <option value="desgaste">Desgaste normal</option>
                <option value="vandalismo">Vandalismo</option>
                <option value="otros">Otros</option>
              </select>
              <div v-if="errors.tipoDano" class="invalid-feedback">
                {{ errors.tipoDano }}
              </div>
            </div>
          </div>

          <!-- Descripción del daño -->
          <div class="mb-3">
            <label for="descripcion" class="form-label">
              Descripción del Daño <span class="text-danger">*</span>
            </label>
            <textarea 
              class="form-control" 
              id="descripcion" 
              rows="4"
              v-model="form.descripcion"
              :class="{ 'is-invalid': errors.descripcion }"
              placeholder="Describa detalladamente el daño encontrado..."
              maxlength="500"
            ></textarea>
            <div class="d-flex justify-content-between">
              <div v-if="errors.descripcion" class="invalid-feedback d-block">
                {{ errors.descripcion }}
              </div>
              <small class="text-muted">{{ form.descripcion.length }}/500 caracteres</small>
            </div>
          </div>

          <div class="row">
            <!-- Ubicación actual -->
            <div class="col-md-6 mb-3">
              <label for="ubicacion" class="form-label">Ubicación Actual</label>
              <input 
                type="text" 
                class="form-control" 
                id="ubicacion"
                v-model="form.ubicacion"
                :readonly="!!mobiliarioEncontrado"
                placeholder="Edificio, Aula"
              />
            </div>

            <!-- Fecha de reporte -->
            <div class="col-md-6 mb-3">
              <label for="fechaReporte" class="form-label">
                Fecha de Reporte <span class="text-danger">*</span>
              </label>
              <input 
                type="date" 
                class="form-control" 
                id="fechaReporte"
                v-model="form.fechaReporte"
                :class="{ 'is-invalid': errors.fechaReporte }"
                :max="today"
              />
              <div v-if="errors.fechaReporte" class="invalid-feedback">
                {{ errors.fechaReporte }}
              </div>
            </div>
          </div>

          <div class="row">
            <!-- Nivel de urgencia -->
            <div class="col-md-6 mb-3">
              <label for="nivelUrgencia" class="form-label">
                Nivel de Urgencia <span class="text-danger">*</span>
              </label>
              <select 
                class="form-select" 
                id="nivelUrgencia"
                v-model="form.nivelUrgencia"
                :class="{ 'is-invalid': errors.nivelUrgencia }"
              >
                <option value="" disabled>Seleccione...</option>
                <option value="bajo">Bajo - Puede esperar</option>
                <option value="medio">Medio - Revisar pronto</option>
                <option value="alto">Alto - Requiere atención inmediata</option>
                <option value="critico">Crítico - Peligro para usuarios</option>
              </select>
              <div v-if="errors.nivelUrgencia" class="invalid-feedback">
                {{ errors.nivelUrgencia }}
              </div>
            </div>

            <!-- Reportado por -->
            <div class="col-md-6 mb-3">
              <label for="reportadoPor" class="form-label">Reportado por</label>
              <input 
                type="text" 
                class="form-control" 
                id="reportadoPor"
                v-model="form.reportadoPor"
                :readonly="true"
                :value="user?.fullName || 'Usuario actual'"
              />
            </div>
          </div>

          <!-- Evidencia fotográfica -->
          <div class="mb-3">
            <label for="imagenDano" class="form-label">Evidencia Fotográfica</label>
            <input 
              type="file" 
              class="form-control" 
              id="imagenDano"
              @change="handleFileUpload"
              accept="image/*"
              multiple
            />
            <div class="form-text">
              Suba imágenes que muestren el daño (máx. 3 imágenes, 2MB cada una)
            </div>
            <div v-if="errors.imagenes" class="text-danger small mt-1">
              {{ errors.imagenes }}
            </div>
          </div>

          <!-- Preview de imágenes -->
          <div v-if="imagePreviews.length > 0" class="mb-3">
            <label class="form-label">Imágenes seleccionadas:</label>
            <div class="row">
              <div 
                v-for="(preview, index) in imagePreviews" 
                :key="index"
                class="col-md-4 mb-2"
              >
                <div class="position-relative">
                  <img 
                    :src="preview.url" 
                    class="img-thumbnail w-100" 
                    style="height: 150px; object-fit: cover;"
                    :alt="`Evidencia ${index + 1}`"
                  />
                  <button 
                    type="button" 
                    class="btn btn-danger btn-sm position-absolute top-0 end-0 m-1"
                    @click="removeImage(index)"
                  >
                    <i class="fas fa-times"></i>
                  </button>
                  <small class="d-block text-center mt-1">{{ preview.name }}</small>
                </div>
              </div>
            </div>
          </div>

          <!-- Acciones adicionales -->
          <div class="mb-3">
            <label for="accionesRecomendadas" class="form-label">Acciones Recomendadas</label>
            <div class="form-check">
              <input 
                class="form-check-input" 
                type="checkbox" 
                id="requiereRemplazo" 
                v-model="form.requiereRemplazo"
              />
              <label class="form-check-label" for="requiereRemplazo">
                Requiere reemplazo inmediato
              </label>
            </div>
            <div class="form-check">
              <input 
                class="form-check-input" 
                type="checkbox" 
                id="peligroUsuarios" 
                v-model="form.peligroUsuarios"
              />
              <label class="form-check-label" for="peligroUsuarios">
                Representa peligro para los usuarios
              </label>
            </div>
            <div class="form-check">
              <input 
                class="form-check-input" 
                type="checkbox" 
                id="retirarlUso" 
                v-model="form.retirarUso"
              />
              <label class="form-check-label" for="retirarlUso">
                Retirar del uso inmediatamente
              </label>
            </div>
          </div>

          <!-- Comentarios adicionales -->
          <div class="mb-3">
            <label for="comentarios" class="form-label">Comentarios Adicionales</label>
            <textarea 
              class="form-control" 
              id="comentarios" 
              rows="3"
              v-model="form.comentarios"
              placeholder="Información adicional que considere relevante..."
              maxlength="300"
            ></textarea>
            <small class="text-muted">{{ form.comentarios.length }}/300 caracteres</small>
          </div>

          <!-- Botones -->
          <div class="d-grid gap-2 d-md-flex justify-content-md-end">
            <button 
              type="button" 
              class="btn btn-secondary me-md-2" 
              @click="handleCancel"
              :disabled="loading"
            >
              <i class="fas fa-times me-2"></i>Cancelar
            </button>
            <button 
              type="submit" 
              class="btn btn-danger"
              :disabled="loading"
            >
              <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
              <i v-else class="fas fa-paper-plane me-2"></i>
              {{ loading ? 'Enviando...' : 'Reportar Daño' }}
            </button>
          </div>
        </form>
      </div>
    </div>

    <!-- Modal de confirmación mejorado -->
    <div 
      class="modal fade" 
      :class="{ show: showSuccessModal }"
      :style="{ display: showSuccessModal ? 'block' : 'none' }"
      tabindex="-1"
    >
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content border-0 shadow">
          <div class="modal-header bg-success text-white text-center">
            <div class="w-100">
              <div class="modal-icon mb-2">
                <i class="fas fa-check-circle fa-3x"></i>
              </div>
              <h4 class="modal-title mb-0">¡Reporte Enviado Exitosamente!</h4>
            </div>
          </div>
          <div class="modal-body text-center py-4" v-if="lastReport">
            <div class="success-animation mb-3">
              <div class="checkmark-circle">
                <div class="background"></div>
                <div class="checkmark draw"></div>
              </div>
            </div>
            
            <h5 class="text-success mb-3">El daño ha sido reportado correctamente</h5>
            
            <div class="alert alert-success text-start">
              <div class="row">
                <div class="col-12 mb-2">
                  <strong><i class="fas fa-barcode me-2"></i>Código de reporte:</strong> 
                  <span class="badge bg-primary ms-2">{{ lastReport.codigo }}</span>
                </div>
                <div class="col-12 mb-2">
                  <strong><i class="fas fa-chair me-2"></i>Mobiliario:</strong> 
                  {{ lastReport.mobiliario }}
                </div>
                <div class="col-12 mb-2">
                  <strong><i class="fas fa-tools me-2"></i>Tipo de daño:</strong> 
                  {{ lastReport.tipoDano }}
                </div>
                <div class="col-12 mb-2">
                  <strong><i class="fas fa-exclamation-triangle me-2"></i>Urgencia:</strong> 
                  <span class="badge ms-2" :class="getUrgenciaBadgeClass(lastReport.urgencia)">
                    {{ getUrgenciaText(lastReport.urgencia) }}
                  </span>
                </div>
                <div class="col-12">
                  <strong><i class="fas fa-calendar me-2"></i>Fecha:</strong> 
                  {{ formatDate(new Date()) }}
                </div>
              </div>
            </div>
            
            <div class="info-box bg-light p-3 rounded">
              <p class="text-muted mb-2">
                <i class="fas fa-info-circle me-2"></i>
                <strong>¿Qué sigue ahora?</strong>
              </p>
              <ul class="list-unstyled text-muted small text-start">
                <li><i class="fas fa-check text-success me-2"></i>El reporte ha sido registrado en el sistema</li>
                <li><i class="fas fa-paper-plane text-info me-2"></i>Se notificará al departamento de mantenimiento</li>
                <li><i class="fas fa-clock text-warning me-2"></i>Recibirá actualizaciones sobre el progreso</li>
                <li v-if="lastReport.urgencia === 'critico' || lastReport.urgencia === 'alto'">
                  <i class="fas fa-bolt text-danger me-2"></i>
                  Por la urgencia, se atenderá con prioridad
                </li>
              </ul>
            </div>
            
            <!-- Información adicional si se marcaron acciones especiales -->
            <div v-if="form.retirarUso || form.peligroUsuarios" class="alert alert-warning mt-3">
              <h6 class="alert-heading">
                <i class="fas fa-exclamation-triangle me-2"></i>Atención Especial
              </h6>
              <ul class="list-unstyled mb-0 small">
                <li v-if="form.retirarUso">
                  <i class="fas fa-ban me-2"></i>El mobiliario será retirado del uso inmediatamente
                </li>
                <li v-if="form.peligroUsuarios">
                  <i class="fas fa-shield-alt me-2"></i>Se ha marcado como peligro para usuarios
                </li>
              </ul>
            </div>
          </div>
          
          <div class="modal-footer justify-content-center border-0 pb-4">
            <button 
              type="button" 
              class="btn btn-success btn-lg me-2" 
              @click="handleNewReport"
            >
              <i class="fas fa-plus me-2"></i>Reportar Otro Daño
            </button>
            <router-link to="/mantenimientos" class="btn btn-outline-primary btn-lg">
              <i class="fas fa-list me-2"></i>Ver Todos los Reportes
            </router-link>
          </div>
        </div>
      </div>
    </div>

    <!-- Backdrop -->
    <div 
      v-if="showSuccessModal"
      class="modal-backdrop fade show"
      @click="showSuccessModal = false"
    ></div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted } from 'vue'
import { useMobiliarioStore } from '@/stores/mobiliario'
import { useAuthStore } from '@/stores/auth'
import { useLocalStorage } from '@/composables/useLocalStorage'

const mobiliarioStore = useMobiliarioStore()
const authStore = useAuthStore()
const { getItem, setItem } = useLocalStorage()

const loading = ref(false)
const mobiliarioEncontrado = ref(null)
const imagePreviews = ref([])
const showSuccessModal = ref(false)
const lastReport = ref(null)

const today = computed(() => new Date().toISOString().split('T')[0])
const user = computed(() => authStore.user)

const form = reactive({
  codigoMobiliario: '',
  tipoDano: '',
  descripcion: '',
  ubicacion: '',
  fechaReporte: today.value,
  nivelUrgencia: '',
  reportadoPor: '',
  requiereRemplazo: false,
  peligroUsuarios: false,
  retirarUso: false,
  comentarios: '',
  imagenes: []
})

const errors = reactive({
  codigoMobiliario: '',
  tipoDano: '',
  descripcion: '',
  fechaReporte: '',
  nivelUrgencia: '',
  imagenes: ''
})

// Methods
const buscarMobiliario = () => {
  if (!form.codigoMobiliario.trim()) {
    errors.codigoMobiliario = 'Ingrese un código válido'
    return
  }

  const item = mobiliarioStore.items.find(
    item => item.codigo.toLowerCase() === form.codigoMobiliario.toLowerCase()
  )

  if (item) {
    mobiliarioEncontrado.value = item
    form.ubicacion = `${item.ubicacion.edificio}, ${item.ubicacion.aula}`
    errors.codigoMobiliario = ''
  } else {
    mobiliarioEncontrado.value = null
    form.ubicacion = ''
    errors.codigoMobiliario = 'Mobiliario no encontrado'
  }
}

const clearMobiliarioData = () => {
  if (mobiliarioEncontrado.value) {
    mobiliarioEncontrado.value = null
    form.ubicacion = ''
  }
  if (errors.codigoMobiliario) {
    errors.codigoMobiliario = ''
  }
}

const handleFileUpload = (event) => {
  const files = Array.from(event.target.files)
  
  // Validar número de archivos
  if (files.length > 3) {
    errors.imagenes = 'Máximo 3 imágenes permitidas'
    event.target.value = ''
    return
  }

  // Validar cada archivo
  const validFiles = []
  const allowedTypes = ['image/jpeg', 'image/png', 'image/jpg']
  
  for (const file of files) {
    // Validar tipo
    if (!allowedTypes.includes(file.type)) {
      errors.imagenes = 'Solo se permiten imágenes JPEG o PNG'
      event.target.value = ''
      return
    }
    
    // Validar tamaño (2MB)
    if (file.size > 2 * 1024 * 1024) {
      errors.imagenes = 'Cada imagen debe pesar menos de 2MB'
      event.target.value = ''
      return
    }
    
    validFiles.push(file)
  }

  errors.imagenes = ''
  form.imagenes = validFiles

  // Crear previews
  imagePreviews.value = []
  validFiles.forEach((file, index) => {
    const reader = new FileReader()
    reader.onload = (e) => {
      imagePreviews.value.push({
        url: e.target.result,
        name: file.name,
        file: file
      })
    }
    reader.readAsDataURL(file)
  })
}

const removeImage = (index) => {
  imagePreviews.value.splice(index, 1)
  form.imagenes.splice(index, 1)
  
  // Actualizar el input file
  const fileInput = document.getElementById('imagenDano')
  if (form.imagenes.length === 0) {
    fileInput.value = ''
  }
}

const validateForm = () => {
  // Limpiar errores
  Object.keys(errors).forEach(key => {
    errors[key] = ''
  })

  let isValid = true

  // Código mobiliario
  if (!form.codigoMobiliario.trim()) {
    errors.codigoMobiliario = 'El código del mobiliario es obligatorio'
    isValid = false
  } else if (!mobiliarioEncontrado.value) {
    errors.codigoMobiliario = 'Debe buscar y seleccionar un mobiliario válido'
    isValid = false
  }

  // Tipo de daño
  if (!form.tipoDano) {
    errors.tipoDano = 'El tipo de daño es obligatorio'
    isValid = false
  }

  // Descripción
  if (!form.descripcion.trim()) {
    errors.descripcion = 'La descripción del daño es obligatoria'
    isValid = false
  } else if (form.descripcion.length < 10) {
    errors.descripcion = 'La descripción debe tener al menos 10 caracteres'
    isValid = false
  }

  // Fecha
  if (!form.fechaReporte) {
    errors.fechaReporte = 'La fecha de reporte es obligatoria'
    isValid = false
  } else if (form.fechaReporte > today.value) {
    errors.fechaReporte = 'La fecha no puede ser futura'
    isValid = false
  }

  // Nivel de urgencia
  if (!form.nivelUrgencia) {
    errors.nivelUrgencia = 'El nivel de urgencia es obligatorio'
    isValid = false
  }

  return isValid
}

const handleSubmit = async () => {
  console.log('🚀 Iniciando envío de reporte...')
  
  if (!validateForm()) {
    console.log('❌ Formulario no válido')
    return
  }

  loading.value = true

  try {
    // Generar código de reporte
    const reportCode = `RPT-${Date.now()}`
    console.log('📋 Código generado:', reportCode)
    
    // Crear objeto de reporte
    const reportData = {
      id: Date.now(),
      codigo: reportCode,
      mobiliarioId: mobiliarioEncontrado.value?.id,
      mobiliarioCodigo: form.codigoMobiliario,
      mobiliarioTipo: mobiliarioEncontrado.value?.tipo,
      tipoDano: form.tipoDano,
      descripcion: form.descripcion,
      ubicacion: form.ubicacion,
      fechaReporte: form.fechaReporte,
      nivelUrgencia: form.nivelUrgencia,
      reportadoPor: user.value?.fullName || 'Usuario desconocido',
      requiereRemplazo: form.requiereRemplazo,
      peligroUsuarios: form.peligroUsuarios,
      retirarUso: form.retirarUso,
      comentarios: form.comentarios,
      imagenes: form.imagenes.map(img => img.name),
      estado: 'pendiente',
      fechaCreacion: new Date().toISOString(),
      usuarioCreacion: user.value?.id || 'unknown'
    }

    console.log('📊 Datos del reporte:', reportData)

    // Guardar reporte en localStorage
    try {
      // Obtener reportes existentes o inicializar array si no existe
      const reportesExistentes = getItem('damage_reports') || []
      console.log('📚 Reportes existentes:', reportesExistentes)
      
      // Agregar nuevo reporte
      reportesExistentes.push(reportData)
      
      // Guardar en localStorage
      const guardado = setItem('damage_reports', reportesExistentes)
      
      if (!guardado) {
        throw new Error('No se pudo guardar el reporte en localStorage')
      }

      console.log('✅ Reporte guardado exitosamente')
      console.log('📈 Total reportes:', reportesExistentes.length)

      // Preparar datos para el modal
      lastReport.value = {
        codigo: reportCode,
        mobiliario: `${mobiliarioEncontrado.value?.tipo} - ${form.codigoMobiliario}`,
        tipoDano: getTipoDanoText(form.tipoDano),
        urgencia: form.nivelUrgencia
      }

      // Mostrar modal de éxito
      showSuccessModal.value = true

    } catch (storageError) {
      console.error('💾 Error al guardar en localStorage:', storageError)
      throw new Error('Error al guardar el reporte en el almacenamiento local')
    }

  } catch (error) {
    console.error('❌ Error completo al crear reporte:', error)
    console.error('📍 Stack trace:', error.stack)
    alert(`Error al enviar el reporte: ${error.message}\n\nRevisa la consola del navegador para más detalles.`)
  } finally {
    loading.value = false
  }
}

const handleCancel = () => {
  if (confirm('¿Está seguro de cancelar? Se perderán todos los datos ingresados.')) {
    resetForm()
  }
}

const handleNewReport = () => {
  showSuccessModal.value = false
  resetForm()
}

const resetForm = () => {
  Object.keys(form).forEach(key => {
    if (key === 'fechaReporte') {
      form[key] = today.value
    } else if (typeof form[key] === 'boolean') {
      form[key] = false
    } else if (Array.isArray(form[key])) {
      form[key] = []
    } else {
      form[key] = ''
    }
  })
  
  Object.keys(errors).forEach(key => {
    errors[key] = ''
  })
  
  mobiliarioEncontrado.value = null
  imagePreviews.value = []
  
  // Limpiar input file
  const fileInput = document.getElementById('imagenDano')
  if (fileInput) fileInput.value = ''
}

const getTipoDanoText = (tipo) => {
  const tipos = {
    'estructura': 'Daño estructural',
    'superficie': 'Daño superficial', 
    'funcional': 'Falla funcional',
    'desgaste': 'Desgaste normal',
    'vandalismo': 'Vandalismo',
    'otros': 'Otros'
  }
  return tipos[tipo] || tipo
}

const getUrgenciaText = (urgencia) => {
  const urgencias = {
    'bajo': 'Bajo',
    'medio': 'Medio',
    'alto': 'Alto',
    'critico': 'Crítico'
  }
  return urgencias[urgencia] || urgencia
}

const getUrgenciaBadgeClass = (urgencia) => {
  const classes = {
    'bajo': 'bg-success',
    'medio': 'bg-warning text-dark',
    'alto': 'bg-danger',
    'critico': 'bg-dark'
  }
  return classes[urgencia] || 'bg-secondary'
}

const formatDate = (date) => {
  return date.toLocaleDateString('es-ES', { 
    year: 'numeric', 
    month: 'long', 
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

// Función para verificar localStorage (debugging)
const verificarLocalStorage = () => {
  try {
    const reportes = getItem('damage_reports') || []
    console.log('🗃️ Reportes en localStorage:', reportes)
    return reportes
  } catch (error) {
    console.error('❌ Error al leer localStorage:', error)
    return []
  }
}

onMounted(() => {
  console.log('🔧 Componente montado') // Debug
  
  // Cargar items del inventario
  try {
    mobiliarioStore.loadItems()
    console.log('📦 Items del mobiliario cargados') // Debug
  } catch (error) {
    console.error('❌ Error al cargar items:', error)
  }
  
  // Establecer usuario actual
  form.reportadoPor = user.value?.fullName || 'Usuario actual'
  console.log('👤 Usuario actual:', form.reportadoPor) // Debug
  
  // Verificar localStorage
  const reportesExistentes = verificarLocalStorage()
  console.log(`📊 Se encontraron ${reportesExistentes.length} reportes existentes`) // Debug
})

// Función adicional para debugging - puedes llamarla desde la consola del navegador
if (typeof window !== 'undefined') {
  window.debugReportForm = {
    verificarLocalStorage,
    form,
    errors,
    mobiliarioEncontrado,
    user,
    showModal: () => showSuccessModal.value = true,
    hideModal: () => showSuccessModal.value = false
  }
}
</script>

<style scoped>
.reporte-mobiliario {
  animation: fadeInUp 0.6s ease-out;
}

.titulo-principal {
  color: #dc3545;
  font-weight: bold;
  font-size: 2.2rem;
  text-align: center;
  margin-bottom: 1rem;
}

.card {
  border: none;
  border-radius: 15px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.08);
}

.form-control:focus,
.form-select:focus {
  border-color: #dc3545;
  box-shadow: 0 0 0 0.2rem rgba(220, 53, 69, 0.25);
}

.btn-danger {
  background: linear-gradient(45deg, #dc3545, #c82333);
  border: none;
  font-weight: 500;
  transition: all 0.3s ease;
}

.btn-danger:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 5px 15px rgba(220, 53, 69, 0.4);
}

.btn-danger:disabled {
  opacity: 0.7;
  transform: none;
  box-shadow: none;
}

.img-thumbnail {
  border: 2px solid #dee2e6;
  border-radius: 8px;
}

.position-relative .btn-danger {
  width: 30px;
  height: 30px;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.modal.show {
  background-color: rgba(0, 0, 0, 0.5);
}

.modal-backdrop {
  background-color: rgba(0, 0, 0, 0.5);
}

/* Estilos mejorados para el modal */
.modal-content {
  border-radius: 15px;
  overflow: hidden;
}

.modal-header {
  border-bottom: none;
  padding: 2rem 1.5rem 1rem;
}

.modal-icon {
  animation: bounceIn 0.8s ease-out;
}

.success-animation {
  margin: 1rem 0;
}

.checkmark-circle {
  width: 80px;
  height: 80px;
  position: relative;
  display: inline-block;
  vertical-align: top;
  margin: 0 auto;
}

.checkmark-circle .background {
  width: 80px;
  height: 80px;
  border-radius: 50%;
  background: #28a745;
  position: absolute;
  animation: scaleIn 0.3s ease-in-out 0.9s both;
}

.checkmark-circle .checkmark {
  border-radius: 5px;
}

.checkmark-circle .checkmark.draw:after {
  animation-delay: 1.2s;
  animation-duration: 0.3s;
  animation-timing-function: ease;
  animation-name: checkmark;
  transform: scaleX(-1) rotate(135deg);
  animation-fill-mode: forwards;
}

.checkmark-circle .checkmark:after {
  opacity: 1;
  height: 35px;
  width: 15px;
  transform-origin: left top;
  border-right: 4px solid white;
  border-top: 4px solid white;
  border-radius: 2.5px;
  content: '';
  left: 25px;
  top: 40px;
  position: absolute;
}

.info-box {
  border-left: 4px solid #28a745;
}

.btn-lg {
  padding: 0.75rem 1.5rem;
  font-size: 1.1rem;
  border-radius: 10px;
  font-weight: 500;
}

@keyframes fadeInUp {
  from {
    opacity: 0;
    transform: translateY(30px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@keyframes bounceIn {
  0% {
    opacity: 0;
    transform: scale(0.3);
  }
  50% {
    opacity: 1;
    transform: scale(1.05);
  }
  70% {
    transform: scale(0.9);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes scaleIn {
  0% {
    opacity: 0;
    transform: scale(0);
  }
  100% {
    opacity: 1;
    transform: scale(1);
  }
}

@keyframes checkmark {
  0% {
    height: 0;
    width: 0;
    opacity: 1;
  }
  20% {
    height: 0;
    width: 15px;
    opacity: 1;
  }
  40% {
    height: 35px;
    width: 15px;
    opacity: 1;
  }
  100% {
    height: 35px;
    width: 15px;
    opacity: 1;
  }
}

/* Responsivo */
@media (max-width: 768px) {
  .titulo-principal {
    font-size: 1.8rem;
  }
  
  .modal-dialog {
    margin: 1rem;
  }
  
  .btn-lg {
    padding: 0.5rem 1rem;
    font-size: 1rem;
  }
}
</style>