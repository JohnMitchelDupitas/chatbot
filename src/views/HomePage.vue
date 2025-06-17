<template>
  <ion-page>
    <ion-content>
      <div class="container">
        <h1>Chatbot ni Jm</h1>
        
        <div ref="msgContainer" class="messages">
          <div v-for="(msg, i) in chat" :key="i" 
               :class="['msg', msg.from === 'user' ? 'user' : 'bot']">
            <img v-if="msg.from === 'bot'" src="https://res.cloudinary.com/dflyqatql/image/upload/v1750159886/470183694_1246212846595198_3676952656073397783_n_i1xjxo.jpg" class="pic" />
            <p>{{ msg.content }}</p>
          </div>
        </div>

        <form @submit.prevent="sendMessage" class="input-form">
          <ion-input v-model="input" placeholder="Message..." class="text-input" />
          <ion-button type="submit">Send</ion-button>
        </form>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup>
import { ref } from 'vue';
import { IonPage, IonContent, IonInput, IonButton } from '@ionic/vue';

const API_KEY = "AIzaSyBJkduUoy5109BaF82PywkWCKAbq3HbRtY";
const API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${API_KEY}`;

const chat = ref([]);
const input = ref('');
const msgContainer = ref(null);
const history = ref([]);

const scroll = () => {
  setTimeout(() => {
    if (msgContainer.value) {
      msgContainer.value.scrollTop = msgContainer.value.scrollHeight;
    }
  }, 50);
};

const type = (text, idx) => {
  const words = text.split(" ");
  let i = 0;
  
  const timer = setInterval(() => {
    if (i < words.length) {
      chat.value[idx].content += (i > 0 ? " " : "") + words[i++];
      scroll();
    } else {
      clearInterval(timer);
    }
  }, 40);
};

const getReply = async () => {
  const idx = chat.value.length - 1;
  
  try {
    const res = await fetch(API_URL, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ contents: history.value })
    });

    const data = await res.json();
    if (!res.ok) throw new Error(data.error?.message || "Error occurred");

    const reply = data.candidates[0].content.parts[0].text.trim();
    chat.value[idx].content = '';
    type(reply, idx);
    history.value.push({ role: "model", parts: [{ text: reply }] });
  } catch (err) {
    console.error("Error:", err.message);
    chat.value[idx].content = "Sorry, an error occurred";
  }
};

const sendMessage = () => {
  if (!input.value.trim()) return;

  chat.value.push({
    from: 'user',
    content: input.value.trim()
  });

  history.value.push({
    role: "user",
    parts: [{ text: input.value.trim() }]
  });

  input.value = '';
  scroll();

  setTimeout(() => {
    chat.value.push({
      from: 'bot',
      content: 'Wait lang lods i chatgpt ko lang'
    });
    scroll();
    getReply();
  }, 500);
};
</script>

<style scoped>
.container {
  padding: 20px;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.messages {
  flex: 1;
  overflow-y: auto;
  margin-bottom: 20px;
}

.msg {
  padding: 8px 12px;
  margin: 8px 0;
  border-radius: 8px;
  max-width: 75%;
  clear: both;
  font-size: 15px;
}

.user {
  background: #4285f4;
  color: white;
  float: right;
}

.bot {
  background: #f1f0f0;
  color: #333;
  float: left;
  display: flex;
  align-items: center;
  gap: 8px;
}

.pic {
  width: 20px;
  height: 20px;
}

.input-form {
  display: flex;
  gap: 8px;
  padding: 10px;
  background: white;
  position: sticky;
  bottom: 0;
  border-top: 1px solid #eee;
}

.text-input {
  --background: white;
  --color: black;
  --padding-start: 10px;
  --padding-end: 10px;
  border: 1px solid #ddd;
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