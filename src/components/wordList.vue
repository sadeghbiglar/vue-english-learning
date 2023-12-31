<template>
  <div class="container-fluid">
    <div>
      <div class="progress-container">
        <div class="progress-bar" :style="{ width: `${stage1Progress}%` }"></div>
        <div class="progress-bar" :style="{ width: `${stage2Progress}%` }"></div>
        <div class="progress-bar" :style="{ width: `${stage3Progress}%` }"></div>
      </div>
    </div>
    <div class="row">
      <div v-for="(item, index) in shuffledWords" :key="item.id" class="col-lg-4 col-md-6 col-xl-3">
        <div v-if="!item.known" class="word-box">
          <div>
            <img :src="`src/assets/images/${item.word}.jpg`" class="image-container">
          </div>
          <div>
            <div style="font-size:40px ">{{ item.word }}</div>

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
            <button @click="toggleMeaning(index)" class="btn btn-primary">نمایش معنی</button>
            <div v-if="item.showMeaning">{{ item.meaning }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const words = ref([]); // آرایه‌ای برای نگهداری کلمات
const shuffledWords = ref([]); // کلمات به ترتیب تصادفی
const allWordsKnown = ref(false); // وضعیت بلد بودن همه کلمات
const stage = ref("stage1"); // مرحله فعلی (stage1 یا stage2 یا stage3)
const correctWords = ref([]); // آرایه‌ای برای ذخیره کلمات درست
const wrongWords = ref([]); // آرایه‌ای برای ذخیره کلمات غلط
const stage1Progress = ref(0);
const stage2Progress = ref(0);
const stage3Progress = ref(0);
const totalProgress = 100;
const loadWords = async () => {
  try {
    const response = await fetch("/src/assets/words.json");
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

  if (stage.value === "stage3") {
    const removedValue = shuffledWords.value.splice(index, 1)[0];
    correctWords.value.push(removedValue);
  } else {
    shuffledWords.value.splice(index, 1);
  }
  // بررسی آیا همه کلمات بلدبودم هستند
  allWordsKnown.value = shuffledWords.value.length === 0;
  if (allWordsKnown.value && stage.value === "stage1") {
    stage.value = "stage2";
    stage1Progress.value = 30; // مرحله 1
    resetWordStatus();
    // console.log(shuffledWords.value);
  } else if (allWordsKnown.value && stage.value === "stage2") {
    stage.value = "stage3";
    stage2Progress.value = 60; // مرحله 2
    resetWordStatus();
    // console.log(shuffledWords.value);
  } else if (allWordsKnown.value && stage.value === "stage3") {
    stage.value = "stage_end";
    stage3Progress.value = 100; // مرحله 3
    //resetWordStatus();
    // console.log(shuffledWords.value);
    console.log(correctWords.value);
    console.log(wrongWords.value);
  }
};

const markAsUnknown = index => {
  // علامت‌گذاری به عنوان "بلد نبودم"

  if (stage.value === "stage3") {
    shuffledWords.value[index].known = false;
    const removedValue = shuffledWords.value.splice(index, 1)[0];
    wrongWords.value.push(removedValue);
    if(shuffledWords.value.length===0){
       stage.value = "stage_end";
    stage3Progress.value = 100; // مرحله 3
    //resetWordStatus();
    // console.log(shuffledWords.value);
    console.log(correctWords.value);
    console.log(wrongWords.value);
    }
  } else {
    shuffledWords.value = shuffleArray([...shuffledWords.value]);
  }
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

<style scoped>
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
.progress-container {
  width: 100%;
  height: 20px;
  background-color: #eee;
  margin-top: 10px;
  position: relative;
  border-radius: 10px;
}

.progress-bar {
  height: 100%;
  background-color: #e228e2;
  position: absolute;
  border-radius: 10px;
}
.image-container {
  width: 200px;
  height: 150px;
}
</style>
