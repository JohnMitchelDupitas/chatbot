<template>
  <ion-page>
    <ion-content>
      <div class="chat-container">
        <h1>CHATBOT NI JM</h1>
        
        <div ref="chatsContainer" class="chats-container">
          <div v-for="(message, index) in messages" :key="index" 
               :class="['message', message.type === 'user' ? 'user-message' : 'bot-message']">
            <img v-if="message.type === 'bot'" src="https://scontent.fmnl13-2.fna.fbcdn.net/v/t39.30808-6/470183694_1246212846595198_3676952656073397783_n.jpg?_nc_cat=106&ccb=1-7&_nc_sid=a5f93a&_nc_eui2=AeHH7el1uEALN1DVh7v7p30b-VSLmyp7Hvr5VIubKnse-tcLlKlp5jPm6A3dW7ykfFMAFmptk_YzU1Ahe7o4NB3q&_nc_ohc=9C5IaFtZbu8Q7kNvwH3t3VY&_nc_oc=AdlQ_qqqi8mZe0aVc-oKyeVRPJeqhQbIT_EudtXpNaNe2v5WWwoervUkJsk21-tDhUA&_nc_zt=23&_nc_ht=scontent.fmnl13-2.fna&_nc_gid=PXUuL_BvwmOqJpKn3beGjA&oh=00_AfNQ6FDSE7lwuLrrnIvj5qD4ehWeb2Zk6V2STgbM2yS9LA&oe=684A0435" class="avatar" />
            <p class="message-text">{{ message.text }}</p>
          </div>
        </div>

        <form @submit.prevent="handleFormSubmit" class="prompt-form">
          <ion-input
            v-model="userInput"
            placeholder="Type your message..."
            class="prompt-input"
          />
          <ion-button type="submit">Send</ion-button>
        </form>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import { IonPage, IonContent, IonInput, IonButton } from '@ionic/vue';

const API_KEY = "AIzaSyBJkduUoy5109BaF82PywkWCKAbq3HbRtY";
const API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${API_KEY}`;

const messages = ref([]);
const userInput = ref('');
const chatsContainer = ref(null);
const chatHistory = ref([]);

const scrollToBottom = () => {
  setTimeout(() => {
    if (chatsContainer.value) {
      chatsContainer.value.scrollTop = chatsContainer.value.scrollHeight;
    }
  }, 100);
};

const typingEffect = (text, messageIndex) => {
  const words = text.split(" ");
  let index = 0;
  
  const interval = setInterval(() => {
    if (index < words.length) {
      messages.value[messageIndex].text += (index > 0 ? " " : "") + words[index++];
      scrollToBottom();
    } else {
      clearInterval(interval);
    }
  }, 40);
};

const generateResponse = async () => {
  const messageIndex = messages.value.length - 1;
  
  try {
    const response = await fetch(API_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ contents: chatHistory.value })
    });

    const data = await response.json();
    if (!response.ok) throw new Error(data.error?.message || "Unknown error");

    const responseText = data.candidates[0].content.parts[0].text
      .replace(/\*\*([^*]+)\*\*/g, "$1")
      .trim();

    messages.value[messageIndex].text = '';
    typingEffect(responseText, messageIndex);
    chatHistory.value.push({ role: "model", parts: [{ text: responseText }] });
  } catch (error) {
    console.error("API Error:", error.message);
    messages.value[messageIndex].text = "Error: " + error.message;
  }
};

const handleFormSubmit = () => {
  if (!userInput.value.trim()) return;

  // Add user message
  messages.value.push({
    type: 'user',
    text: userInput.value.trim()
  });

  chatHistory.value.push({
    role: "user",
    parts: [{ text: userInput.value.trim() }]
  });

  // Clear input
  userInput.value = '';
  scrollToBottom();

  // Add bot message with new reply text
  setTimeout(() => {
    messages.value.push({
      type: 'bot',
      text: 'J Dupitas reply...'
    });
    scrollToBottom();
    generateResponse();
  }, 600);
};
</script>

<style scoped>
.chat-container {
  padding: 20px;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.chats-container {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 20px;
}

.message {
  padding: 8px 12px;
  margin-bottom: 10px;
  border-radius: 10px;
  max-width: 75%;
  clear: both;
  font-size: 15px;
  line-height: 1.4;
}

.user-message {
  background-color: #4285f4;
  color: white;
  float: right;
  text-align: right;
}

.bot-message {
  background-color: #f1f0f0;
  color: #333;
  float: left;
  text-align: left;
  display: flex;
  align-items: center;
  gap: 10px;
}

.avatar {
  width: 20px;
  height: 20px;
}

.prompt-form {
  display: flex;
  gap: 10px;
  padding: 10px;
  background: white;
  position: sticky;
  bottom: 0;
  border-top: 1px solid #eee;
}

.prompt-input {
  --background: white;
  --color: black;
  --padding-start: 10px;
  --padding-end: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  flex: 1;
}

ion-button {
  --background: #4285f4;
}

h1 {
  text-align: center;
  margin-bottom: 20px;
}
</style>