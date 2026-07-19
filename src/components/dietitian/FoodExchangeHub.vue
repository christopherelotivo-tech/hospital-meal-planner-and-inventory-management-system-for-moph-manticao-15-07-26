<template>
  <div class="space-y-6">
    <!-- Header -->
    <div class="bg-gradient-to-r from-emerald-600 to-green-500 rounded-2xl p-5 md:p-8 text-white shadow-lg relative overflow-hidden">
      <div class="relative z-10">
        <h2 class="font-caprasimo text-2xl md:text-3xl mb-2 tracking-wide text-shadow-sm">Food Exchange & AI Hub</h2>
        <p class="font-dm-sans text-emerald-50 text-sm md:text-base max-w-2xl leading-relaxed">
          Your digital assistant for macronutrient calculations and meal planning. 
          Use the reference tables for standard Philippine food exchanges or let the AI instantly generate menus for you.
        </p>
      </div>
      <div class="absolute right-0 bottom-0 opacity-20 transform translate-x-4 translate-y-4">
        <Sparkles :size="80" class="md:w-[120px] md:h-[120px]" />
      </div>
    </div>

    <!-- Tabs -->
    <div class="flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-2 bg-gray-100 p-1.5 rounded-xl w-full sm:w-max shadow-inner">
      <button 
        @click="activeTab = 'table'" 
        class="px-5 py-2.5 sm:py-2 rounded-lg font-semibold text-sm transition-all flex items-center justify-center space-x-2"
        :class="activeTab === 'table' ? 'bg-white text-emerald-700 shadow-sm' : 'text-gray-500 hover:text-gray-700 hover:bg-gray-200'"
      >
        <Table2 :size="16" />
        <span>Reference Tables</span>
      </button>
      <button 
        @click="activeTab = 'ai'" 
        class="px-5 py-2.5 sm:py-2 rounded-lg font-semibold text-sm transition-all flex items-center justify-center space-x-2"
        :class="activeTab === 'ai' ? 'bg-white text-purple-700 shadow-sm' : 'text-gray-500 hover:text-gray-700 hover:bg-gray-200'"
      >
        <Sparkles :size="16" />
        <span>AI Menu Assistant</span>
      </button>
    </div>

    <!-- Tab 1: Reference Table -->
    <div v-if="activeTab === 'table'" class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden">
      <div class="p-4 md:p-6 border-b border-gray-100 bg-gray-50 flex flex-col sm:flex-row sm:items-center justify-between gap-4">
        <div>
          <h3 class="text-base md:text-lg font-bold text-gray-900 flex items-center space-x-2">
            <BookOpen :size="18" class="text-emerald-600 shrink-0" />
            <span>Philippine Food Exchange List</span>
          </h3>
          <p class="text-xs md:text-sm text-gray-500 mt-1">Standard macronutrients per 1 exchange based on FNRI guidelines.</p>
        </div>
        <button class="w-full sm:w-auto px-4 py-2.5 bg-emerald-100 text-emerald-700 rounded-lg text-sm font-semibold hover:bg-emerald-200 transition-colors flex items-center justify-center space-x-2">
          <Printer :size="16" />
          <span>Print</span>
        </button>
      </div>
      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse">
          <thead>
            <tr class="bg-gray-100/50">
              <th class="p-4 font-semibold text-sm text-gray-700 border-b border-gray-200">Food Group</th>
              <th class="p-4 font-semibold text-sm text-gray-700 border-b border-gray-200 text-center">CHO (g)</th>
              <th class="p-4 font-semibold text-sm text-gray-700 border-b border-gray-200 text-center">PRO (g)</th>
              <th class="p-4 font-semibold text-sm text-gray-700 border-b border-gray-200 text-center">FAT (g)</th>
              <th class="p-4 font-semibold text-sm text-gray-700 border-b border-gray-200 text-center">Energy (kcal)</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-100 text-sm">
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Milk (Cow's, Whole)</td>
              <td class="p-4 text-center text-gray-600">12</td>
              <td class="p-4 text-center text-gray-600">8</td>
              <td class="p-4 text-center text-gray-600">10</td>
              <td class="p-4 text-center font-semibold text-emerald-600">170</td>
            </tr>
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Vegetable A (Raw/Leafy)</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center font-semibold text-emerald-600">Free</td>
            </tr>
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Vegetable B (Starchy/Cooked)</td>
              <td class="p-4 text-center text-gray-600">3</td>
              <td class="p-4 text-center text-gray-600">1</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center font-semibold text-emerald-600">16</td>
            </tr>
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Fruit</td>
              <td class="p-4 text-center text-gray-600">10</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center font-semibold text-emerald-600">40</td>
            </tr>
            <tr class="hover:bg-gray-50 bg-emerald-50/30">
              <td class="p-4 font-bold text-gray-900">Rice / Carbohydrates</td>
              <td class="p-4 text-center font-bold text-gray-700">23</td>
              <td class="p-4 text-center font-bold text-gray-700">2</td>
              <td class="p-4 text-center font-bold text-gray-700">-</td>
              <td class="p-4 text-center font-bold text-emerald-700">100</td>
            </tr>
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Meat (Low Fat)</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">8</td>
              <td class="p-4 text-center text-gray-600">1</td>
              <td class="p-4 text-center font-semibold text-emerald-600">41</td>
            </tr>
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Meat (Medium Fat)</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">8</td>
              <td class="p-4 text-center text-gray-600">5</td>
              <td class="p-4 text-center font-semibold text-emerald-600">77</td>
            </tr>
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Fat</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">5</td>
              <td class="p-4 text-center font-semibold text-emerald-600">45</td>
            </tr>
            <tr class="hover:bg-gray-50">
              <td class="p-4 font-medium text-gray-900">Sugar</td>
              <td class="p-4 text-center text-gray-600">5</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center text-gray-600">-</td>
              <td class="p-4 text-center font-semibold text-emerald-600">20</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <!-- Tab 2: AI Chatbot -->
    <div v-if="activeTab === 'ai'" class="bg-white rounded-2xl shadow-[0_8px_30px_rgb(0,0,0,0.04)] border border-purple-100 flex flex-col h-[calc(100vh-250px)] min-h-[500px] max-h-[700px] overflow-hidden">
      <!-- Chat Header -->
      <div class="px-4 md:px-6 py-4 border-b border-purple-100 bg-gradient-to-r from-purple-50 to-white flex items-center justify-between shrink-0">
        <div class="flex items-center space-x-3">
          <div class="w-10 h-10 rounded-full bg-purple-600 flex items-center justify-center text-white shadow-sm">
            <Sparkles :size="20" />
          </div>
          <div>
            <h3 class="font-bold text-gray-900 text-sm md:text-base">Dietitian AI Assistant</h3>
            <p class="text-[10px] md:text-xs text-purple-600 font-medium">Powered by Food Exchange Engine</p>
          </div>
        </div>
        <button @click="messages = [initialMessage]" class="p-2 text-gray-400 hover:text-gray-600 hover:bg-gray-100 rounded-lg transition-colors" title="Clear Chat">
          <RefreshCw :size="18" />
        </button>
      </div>

      <!-- Chat Messages -->
      <div class="flex-1 overflow-y-auto p-6 space-y-6 bg-gray-50/50" ref="chatContainer">
        <div v-for="(msg, index) in messages" :key="index" class="flex" :class="msg.role === 'user' ? 'justify-end' : 'justify-start'">
          
          <!-- AI Avatar -->
          <div v-if="msg.role === 'ai'" class="w-8 h-8 rounded-full bg-purple-600 flex items-center justify-center text-white shrink-0 mr-3 mt-1 shadow-sm">
            <Sparkles :size="14" />
          </div>

          <!-- Message Bubble -->
          <div class="max-w-[85%] md:max-w-[80%] rounded-2xl px-4 py-3 md:px-5 md:py-3.5 shadow-sm group relative" :class="msg.role === 'user' ? 'bg-purple-600 text-white rounded-tr-sm' : 'bg-white border border-gray-100 text-gray-800 rounded-tl-sm'">
            <!-- Markdown Rendering for AI -->
            <div v-if="msg.role === 'ai'" class="prose prose-sm prose-purple max-w-none font-dm-sans leading-relaxed text-[13px] md:text-sm" v-html="renderMarkdown(msg.content)"></div>
            <div v-else class="font-dm-sans text-[13px] md:text-sm">{{ msg.content }}</div>

            <!-- Print Button (Only for AI messages) -->
            <button 
              v-if="msg.role === 'ai' && index > 0" 
              @click="printAIResponse(msg.content)"
              class="absolute -right-10 bottom-2 p-1.5 text-gray-400 hover:text-emerald-600 hover:bg-emerald-50 rounded-lg opacity-0 group-hover:opacity-100 transition-all shadow-sm border border-transparent hover:border-emerald-200 bg-white"
              title="Print this plan for patient"
            >
              <Printer :size="16" />
            </button>
          </div>
        </div>

        <!-- Typing Indicator -->
        <div v-if="isTyping" class="flex justify-start">
          <div class="w-8 h-8 rounded-full bg-purple-600 flex items-center justify-center text-white shrink-0 mr-3 mt-1 shadow-sm">
            <Sparkles :size="14" />
          </div>
          <div class="bg-white border border-gray-100 rounded-2xl rounded-tl-sm px-5 py-4 shadow-sm flex items-center space-x-2">
            <div class="w-2 h-2 bg-purple-400 rounded-full animate-bounce" style="animation-delay: 0ms"></div>
            <div class="w-2 h-2 bg-purple-400 rounded-full animate-bounce" style="animation-delay: 150ms"></div>
            <div class="w-2 h-2 bg-purple-400 rounded-full animate-bounce" style="animation-delay: 300ms"></div>
          </div>
        </div>
      </div>

      <!-- Chat Input -->
      <div class="p-4 bg-white border-t border-gray-100 shrink-0">
        <form @submit.prevent="sendMessage" class="relative flex items-center">
          <input 
            v-model="newMessage" 
            type="text" 
            placeholder="E.g., Generate a 1500 kcal diabetic menu using Filipino foods..." 
            class="w-full pl-5 pr-14 py-4 bg-gray-50 border border-gray-200 rounded-xl focus:bg-white focus:border-purple-400 focus:ring-2 focus:ring-purple-100 outline-none transition-all font-dm-sans text-sm"
            :disabled="isTyping"
          />
          <button 
            type="submit" 
            :disabled="!newMessage.trim() || isTyping"
            class="absolute right-2 p-2.5 bg-purple-600 hover:bg-purple-700 disabled:bg-gray-300 disabled:cursor-not-allowed text-white rounded-lg transition-colors flex items-center justify-center"
          >
            <Send :size="18" />
          </button>
        </form>
        <div class="mt-3 flex gap-2 overflow-x-auto pb-1 hide-scrollbar">
          <button @click="newMessage = 'Generate a 1500 kcal Diabetic Diet menu'" class="shrink-0 px-3 py-1.5 bg-purple-50 hover:bg-purple-100 text-purple-700 rounded-full text-xs font-medium transition-colors border border-purple-200">1500 kcal Diabetic</button>
          <button @click="newMessage = 'Create a High Protein (100g) meal pattern'" class="shrink-0 px-3 py-1.5 bg-purple-50 hover:bg-purple-100 text-purple-700 rounded-full text-xs font-medium transition-colors border border-purple-200">High Protein Pattern</button>
          <button @click="newMessage = 'Low Sodium/Low Fat menu ideas'" class="shrink-0 px-3 py-1.5 bg-purple-50 hover:bg-purple-100 text-purple-700 rounded-full text-xs font-medium transition-colors border border-purple-200">Low Sodium/Fat</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick } from 'vue';
import { Table2, Sparkles, BookOpen, Printer, RefreshCw, Send } from 'lucide-vue-next';
import { marked } from 'marked';
import DOMPurify from 'dompurify';
import axios from 'axios';

const activeTab = ref('table');
const isTyping = ref(false);
const newMessage = ref('');
const chatContainer = ref(null);

const initialMessage = {
  role: 'ai',
  content: "Hello! I am your Dietary Assistant. I can help you calculate Food Exchanges, generate Daily Food Plans (macros), or create sample Filipino menus based on DOH standards. What would you like to calculate today?"
};

const messages = ref([initialMessage]);

// Smarter mock responder that detects keywords
const getMockResponse = (prompt) => {
  const p = prompt.toLowerCase();
  
  if (p.includes('protein')) {
    return `Here is a **High Protein Pattern (100g)** based on the Food Exchange List:

### Daily Food Plan (Macros)
* **Carbohydrates:** 250g (1000 kcal)
* **Protein:** 100g (400 kcal)
* **Fat:** 65g (585 kcal)
* **Energy:** ~2000 kcal

### Sample Filipino Menu
* **Breakfast:** 2 Eggs (scrambled), 1 cup Garlic Rice, 1 glass Skim Milk
* **Lunch:** Beef Broccoli (150g beef), 1.5 cups Rice, 1 slice Papaya
* **Dinner:** Grilled Tuna (150g), 1.5 cups Rice, Steamed Kang-kong

*Would you like me to instantly add this menu to your Dish Menu database?*`;
  }
  
  if (p.includes('sodium') || p.includes('fat') || p.includes('hypertension')) {
    return `Here is a **Low Sodium / Low Fat Menu** based on the Food Exchange List:

### Daily Food Plan (Macros)
* **Sodium limit:** < 2000mg/day
* **Fat limit:** 40g (360 kcal)

### Sample Filipino Menu (No added salt/patis)
* **Breakfast:** 1 cup Oatmeal with fresh Banana, 1 Boiled Egg
* **Lunch:** Sinigang na Isda (using tamarind, no MSG), 1 cup Brown Rice
* **Dinner:** Baked Chicken Breast (Lemon/Garlic marinade), Steamed Veggies

*Would you like me to instantly add this menu to your Dish Menu database?*`;
  }

  // Default response (Diabetic)
  return `Here is a **1500 kcal Diabetic Menu** based on the Philippine Food Exchange List:

### Daily Food Plan (Macros)
* **Carbohydrates:** 200g (800 kcal)
* **Protein:** 60g (240 kcal)
* **Fat:** 50g (450 kcal)

### Sample Filipino Menu
* **Breakfast:** 1 cup Oatmeal, 1 Boiled Egg, 1 slice Papaya
* **Lunch:** 1 cup Brown Rice, Chicken Tinola (100g breast)
* **Dinner:** 1 cup Brown Rice, Grilled Bangus (60g), Steamed Okra

*Would you like me to instantly add this menu to your Dish Menu database?*`;
};

const renderMarkdown = (text) => {
  return DOMPurify.sanitize(marked.parse(text));
};

const scrollToBottom = async () => {
  await nextTick();
  if (chatContainer.value) {
    chatContainer.value.scrollTop = chatContainer.value.scrollHeight;
  }
};

const sendMessage = async () => {
  if (!newMessage.value.trim() || isTyping.value) return;

  // Add user message
  messages.value.push({
    role: 'user',
    content: newMessage.value
  });
  
  const userPrompt = newMessage.value;
  newMessage.value = '';
  isTyping.value = true;
  scrollToBottom();

  try {
    // Try to call the real backend API
    const response = await axios.post('http://localhost:8000/api/chat/food-exchange', {
      prompt: userPrompt
    });
    
    isTyping.value = false;
    messages.value.push({
      role: 'ai',
      content: response.data.message
    });
    scrollToBottom();
  } catch (error) {
    console.warn("AI Backend API not ready or failed. Falling back to mock data.", error);
    // Simulate AI delay for the mock fallback
    setTimeout(() => {
      isTyping.value = false;
      messages.value.push({
        role: 'ai',
        content: getMockResponse(userPrompt)
      });
      scrollToBottom();
    }, 1500);
  }
};

const printAIResponse = (content) => {
  const htmlContent = renderMarkdown(content);
  const printWindow = window.open('', '_blank');
  
  printWindow.document.write(`
    <html>
      <head>
        <title>Dietary Plan & Food Exchanges</title>
        <style>
          body { font-family: 'Inter', sans-serif; padding: 40px; color: #1f2937; max-width: 800px; margin: 0 auto; line-height: 1.6; }
          .header { border-bottom: 2px solid #10b981; padding-bottom: 20px; margin-bottom: 30px; }
          .header h1 { margin: 0; color: #047857; font-size: 24px; }
          .header p { margin: 5px 0 0 0; color: #6b7280; font-size: 14px; }
          h3 { color: #4c1d95; margin-top: 30px; font-size: 18px; border-bottom: 1px solid #e5e7eb; padding-bottom: 8px; }
          ul { padding-left: 20px; }
          li { margin-bottom: 8px; }
          strong { color: #111827; }
          @media print {
            body { padding: 0; }
            button { display: none; }
          }
        </style>
      </head>
      <body>
        <div class="header">
          <h1>MOPH - Manticao Dietary Plan</h1>
          <p>Generated by Dietitian AI Assistant | Date: ${new Date().toLocaleDateString()}</p>
        </div>
        <div class="content">
          ${htmlContent}
        </div>
        <script>
          setTimeout(() => {
            window.print();
            window.close();
          }, 500);
        <\/script>
      </body>
    </html>
  `);
  printWindow.document.close();
};
</script>

<style scoped>
.hide-scrollbar::-webkit-scrollbar {
  display: none;
}
.hide-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
.prose :deep(h3) {
  @apply text-purple-900 mt-4 mb-2 font-bold text-base;
}
.prose :deep(ul) {
  @apply list-disc pl-5 mb-4 space-y-1;
}
.prose :deep(li) {
  @apply text-gray-700;
}
.prose :deep(strong) {
  @apply text-gray-900;
}
</style>
