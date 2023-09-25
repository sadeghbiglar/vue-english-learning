<template>
  <div class="container-fluid">
    <div class="row">
      <div v-for="(item, index) in words" :key="index" class="col-lg-4 col-md-6 col-xl-3">
        <div class="word-box" @click="toggleMeaning(index)">
          <div v-if="!item.showMeaning">{{ item.word }}</div>
          <div class="meaning" v-if="item.showMeaning">{{ item.meaning }}</div>
          <div class="button-container">
            <div class="button-wrapper">
              <button
                @click="markAsKnown(index)"
                class="btn btn-success"
                v-if="item.showMeaning"
              >بلد بودم</button>
            </div>
            <div class="button-wrapper">
              <button
                @click="markAsUnknown(index)"
                class="btn btn-danger"
                v-if="item.showMeaning"
              >بلد نبودم</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-if="allWordsKnown" class="alert alert-success">موفق شدید!</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      words: [], // آرایه‌ای برای نگهداری کلمات
      allWordsKnown: false // وضعیت بلد بودن همه کلمات
    };
  },
  mounted() {
    // خواندن داده‌های JSON از فایل
    this.loadWords();
  },
  methods: {
    async loadWords() {
      try {
        const response = await fetch("/src/assets/words.json"); // مسیر فایل JSON را تعیین کنید
        if (!response.ok) {
          throw new Error("Unable to fetch data");
        }
        this.words = await response.json();
        // اضافه کردن ویژگی showMeaning به هر کلمه برای نمایش معنی
        this.words.forEach(word => (word.showMeaning = false));
      } catch (error) {
        console.error("Error loading words:", error);
      }
    },
    toggleMeaning(index) {
      // تغییر وضعیت نمایش معنی با کلیک روی باکس
      this.words[index].showMeaning = !this.words[index].showMeaning;
    },
    markAsKnown(index) {
      // علامت‌گذاری به عنوان "بلد بودم"
      this.words[index].known = true;
      // بررسی آیا همه کلمات بلد هستند
      this.allWordsKnown = this.words.every(word => word.known);
    },
    markAsUnknown(index) {
      // علامت‌گذاری به عنوان "بلد نبودم"
      this.words[index].known = false;
      // کنترل مجدد برای پنهان کردن پیام هشدار
      this.allWordsKnown = false;
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

.button-container {
  display: flex;
  justify-content: space-between; /* تنظیم فضای بین دو دکمه */
}

.button-wrapper {
  flex: 1;
  margin: 5px; /* تنظیم فاصله بین دکمه‌ها */
}

.alert-success {
  margin-top: 20px; /* تعیین فاصله بین پیام هشدار و کلمات */
}
</style>
