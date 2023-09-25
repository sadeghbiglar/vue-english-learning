<template>
  <div class="container-fluid">
    <div class="row">
      <div v-for="(item, index) in words" :key="index" class="col-lg-4 col-md-6 col-xl-3">
        <div class="word-box" @click="toggleMeaning(index)">
          {{ item.word }}
          <div class="meaning" v-if="item.showMeaning">
            <strong>معنی:</strong>
            {{ item.meaning }}
          </div>
        </div>
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
       
        this.words.forEach(word => (word.showMeaning = false));
      } catch (error) {
        console.error("Error loading words:", error);
      }
    },
    toggleMeaning(index) {
      this.words[index].showMeaning = !this.words[index].showMeaning;
    }
  }
};
</script>

<style>
.word-box {
  border: 1px solid #ccc;
  padding: 10px;
  margin: 10px;
  cursor: pointer;
  text-align: center;
}

.meaning {
  display: none;
}

.word-box:hover .meaning {
  display: block;
}
</style>
