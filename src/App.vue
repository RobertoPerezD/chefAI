<script setup lang="ts">
import { ref } from 'vue'
import { GoogleGenerativeAI } from '@google/generative-ai'
import { ChefHat, Loader2, Sparkles, Clock, BarChart3, ChevronRight } from 'lucide-vue-next'

// --- State ---
const userInput = ref('')
const isGenerating = ref(false)
const options = ref<any[]>([])
const selectedRecipe = ref<any>(null)

// --- Gemini Setup ---
const API_KEY = (import.meta.env.VITE_GEMINI_API_KEY || '').trim()
const genAI = new GoogleGenerativeAI(API_KEY)

const generateOptions = async () => {
  if (!userInput.value) return

  isGenerating.value = true
  options.value = []
  selectedRecipe.value = null

  try {
    const model = genAI.getGenerativeModel({ model: "gemini-flash-latest" })

    let prompt = `Actúa como un Chef Ejecutivo especializado en Alta Cocina Mexicana y Tradicional.
    El usuario dice: "${userInput.value}".
    
    INSTRUCCIONES:
    1. Analiza si el usuario pide algo genérico (ej: "quesadilla", "taco", "mole") o algo específico.
    2. Genera EXACTAMENTE 3 propuestas creativas de platos mexicanos basadas en esa idea.
    3. Si el usuario ya fue muy específico, genera la receta principal y 2 variaciones gourmet.
    4. Los nombres de los platos deben ser atractivos y profesionales.
    
    IMPORTANTE: Responde ÚNICAMENTE en formato JSON con la siguiente estructura (un array de 3 objetos):
    [
      {
        "name": "Nombre creativo del plato",
        "difficulty": "Fácil/Media/Alta",
        "time": "Tiempo total",
        "description": "Por qué esta opción es especial",
        "ingredients": ["ingrediente con medida", "ingrediente 2"],
        "steps": ["paso 1", "paso 2"],
        "tips": "Secreto del chef"
      }
    ]`

    const result = await model.generateContent(prompt)
    const responseText = result.response.text()
    const jsonMatch = responseText.match(/\[[\s\S]*\]/)
    if (jsonMatch) {
      options.value = JSON.parse(jsonMatch[0])
    }
  } catch (error: any) {
    console.error("Gemini Error:", error)
    alert("Error del Chef IA: " + (error.message || "Error desconocido"))
  } finally {
    isGenerating.value = false
  }
}

const selectRecipe = (recipe: any) => {
  selectedRecipe.value = recipe
  setTimeout(() => {
    window.scrollTo({ top: document.body.scrollHeight, behavior: 'smooth' })
  }, 100)
}
</script>

<template>
  <div class="container">
    <header class="header fade-in">
      <div class="logo-area">
        <ChefHat class="icon-chef" />
        <h1 class="gradient-text">Chef IA</h1>
      </div>
      <p class="subtitle">Alta Cocina Mexicana personalizada a tu gusto.</p>
    </header>

    <main class="main-card glass fade-in">
      <div class="input-section">
        <label>¿Qué se te antoja hoy?</label>
        <div class="input-wrapper">
          <textarea v-model="userInput" placeholder="Ej: Quiero una quesadilla, o algo con mariscos..."
            rows="2"></textarea>

          <button @click="generateOptions" :disabled="isGenerating || !userInput" class="btn-primary">
            <Loader2 v-if="isGenerating" class="spin" />
            <Sparkles v-else />
            {{ isGenerating ? 'Inspirándome...' : 'Explorar Opciones' }}
          </button>
        </div>
      </div>
    </main>

    <section v-if="options.length > 0 && !selectedRecipe" class="options-grid fade-in">
      <h3>El Chef te sugiere estas creaciones:</h3>
      <div class="options-container">
        <div v-for="(opt, idx) in options" :key="idx" @click="selectRecipe(opt)" class="option-card glass">
          <div class="option-header">
            <span class="option-tag">Opción {{ Number(idx) + 1 }}</span>
            <h4>{{ opt.name }}</h4>
          </div>
          <p class="opt-desc">{{ opt.description }}</p>
          <div class="option-footer">
            <span>
              <Clock :size="14" /> {{ opt.time }}
            </span>
            <ChevronRight :size="18" class="arrow" />
          </div>
        </div>
      </div>
    </section>

    <section v-if="selectedRecipe" class="recipe-area fade-in">
      <button @click="selectedRecipe = null" class="btn-back">← Volver a las opciones</button>

      <div class="recipe-card glass highlight-card">
        <div class="recipe-header">
          <h2>{{ selectedRecipe.name }}</h2>
          <div class="recipe-meta">
            <span>
              <Clock :size="16" /> {{ selectedRecipe.time }}
            </span>
            <span>
              <BarChart3 :size="16" /> {{ selectedRecipe.difficulty }}
            </span>
          </div>
        </div>

        <p class="recipe-desc">{{ selectedRecipe.description }}</p>

        <div class="recipe-grid">
          <div class="ingredients">
            <h3>Ingredientes</h3>
            <ul>
              <li v-for="ing in selectedRecipe.ingredients" :key="ing">{{ ing }}</li>
            </ul>
          </div>

          <div class="steps">
            <h3>Preparación</h3>
            <ol>
              <li v-for="(step, index) in selectedRecipe.steps" :key="index">
                <span class="step-num">{{ Number(index) + 1 }}</span>
                {{ step }}
              </li>
            </ol>
          </div>
        </div>

        <div class="chef-tip">
          <Sparkles class="tip-icon" />
          <p><strong>Tip del Chef:</strong> {{ selectedRecipe.tips }}</p>
        </div>
      </div>
    </section>

    <div v-if="options.length === 0 && !isGenerating" class="empty-placeholder">
      <Sparkles class="empty-icon" />
      <p>Escribe tu antojo para empezar</p>
    </div>
  </div>
</template>

<style scoped>
.header {
  text-align: center;
  margin-bottom: 3rem;
}

.logo-area {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 1rem;
  margin-bottom: 0.5rem;
}

.icon-chef {
  width: 40px;
  height: 40px;
  color: var(--primary);
}

.subtitle {
  color: var(--text-muted);
}

.main-card {
  padding: 2rem;
  margin-bottom: 2rem;
}

label {
  display: block;
  margin-bottom: 0.75rem;
  font-weight: 500;
  color: var(--primary);
}

textarea {
  width: 100%;
  background: rgba(0, 0, 0, 0.3);
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  padding: 1rem;
  color: white;
  font-family: inherit;
  resize: none;
  font-size: 1.1rem;
  margin-bottom: 1rem;
}

.btn-primary {
  width: 100%;
  background: var(--primary);
  color: white;
  border: none;
  padding: 1rem;
  border-radius: 0.75rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  font-size: 1.1rem;
  box-shadow: 0 4px 20px var(--primary-glow);
}

.options-grid h3 {
  margin-bottom: 1.5rem;
  color: var(--text-muted);
  font-size: 1.1rem;
  text-align: center;
}

.options-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 1.5rem;
  margin-bottom: 3rem;
}

.option-card {
  padding: 1.5rem;
  cursor: pointer;
  transition: all 0.3s ease;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.option-card:hover {
  border-color: var(--primary);
  transform: translateY(-5px);
  box-shadow: 0 8px 30px var(--primary-glow);
}

.option-tag {
  font-size: 0.7rem;
  text-transform: uppercase;
  color: var(--primary);
  font-weight: bold;
  letter-spacing: 1px;
}

.option-card h4 {
  margin: 0.5rem 0;
  font-size: 1.2rem;
}

.opt-desc {
  font-size: 0.9rem;
  color: var(--text-muted);
  display: -webkit-box;
  -webkit-line-clamp: 2;
  line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
  margin-bottom: 1rem;
}

.option-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  color: var(--text-muted);
  font-size: 0.8rem;
}

.btn-back {
  background: none;
  border: 1px solid var(--border);
  color: var(--text-muted);
  padding: 0.5rem 1rem;
  border-radius: 0.5rem;
  margin-bottom: 1.5rem;
  cursor: pointer;
}

.btn-back:hover {
  color: white;
  border-color: white;
}

.recipe-card {
  padding: 2.5rem;
  margin-bottom: 3rem;
}

.recipe-header {
  border-bottom: 1px solid var(--border);
  padding-bottom: 1.5rem;
  margin-bottom: 1.5rem;
}

.recipe-meta {
  display: flex;
  gap: 1.5rem;
  margin-top: 0.5rem;
  color: var(--text-muted);
  font-size: 0.9rem;
}

.recipe-meta span {
  display: flex;
  align-items: center;
  gap: 0.4rem;
}

.recipe-desc {
  font-style: italic;
  margin-bottom: 2rem;
  color: var(--text-muted);
}

.recipe-grid {
  display: grid;
  grid-template-columns: 1fr 1.5fr;
  gap: 2.5rem;
}

h3 {
  color: var(--primary);
  margin-bottom: 1rem;
  font-size: 1.1rem;
  text-transform: uppercase;
}

.ingredients li {
  padding: 0.5rem 0;
  border-bottom: 1px solid rgba(255, 255, 255, 0.05);
}

.steps li {
  margin-bottom: 1.25rem;
  position: relative;
  padding-left: 2.5rem;
}

.step-num {
  position: absolute;
  left: 0;
  width: 1.8rem;
  height: 1.8rem;
  background: var(--primary);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 0.8rem;
}

.chef-tip {
  margin-top: 2.5rem;
  background: rgba(249, 115, 22, 0.1);
  padding: 1.5rem;
  border-radius: 0.75rem;
  display: flex;
  gap: 1rem;
  align-items: center;
  border-left: 4px solid var(--primary);
}

.spin {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.fade-in {
  animation: fadeIn 0.8s ease;
}

.empty-placeholder {
  text-align: center;
  padding: 4rem;
  color: var(--text-muted);
  opacity: 0.3;
}

.empty-icon {
  width: 48px;
  height: 48px;
  margin-bottom: 1rem;
}

@media (max-width: 768px) {
  .recipe-grid {
    grid-template-columns: 1fr;
  }
}
</style>
