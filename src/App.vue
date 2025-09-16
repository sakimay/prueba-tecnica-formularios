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

          <div v-for="(question, index) in questions" :key="index" class="input-group" :class="question.type">

            <template v-if="['text', 'email', 'password', 'date', 'file'].includes(question.type)">
              <label class="custom-label" :for="`q_${question.id}`">{{ question.label }}</label>
              <input 
                :type="question.type" 
                :placeholder="question.placeholder || ''"
                class="custom-input" 
                :id="`q_${question.id}`"
                v-model="formData[question.id]"
                :class="{ 'custom-input-error': !!errors[question.id] }"
              />
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </template>

            <template v-else-if="question.type === 'textarea'">
              <label class="custom-label" :for="`q_${question.id}`">{{ question.label }}</label>
              <textarea 
                :placeholder="question.placeholder || ''" 
                class="custom-input"
                :id="`q_${question.id}`"
                v-model="formData[question.id]"
                :class="{ 'custom-input-error': !!errors[question.id] }"
              >{{ question.placeholder }}</textarea>
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </template>

            <template  v-else-if="question.type === 'select'">
              <label class="custom-label" :for="`q_${question.id}`">{{ question.label }}</label>
              <select 
                class="custom-input" 
                :id="`q_${question.id}`"
                v-model="formData[question.id]"
                :class="{ 'custom-input-error': !!errors[question.id] }"
              >
                <option value="">{{ question.placeholder }}</option>
                <option 
                  v-for="(option, optIndex) in question.options" 
                  :key="optIndex" 
                  :value="option"
                >{{ option }}
                </option>
              </select>
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </template>

            <template  v-else-if="question.type === 'radio'" class="flex flex-col gap-2">
              <p class="custom-label">{{ question.label }}</p>
              <div v-for="(option, optIndex) in question.options" :key="optIndex" class="flex items-center">
                <input 
                  type="radio" 
                  :id="`q_${question.id}_opt${optIndex}`" 
                  :name="`q_${question.id}`" 
                  :value="option"
                  class="custom-input"
                  v-model="formData[question.id]"
                  :class="{ 'custom-input-error': !!errors[question.id] }"
                />
                <label :for="`q_${question.id}_opt${optIndex}`">{{ option }}</label>
              </div>
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </template>

            <template  v-else-if="question.type === 'checkbox'" class="flex gap-2 items-center">
              <input
                :type="question.type" 
                :placeholder="question.placeholder || ''" 
                class="custom-input" 
                :id="`q_${question.id}`" 
                v-model="formData[question.id]"
                :class="{ 'custom-input-error': !!errors[question.id] }"
              />
              <label class="custom-label" :for="`q_${question.id}`">{{ question.label }}</label>
              <span v-if="errors[question.id]" class="custom-error">{{ errors[question.id] }}</span>
            </template>

          </div>
          <div class="p-6 flex justify-center">
            <button class="custom-button-submit">Enviar</button>
          </div>

        </form>
      </div>

    </section>
  </main>
  <footer></footer>
</template>

<script setup>
import Header from './components/Header.vue'
import questions from '@/config/questions.json'
import { reactive, watch } from 'vue'

const formData = reactive({})
const errors = reactive({})

questions.forEach(field => {
  if(field.type === 'checkbox') formData[field.id] = false
  else formData[field.id] = ''

  // Watch individual
  watch(
    () => formData[field.id],
    () => { errors[field.id] = '' } // limpia error al escribir/cambiar
  )
})

// Validación simple
function validateField(field) {
  const value = formData[field.id]
  const v = field.validation || {}
  let error = ''

  if(v.required) {
    if(field.type === 'checkbox' && !value) error = 'Este campo es obligatorio'
    else if(!value) error = 'Este campo es obligatorio'
  }

  if(!error && v.minLength && value.length < v.minLength)
    error = `Mínimo ${v.minLength} caracteres`

  if(!error && v.maxLength && value.length > v.maxLength)
    error = `Máximo ${v.maxLength} caracteres`

  if(!error && v.pattern) {
    const regex = new RegExp(v.pattern)
    if(!regex.test(value) && field.type == 'password') error = 'La contraseña debe tener al menos 8 caracteres, una mayúscula, un número y un carácter especial.'
    else if(!regex.test(value) && field.type == 'email') error = 'Formato del email inválido'
    else if(!regex.test(value) && field.id == 1) error = 'El nombre no debe contener números ni caracteres especiales.'
    else error = 'Formato inválido'
  }

  // Validación de matchField
  if(!error && v.matchField) {
    if(value !== formData[v.matchField]) error = 'No coincide con ' + v.matchField
  }

  // Validación minAge para fechas
  if(!error && v.minAge && field.type === 'date') {
    const today = new Date()
    const birth = new Date(value)
    const age = today.getFullYear() - birth.getFullYear()
    if(age < v.minAge) error = `Debes tener al menos ${v.minAge} años`
  }

  // Validación de accept para archivos
  if (!error && v.accept && field.type === 'file') {
    const input = document.getElementById(`q_${field.id}`)
    const file = input?.files?.[0]

    if (file) {
      const extension = '.' + file.name.split('.').pop().toLowerCase()
      if (!v.accept.map(ext => ext.toLowerCase()).includes(extension)) {
        error = 'Formato de archivo inválido. Solo se permiten: ' + v.accept.join(', ')
      }
    }
  }

  return error
}

const handleSubmit = () => {
  Object.keys(errors).forEach(key => errors[key] = '')
  console.log(formData)

  let hasError = false
  questions.forEach(field => {
    const error = validateField(field)
    if(error) {
      errors[field.id] = error
      hasError = true
    }
  })

  if(!hasError) {
    console.log('Formulario enviado:', JSON.parse(JSON.stringify(formData)))
    alert('Formulario enviado! Revisa la consola.')
  }
}
</script>

<style scoped></style>
