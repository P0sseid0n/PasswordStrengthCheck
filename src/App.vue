<script setup lang="ts">
import { computed, ref, watch, type StyleValue } from 'vue'

const rules = [
   {
      description: 'Deve ter letras e números',
      completed: false,
      type: 'number'
   },
   {
      description: 'Deve ter conter símbolos (@, #, $)',
      completed: false,
      type: 'symbol'
   },
   {
      description: 'Deve ter conter letras maiúsculas e minúsculas',
      completed: false,
      type: 'letter'
   },
   {
      description: 'Deve ter no mínimo 8 caracteres',
      completed: false,
      type: 'length8'
   },
   {
      description: 'Deve ser longa (+14 caracteres)',
      completed: false,
      type: 'length14'
   },
]

const password = ref('')
const fetching = ref(false)
const strengthScore = ref(0)

const strengthBarStyle = computed(() => {
   const returnStyle: StyleValue = {
      width: `${strengthScore.value}%`
   }
   if (strengthScore.value < 100 / 3 * 1) {
      returnStyle.background = '#f44336'
   } else if (strengthScore.value < 100 / 3 * 2.5) {
      returnStyle.background = '#ff9800'
   } else {
      returnStyle.background = '#4caf50'
   }

   return returnStyle
})


async function fetchRandomPasswords() {
   if (fetching.value) return
   fetching.value = true
   try {
      const response = await fetch(`https://random.justyy.workers.dev/api/random?cached&n=16`)
      password.value = await response.json()
   } finally {
      fetching.value = false
   }
}

watch(password, (newValue) => {
   strengthScore.value = 0
   rules.forEach(rule => {
      if (rule.type === 'number') rule.completed = /[0-9]+/.test(newValue) && /[a-zA-Z]+/.test(newValue)
      else if (rule.type === 'symbol') rule.completed = /[@#$%?~]+/.test(newValue)
      else if (rule.type === 'letter') rule.completed = /[a-z]+/.test(newValue) && /[A-Z]+/.test(newValue)
      else if (rule.type === 'length8') rule.completed = newValue.length >= 8
      else if (rule.type === 'length14') rule.completed = newValue.length >= 14


      const strengthPoints = 100 / rules.length

      if (rule.completed) strengthScore.value += strengthPoints
   })
})

</script>

<template>
   <main>
      <p id="password-length" v-html="password.length"></p>
      <input id="password" type="text" v-model.trim="password" />
      <div id="strength-bar">
         <div :style="strengthBarStyle"></div>
      </div>
   </main>
   <button @click="fetchRandomPasswords" :disabled="fetching">Gerar senha</button>
   <ul>
      <li v-for="rule in rules" :key="rule.description" v-html="rule.description"
         :style="{ color: rule.completed ? '#367e39' : undefined, fontWeight: rule.completed ? 500 : undefined }"></li>
   </ul>
</template>

<style>
@import url('https://fonts.googleapis.com/css2?family=Poppins:wght@400;500;700&display=swap');

* {
   font-family: 'Poppins', sans-serif;
   box-sizing: border-box;
   padding: 0;
   margin: 0;
   text-align: center;
}

body {
   min-height: 100vh;
   line-height: 1.6;
   font-size: 15px;
   text-rendering: optimizeLegibility;
   -webkit-font-smoothing: antialiased;
   -moz-osx-font-smoothing: grayscale;
}

#app {
   width: 100vw;
   height: 100vh;
   display: flex;
   flex-direction: column;
   align-items: center;
   justify-content: center;
   padding: 0 4%;
}

main {
   display: inline-block;
   width: 100%;
   max-width: 512px;
}

#password-length {
   font-weight: 700;
   color: rgb(125, 125, 125);
   width: 100%;
   text-align: right;
   padding: 4px 6px 0 0;
   font-size: 13px;
}

#password {
   width: 100%;
   padding: 0.5rem 1rem;
   border: 1px solid rgb(100, 100, 100);
   border-radius: 0.25rem;
   font-size: 1.25rem;
   background-color: rgb(235, 235, 235);
   border: none;
   outline: none;
}

#strength-bar {
   width: 100%;
   height: 4px;
   background-color: rgb(235, 235, 235);
   border-radius: 0.25rem;
   margin-top: 0.5rem;
   overflow: hidden;
}

#strength-bar div {
   height: 100%;
   background-color: rgb(136, 67, 67);
   width: 0;
   transition: width 0.5s;
}

button {
   margin-top: 32px;
   font-size: 14px;
   margin-bottom: 64px;
   padding: 0.5rem 1rem;
   border: none;
   background: rgb(230, 230, 230);
   border-radius: 0.25rem;
   font-weight: 500;
   color: rgb(75, 75, 75);
   transition: transform 0.2s;
}

button:hover:enabled {
   transform: scale(1.1);
}

button:disabled {
   opacity: 0.5;
}

ul {
   list-style: none;
}

li {
   line-height: 28px;
}
</style>
