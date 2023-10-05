<template>
  <div class="container-fluid">
    <div class="row">
      <div v-for="(item, index) in shuffledWords" :key="item.id" class="col-lg-4 col-md-6 col-xl-3">
        <div v-if="!item.known" class="word-box">
          <div>
            <div>{{ item.word }}</div>

            <div class="button-container">
              <div class="button-wrapper">
                <button @click="markAsKnown(index)" class="btn btn-success">بلد بودم</button>
              </div>
              <div class="button-wrapper">
                <button @click="markAsUnknown(index)" class="btn btn-danger">بلد نبودم</button>
              </div>
            </div>
          </div>

          <div>
            <button @click="toggleMeaning(index)" class="btn btn-success">نمایش معنی</button>
            <div v-if="item.showMeaning">{{ item.meaning }}</div>
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

<script setup>
import { ref, onMounted } from "vue";

const words = ref([]); // آرایه‌ای برای نگهداری کلمات
const shuffledWords = ref([]); // کلمات به ترتیب تصادفی
const allWordsKnown = ref(false); // وضعیت بلد بودن همه کلمات
const stage = ref("stage1"); // مرحله فعلی (stage1 یا stage2 یا stage3)
const showSuccessMessage1 = ref(false); // نمایش پیام موفقیت مرحله 1
const showSuccessMessage2 = ref(false); // نمایش پیام موفقیت مرحله 2
const showSuccessMessage3 = ref(false); // نمایش پیام موفقیت مرحله 3
const correctWords = ref([]); // آرایه‌ای برای ذخیره کلمات درست
const wrongWords = ref([]); // آرایه‌ای برای ذخیره کلمات غلط
const show_meaning_click = ref(false);
const loadWords = async () => {
  try {
    const response = await fetch("/src/assets/words.json"); // مسیر فایل JSON را تعیین کنید
    if (!response.ok) {
      throw new Error("Unable to fetch data");
    }
    words.value = await response.json();
    // اضافه کردن ویژگی‌های showMeaning و known به هر کلمه برای نمایش معنی و بلد بودن
    words.value.forEach(word => {
      word.showMeaning = false;
      word.known = false;
    });
    // ترتیب تصادفی کلمات
    shuffledWords.value = shuffleArray([...words.value]);
  } catch (error) {
    console.error("Error loading words:", error);
  }
};

const toggleMeaning = index => {
  // تغییر وضعیت نمایش معنی با کلیک روی باکس
  shuffledWords.value[index].showMeaning = !shuffledWords.value[index]
    .showMeaning;
};

const markAsKnown = index => {
  // علامت‌گذاری به عنوان "بلد بودم"
  shuffledWords.value[index].known = true;

  // حذف کلمه از آرایه shuffledWords
  shuffledWords.value.splice(index, 1);

  // بررسی آیا همه کلمات بلد هستند
  allWordsKnown.value = shuffledWords.value.length === 0;
  if (allWordsKnown.value && stage.value === "stage1") {
    stage.value = "stage2";
    showSuccessMessage2.value = true;
    resetWordStatus();
    console.log(shuffledWords.value);
  } else if (allWordsKnown.value && stage.value === "stage2") {
    stage.value = "stage3";
    showSuccessMessage3.value = true;
  }
};

const markAsUnknown = index => {
  // علامت‌گذاری به عنوان "بلد نبودم"
  shuffledWords.value[index].known = false;
};

const shuffleArray = array => {
  // تابعی برای تصادفی کردن یک آرایه
  let shuffledArray = [...array];
  for (let i = shuffledArray.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [shuffledArray[i], shuffledArray[j]] = [shuffledArray[j], shuffledArray[i]];
  }
  return shuffledArray;
};

const resetWordStatus = () => {
  // بازنشانی وضعیت کلمات برای شروع مرحله جدید
  words.value.forEach(word => {
    word.showMeaning = false;
    word.known = false;
  });
  shuffledWords.value = shuffleArray([...words.value]);
};

onMounted(() => {
  // خواندن داده‌های JSON از فایل
  loadWords();
});
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
