<template>
  <div class="container-fluid">
    <h1>لیست کلمات</h1>
    <div class="row">
      <div v-for="(item, index) in shuffledWords" :key="item.id" class="col-lg-4 col-md-6 col-xl-3">
        <div
          v-if="item.known==false"
          class="word-box"
          @click="toggleMeaning(index)"
          :class="{ 'green-box': item.known, 'yellow-box': !item.known }"
        >
          <div v-if="!item.showMeaning">{{ item.word }}</div>
          <div class="meaning" v-if="item.showMeaning">
            <strong>معنی:</strong>
            {{ item.meaning }}
          </div>
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

    <div v-if="stage === 'stage2' && showSuccessMessage2">
      <div v-if="allWordsKnown" class="alert alert-success">موفق شدید!</div>
    </div>
    <div v-if="stage === 'stage3' && showSuccessMessage3">
      <div v-if="allWordsKnown" class="alert alert-success">عالی بود!</div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      words: [], // آرایه‌ای برای نگهداری کلمات
      shuffledWords: [], // کلمات به ترتیب تصادفی
      allWordsKnown: false, // وضعیت بلد بودن همه کلمات
      stage: "stage1", // مرحله فعلی (stage1 یا stage2 یا stage3)
      showSuccessMessage1: false, // نمایش پیام موفقیت مرحله 1
      showSuccessMessage2: false, // نمایش پیام موفقیت مرحله 2
      showSuccessMessage3: false, // نمایش پیام موفقیت مرحله 3
      correctWords: [], // آرایه‌ای برای ذخیره کلمات درست
      wrongWords: [] // آرایه‌ای برای ذخیره کلمات غلط
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
        // اضافه کردن ویژگی‌های showMeaning و known به هر کلمه برای نمایش معنی و بلد بودن
        this.words.forEach(word => {
          word.showMeaning = false;
          word.known = false;
        });
        // ترتیب تصادفی کلمات
        this.shuffledWords = this.shuffleArray(this.words.slice());
      } catch (error) {
        console.error("Error loading words:", error);
      }
    },
    toggleMeaning(index) {
      // تغییر وضعیت نمایش معنی با کلیک روی باکس
      this.shuffledWords[index].showMeaning = !this.shuffledWords[index]
        .showMeaning;
    },
    markAsKnown(index) {
      // علامت‌گذاری به عنوان "بلد بودم"
      this.shuffledWords[index].known = true;

      // حذف کلمه از آرایه shuffledWords
      this.shuffledWords.splice(index, 1);

      // بررسی آیا همه کلمات بلد هستند
      this.allWordsKnown = this.shuffledWords.length === 0;
      if (this.allWordsKnown && this.stage == "stage1") {
        this.stage = "stage2";
        this.showSuccessMessage2 = true;
        this.resetWordStatus();
        console.log(this.shuffledWords);
      } else if (this.allWordsKnown && this.stage == "stage2") {
        this.stage = "stage3";
        this.showSuccessMessage3 = true;
      }
    },
    markAsUnknown(index) {
      // علامت‌گذاری به عنوان "بلد نبودم"
      this.shuffledWords[index].known = false;
    },
    shuffleArray(array) {
      // تابعی برای تصادفی کردن یک آرایه
      let shuffledArray = array.slice();
      for (let i = shuffledArray.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [shuffledArray[i], shuffledArray[j]] = [
          shuffledArray[j],
          shuffledArray[i]
        ];
      }
      return shuffledArray;
    },
    resetWordStatus() {
      // بازنشانی وضعیت کلمات برای شروع مرحله جدید
      this.words.forEach(word => {
        word.showMeaning = false;
        word.known = false;
      });
      this.shuffledWords = this.shuffleArray(this.words.slice());
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
  transition: background-color 0.3s; /* انتقال رنگ با انیمیشن */
}

.green-box {
  background-color: green;
}

.yellow-box {
  background-color: yellow;
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
