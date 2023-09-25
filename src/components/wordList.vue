<template>
  <div>
    <h1>لیست کلمات</h1>
    <div v-for="(item, index) in words" :key="index" class="word-box">
      <div>
        <strong>کلمه:</strong>
        {{ item.word }}
      </div>
      <div>
        <strong>معنی:</strong>
        {{ item.meaning }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      words: []
    };
  },
  mounted() {
    this.loadWords();
  },
  methods: {
    async loadWords() {
      try {
        const response = await fetch("/src/assets/words.json");
        if (!response.ok) {
          throw new Error("Unable to fetch data");
        }
        this.words = await response.json();
      } catch (error) {
        console.error("Error loading words:", error);
      }
    }
  }
};
</script>

<style>
.word-box {
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px;
}
</style>
