<template>
  <div>
    <h2>Ask ChatGPT</h2>
    <form @submit.prevent="handleSubmit">
      <input
        type="text"
        v-model="prompt"
        placeholder="Enter your question..."
        :disabled="isStreaming"
      />
      <button type="submit" :disabled="isStreaming || !prompt">Ask</button>
    </form>
    <div v-if="messages.length > 0">
      <span v-for="(message, index) in messages" :key="index">{{ message }}</span>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      prompt: '',
      messages: [],
      isStreaming: false,
      eventSource: null
    }
  },
  methods: {
    handleSubmit() {
      if (this.eventSource) {
        this.eventSource.close()
      }

      this.messages = []
      this.isStreaming = true

      this.eventSource = new EventSource(
        `http://localhost:8000/chat?message=${encodeURIComponent(this.prompt)}`
      )

      this.eventSource.onmessage = (event) => {
        if (event.data === '[END OF RESPONSE]') {
          this.eventSource.close()
          this.isStreaming = false
        } else {
          this.messages.push(event.data)
        }
      }

      this.eventSource.onerror = () => {
        console.error('EventSource connection error')
        this.eventSource.close()
        this.isStreaming = false
      }
    }
  },
  beforeUnmount() {
    if (this.eventSource) {
      this.eventSource.close()
    }
  }
}
</script>

<style scoped>
input {
  margin-right: 10px;
}
</style>
