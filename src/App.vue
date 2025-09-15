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
      <div class="container p-10 max-w-3xl mx-auto flex">
        <form class="custom-form" @submit.prevent="handleSubmit">

          <div v-for="(question, index) in questions" :key="index" class="p-6">

            <div v-if="['text', 'email', 'password', 'date', 'file'].includes(question.type)">
              <label class="text-xl font-semibold mb-4" :for="`q_${index}`">{{ question.label }}</label>
              <input 
                :type="question.type" 
                :placeholder="question.placeholder || ''"
                class="custom-input" 
                :id="`q_${index}`" 
                :required="question.validation.required"/>
            </div>

            <div v-else-if="question.type === 'textarea'">
              <label class="text-xl font-semibold mb-4" :for="`q_${index}`">{{ question.label }}</label>
              <textarea 
                :placeholder="question.placeholder || ''" 
                class="custom-input"
                :id="`q_${index}`" 
                :required="question.validation.required"
              >
              </textarea>
            </div>

            <div v-else-if="question.type === 'select'">
              <label class="text-xl font-semibold mb-4" :for="`q_${index}`">{{ question.label }}</label>
              <select 
                class="custom-input" 
                :id="`q_${index}`" 
                :required="question.validation.required"
              >
                <option 
                  v-for="(option, optIndex) in question.options" 
                  :key="optIndex" 
                  :value="option"
                >{{ option }}
                </option>
              </select>
            </div>

            <div v-else-if="question.type === 'radio'" class="flex flex-col gap-2">
              <div v-for="(option, optIndex) in question.options" :key="optIndex" class="flex items-center">
                <input 
                  type="radio" 
                  :id="`q_${index}_opt${optIndex}`" 
                  :name="`q_${index}`" 
                  :value="option"
                  class="mr-2" 
                  :required="question.validation.required"
                />
                <label :for="`q_${index}_opt${optIndex}`">{{ option }}</label>
              </div>
            </div>

            <div v-else-if="question.type === 'checkbox'" class="flex gap-2 items-center">
              <input 
                :type="question.type" 
                :placeholder="question.placeholder || ''" 
                class="" 
                :id="`q_${index}`" 
                :required="question.validation.required"
              />
              <label class="text-xl font-semibold" :for="`q_${index}`">{{ question.label }}</label>
            </div>

          </div>
          <div class="p-6">
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
import { reactive } from 'vue'

const formData = reactive({})
const errors = reactive({})

const handleSubmit = () => {
  alert('Formulario enviado!')
}
</script>

<style scoped></style>
