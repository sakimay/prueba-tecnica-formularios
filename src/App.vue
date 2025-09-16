<template>
  <Header />
  <main class="mt-16">

    <section class="background-hero">
      <div class="container p-10 max-w-7xl mx-auto flex">
        <div class="flex flex-col gap-2 text-center md:text-left w-full">
          <h1 class="text-white text-4xl md:text-6xl">Prueba Técnica</h1>
          <h2 class="text-secondary text-4xl md:text-6xl">Formularios dinámicos</h2>
        </div>
      </div>
    </section>

    <section>
      <div class="container p-4 md:p-10 max-w-3xl mx-auto flex">
        <form class="custom-form" @submit.prevent="handleSubmit">
          <h3 class="text-xl font-semibold mb-4">Formulario de registro</h3>

          <div v-for="(question, index) in questions" :key="index" class="input-group" :class="question.type">

            <div v-if="['text', 'email', 'password', 'date', 'file'].includes(question.type)" class="input-group">
              <label class="custom-label" :for="`q_${question.id}`">
                {{ question.label }} <span v-if="question.validation?.required">*</span> 
                <span v-if="question.type == 'file'" class="text-gray-500 font-light italic">({{ question.validation?.accept.join(', ') }})</span>
              </label>
              <input 
                :type="question.type" 
                :placeholder="question.placeholder || ''"
                class="custom-input" 
                :id="`q_${question.id}`"
                v-model="formData[question.id]"
                :class="{ 'custom-input-error': !!errors[question.id] }" 
                @input="errors[question.id] = validateField(question)"  
              />
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </div>

            <div v-else-if="question.type === 'textarea'" class="input-group">
              <label class="custom-label" :for="`q_${question.id}`">{{ question.label }} <span v-if="question.validation?.required">*</span></label>
              <textarea 
                :placeholder="question.placeholder || ''" 
                class="custom-input"
                :id="`q_${question.id}`"
                v-model="formData[question.id]"
                :class="{ 'custom-input-error': !!errors[question.id] }"
                @input="errors[question.id] = validateField(question)"
              ></textarea>
              <div class="flex justify-between items-center">
                <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
                <span v-else></span>
                <span class="text-gray-500">{{ formData[question.id].length }} / {{ question.validation.maxLength }}</span>
              </div>
            </div>

            <div v-else-if="question.type === 'select'" class="input-group">
              <label class="custom-label" :for="`q_${question.id}`">{{ question.label }} <span v-if="question.validation?.required">*</span></label>
              <select 
                class="custom-input" 
                :id="`q_${question.id}`"
                v-model="formData[question.id]"
                :class="{ 'custom-input-error': !!errors[question.id] }"
                @change="errors[question.id] = validateField(question)"
              >
                <option 
                  v-for="(option, optIndex) in question.options" 
                  :key="optIndex" 
                  :value="option"
                >{{ option }}
                </option>
              </select>
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </div>

            <div v-else-if="question.type === 'radio'" class="input-group">
              <div :aria-label="question.label" role="group">
                <legend class="custom-label">{{ question.label }} <span v-if="question.validation?.required">*</span></legend>
                <div v-for="(option, optIndex) in question.options" :key="optIndex" class="flex items-center">
                  <input 
                    type="radio" 
                    :id="`q_${question.id}_opt${optIndex}`" 
                    :name="`q_${question.id}`" 
                    :value="option"
                    class="custom-input"
                    v-model="formData[question.id]"
                    :class="{ 'custom-input-error': !!errors[question.id] }"
                    @change="errors[question.id] = validateField(question)"
                  />
                  <label :for="`q_${question.id}_opt${optIndex}`">{{ option }}</label>
                </div>
                <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
              </div>
            </div>

            <div v-else-if="question.type === 'checkbox'" class="input-group">
              <div :aria-label="question.label" role="group" class="flex items-center">
                <input
                  :type="question.type" 
                  :placeholder="question.placeholder || ''" 
                  class="custom-input" 
                  :id="`q_${question.id}`" 
                  v-model="formData[question.id]"
                  :class="{ 'custom-input-error': !!errors[question.id] }"
                  @change="errors[question.id] = validateField(question)"
                />
                <label class="custom-label" :for="`q_${question.id}`">{{ question.label }}</label>
              </div>
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </div>

          </div>
          <div class="p-6 flex justify-center">
            <button class="custom-button-submit" :disabled="!validateForm()">Enviar</button>
          </div>

          <span class="text-gray-500 font-light italic">* Campos obligatorios</span>
        </form>
      </div>

    </section>
  </main>
  <footer></footer>
</template>

<script setup>
import Header from './components/Header.vue'
import questions from '@/config/questions.json'
import { reactive } from 'vue'

const formData = reactive({})
const errors = reactive({})

questions.forEach(f => formData[f.id] = f.type === "checkbox" ? false : "")

// Validación simple
const validateField = (field) => {
  const value = formData[field.id]
  const validation = field.validation || {}
  let error = ''

  // Validación de required
  if(validation.required) {
    if(field.type === 'checkbox' && !value) error = 'Este campo es obligatorio'
    else if(!value) error = 'Este campo es obligatorio'
  }

  // Validación de minLength
  if(!error && validation.minLength && value.length < validation.minLength)
    error = `Mínimo ${validation.minLength} caracteres`

  // Validación de maxLength
  if(!error && validation.maxLength && value.length > validation.maxLength)
    error = `Máximo ${validation.maxLength} caracteres`

  // Validación de pattern
  if(!error && validation.pattern) {
    const regex = new RegExp(validation.pattern)
    if(value && !regex.test(value) && field.type == 'password') error = 'La contraseña debe tener al menos 8 caracteres, una mayúscula, un número y un carácter especial.'
    if(value && !regex.test(value) && field.type == 'email') error = 'Formato del email inválido'
    if(value && !regex.test(value.trim()) && field.id == 1) error = 'El nombre no debe contener números ni caracteres especiales.'
  }

  // Validación de matchField
  if(!error && validation.matchField) {
    const targetField = questions.find(f => f.label === validation.matchField)
    if(value !== formData[targetField.id]) error = 'No coincide con ' + validation.matchField
  }

  // Validación minAge para fechas
  if(!error && validation.minAge && field.type === 'date') {
    const today = new Date()
    const birth = new Date(value)
    const age = today.getFullYear() - birth.getFullYear()
    if(age < validation.minAge) error = `Debes tener al menos ${validation.minAge} años`
  }

  // Validación de accept para archivos
  if (!error && validation.accept && field.type === 'file') {
    const input = document.getElementById(`q_${field.id}`)
    const file = input?.files?.[0]

    if (file) {
      const extension = '.' + file.name.split('.').pop().toLowerCase()
      if (!validation.accept.map(ext => ext.toLowerCase()).includes(extension)) {
        error = 'Formato de archivo inválido. Solo se permiten: ' + validation.accept.join(', ')
      }
    }
  }

  return error
}

const validateForm = () => {
  let valid = true;
  questions.forEach(field => {
    if (validateField(field)) valid = false;
  });
  return valid;
};


const handleSubmit = () => {
  if (validateForm()) {
    alert("Formulario válido");
    console.log("Datos:", formData);
  } else {
    alert("Hay errores en el formulario");
  }
}
</script>
