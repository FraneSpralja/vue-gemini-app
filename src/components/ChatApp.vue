<script setup>
import { reactive, ref } from 'vue';
const errorMsg = ref('')
const question = ref('')
const chatHistory = ref([])
const  surpriseQuestions = reactive([
  '¿Quién fue el último ganador del premio novel de la paz?',
  '¿De dónde viene la pizza?',
  '¿Cómo se hace una hamburguesa blt?',
  '¿Cuántos pares son tres moscas?',
  'Ha caballo regalado...',
  '¿Cuántos feriados hay en chile este año?'
])
const anchor = ref(null)

const surpriseMe = () => {
  const randomValue = surpriseQuestions[Math.floor(Math.random() * surpriseQuestions.length)]
  question.value = randomValue
}

const getAnswer = async () => {
  try {
    const options = {
      method: 'POST',
      body: JSON.stringify({
        history: chatHistory.value,
        message: question.value
      }),
      headers: {
        'Content-Type': 'application/json'
      }
    }

    const response = await fetch('http://localhost:8000/gemini', options)

    const data = await response.text()

    chatHistory.value = [ ...chatHistory.value, 
    {
      role: "user",
      parts: question.value
    },
    {
      role: "model",
      parts: data
    }
  ]

  question.value = ""
  anchor.value.scrollIntoView()


  } catch (error) {
    console.log(error)
    errorMsg.value = "Algo salió mal, por favor vuelve a intentarlo"
  }
}

const clear = (e) => {
  question.value = ''
  chatHistory.value = []
  if(e.target.classList.contains('error-btn')) {
    errorMsg.value = null
  }
}

</script>

<template>
  <section class="section-search">
    <div class="section-search__answers">
      <p v-if="errorMsg !== ''" class="error-msg">
        {{ errorMsg }}
      </p>
      <template v-else v-for="(chat, i) in chatHistory" :key="`${chat.role}_${i+1}`">
        <p class="chat-text" :class="`chat-text-${chat.role}`"><span>{{ chat.role }}: </span>{{ chat.parts }}</p>
      </template>
      <div id="anchor" ref="anchor"></div>
    </div>
    <p class="section-search__title">Pregúntame algo</p>
    <div class="section-search__input">
      <input type="text" class="search-input" placeholder="¿Cuándo es navidad?" v-model="question">
      <div class="section-search__actions">
        <button class="button surprise-me-btn" @click="surpriseMe">Surprise Me</button>
        <button class="button search-btn" :disabled="question === ''" @click="getAnswer">Preguntar</button>
        <button class="button clean-btn" :class="errorMsg ? 'error-btn' : ''" :disabled="chatHistory.length === 0"  @click="clear">{{ errorMsg ? 'Reiniciar' : 'Limpiar chat' }}</button>
      </div>
    </div>
  </section>
</template>
