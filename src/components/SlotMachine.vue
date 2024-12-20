<template>
  <div
    class="slot-machine"
    style="background: url('/images/bk.jpg') no-repeat center center; background-size: 100% 100%;"
  >
    <!-- 拉桿圖片，位於slot-machine后面 -->
    <div class="lever" :class="{ active: leverState }">
        <img
          src="/images/lever-up.png"
          class="lever-img"
          alt="lever"
          @mousedown="pullLever"
          @mouseup="resetLever"
        />
      </div>

    <!-- 拉霸機主體圖片 -->
    <div class="machine">
      <img src="/images/slot-machine.png" class="machine-img" alt="slot-machine" />

      <!-- 數字輪盤 -->
      <div class="reels">
        <div v-for="(reel, index) in reels" :key="index" class="reel">
          <div
            class="numbers"
            :style="{ transform: `translateY(${reel.position}px)` }"
          >
            <img
              v-for="(num, idx) in reel.numbers"
              :key="idx"
              :src="`/images/${num}`"
              class="number-img"
              alt="number"
            />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";

// 每個輪盤的數字範圍
const reelSets = [
  ["L2.png", "L3.png", "L5.png", "L6.png", "L7.png", "L8.png"], // 第一個輪盤圖片
  ["C8.png", "C9.png"], // 第二個輪盤圖片
  ["R8.png"], // 第三個輪盤固定為 R8.png
];

// 初始化輪盤狀態
const reels = ref([
  { numbers: createReelSet(reelSets[0]), position: 0 },
  { numbers: createReelSet(reelSets[1]), position: 0 },
  { numbers: createReelSet(reelSets[2]), position: 0 },
]);

// 拉桿狀態
const leverState = ref(false);

// 拉桿觸發
const pullLever = () => {
  leverState.value = true;
  resetPositions(); // 重置輪盤位置
  startGame(); // 啟動遊戲邏輯
};

const resetLever = () => {
  leverState.value = false; // 拉桿回復正常
};

// 啟動輪盤滾動邏輯
const startGame = () => {
  // 選擇圖片
  const firstImage = getRandomImage(reelSets[0]);
  const secondImage =
    firstImage === "L8.png"
      ? getRandomImage(reelSets[1].filter((n) => n !== "C9.png"))
      : getRandomImage(reelSets[1]);
  const thirdImage = "R8.png";
  const selectedImages = [firstImage, secondImage, thirdImage];
  reels.value.forEach((reel, index) => {
    rollReel(reel, reelSets[index], selectedImages[index]);
  });
};

// 滾動單個輪盤
const rollReel = (reel, reelSet, targetImage) => {
  const imageIndex = reelSet.indexOf(targetImage); // 目标图片索引
  const extraCycles = 4; // 额外滚动圈数
  const imageHeight = 180; // 每张图片的高度（确保这个值与图片的实际高度一致）
  const totalImages = reelSet.length; // 图片数量
  if (targetImage === "R8.png") {
    reel.isBlurred = true; // 设置标志，应用模糊
  } else {
    reel.isBlurred = false; // 其他轮盘不应用模糊
  }

  // 计算滚动位置，确保位置在合理范围
  const position = -(extraCycles * totalImages + imageIndex) * imageHeight;

  // 限制滚动位置的范围，避免超出容器
  const maxPosition = 0; // 最大滚动位置
  const minPosition = -(imageHeight * totalImages); // 最小滚动位置
  reel.position = Math.min(Math.max(position, minPosition), maxPosition);
  // 延迟确保滚动动画完成
  setTimeout(() => {
    reel.position = position;
  }, 1000); // 适当延迟，调整滚动时间
};

const resetPositions = () => {
  reels.value.forEach((reel) => (reel.position = 0));
};

function getRandomImage(array) {
  const shuffled = shuffleArray([...array]);
  return shuffled[0];
}

function shuffleArray(array) {
  for (let i = array.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [array[i], array[j]] = [array[j], array[i]];
  }
  return array;
}

function createReelSet(numbers) {
  const repeats = 30;
  return Array(repeats)
    .fill()
    .map((_, i) => numbers[i % numbers.length]);
}
</script>

<style scoped>
.slot-machine {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 90vh;
  position: relative;
  padding: 40px;
  background-size: cover;
  overflow: hidden;
}

.machine {
  position: relative;
  width: 950px;
  height: 500px;
  z-index: 2;
  pointer-events: none; 
}

.machine-img {
  position: absolute;
  top: 70px;
  left: 50%;
  transform: translateX(-50%);
  max-width: 600px;
  width: 80%;
  height: auto;
}

.reels {
  display: flex;
  position: absolute;
  top: 300px;
  left: 300px;
  z-index: 3;
}

.reel {
  width: 100px;
  height: 180px;
  overflow: hidden;
  margin-right: 20px;
  position: relative;
}

.numbers {
  display: flex;
  flex-direction: column;
  transition: transform 2s cubic-bezier(0.25, 1, 0.5, 1);
  will-change: transform;
}

.number-img {
  width: 100px;
  height: 180px;
  object-fit: cover;
  display: block;
}

.lever {
  position: absolute;
  top: 48%;
  right: 175px;
  z-index: 1;  /* Ensure the lever is above the slot machine */
  pointer-events: auto; /* Ensure the lever is clickable even behind the machine */
  transition: transform 0.5s ease-in-out;
  
}

.lever-img {
  width: 140px;
  height: 200px;
  transition: transform 0.3s ease-in-out;
}

.lever.active .lever-img {
  transform: rotateX(30deg) rotateZ(10deg) translateY(60px);
}

</style>
