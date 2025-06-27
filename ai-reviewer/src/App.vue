<script setup>
import { ref } from "vue";
const code = ref("");

const prompt = ref(
  `You are a senior JavaScript developer. Please review the following code.

1. Point out any bugs or potential issues.
2. Mention what is well-written or follows best practices.
3. Keep your response concise and structured in bullet points.
`
);
const step = ref(1);
const answer = ref("");
const API_KEY = import.meta.env.VITE_OPENAI_API_KEY;

async function getGptAnswer() {
  console.log("Starting review");
  try {
    const response = await fetch("https://api.openai.com/v1/chat/completions", {
      method: "POST",
      headers: {
        Authorization: `Bearer ${API_KEY}`,
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        max_tokens: 300,
        model: "gpt-4",
        messages: [
          { role: "system", content: "You are a code reviewer." },
          { role: "user", content: `${prompt.value}\n\n${code.value}` },
        ],
      }),
    });

    const data = await response.json();
    console.log(data);
    console.log("API KEY:", API_KEY);

    answer.value = data.choices?.[0]?.message?.content || "No answer";
    console.log(answer.value);
  } catch (err) {
    console.error("GPT error:", err);
    answer.value = "Error during fetch.";
  }
}
</script>

<template>
  <div class="container">
    <div v-if="step === 1">


      <h1>Task:</h1>
      <p>Write a red button with an inline styling</p>
      <pre>
        <p>{{ code }}</p>
        <textarea v-model = "code"></textarea>
      </pre>
      <button @click="step++">Next</button>
    </div>

    <div v-else-if="step === 2">
      <p>Let's assume this code was fetched from GitHub.</p>
      <button @click="step++">Analyze</button>
    </div>

    <div v-else-if="step === 3">
      <p>🔍 Review:</p>
      <ul>
        <button @click="getGptAnswer()">Get Answer</button>
        <p>{{ answer }}</p>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.container {
  padding: 1rem;
  font-family: sans-serif;
}
button {
  margin-top: 1rem;
}
</style>
