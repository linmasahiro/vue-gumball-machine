<script setup>
import { ref, onMounted, nextTick } from "vue";

const canvasEl = ref(null);
const ball1El = ref(null);
const ball2El = ref(null);
const ball3El = ref(null);
const ball4El = ref(null);
const ball5El = ref(null);
const awardBallEl = ref(null);
const ctx = ref(null);
const timer = ref(null);
const ballList = ref([]);
const awardList = ref([]);
const isGameover = ref(false);

/**
 * Gumball
 */
class Ball {
  constructor(index, img) {
    this.r = 30; // 轉蛋圖片半徑
    this.x = this.getRandomNumber(canvasEl.value.width - this.r * 2); // 轉蛋起始X座標
    this.y = this.getRandomNumber(canvasEl.value.height - this.r * 2); // 轉蛋起始Y座標
    this.img = img; // 轉蛋圖片
    do {
      // 移動轉蛋的X軸座標
      this.speedX = this.getRandomNumber(20) - 10;
    } while (this.speedX < 5);
    do {
      // 移動轉蛋Y軸座標
      this.speedY = this.getRandomNumber(20) - 10;
    } while (this.speedY < 5);
  }

  /**
   * 取得隨機數字
   *
   * @param {number} num
   * @returns {number}
   */
  getRandomNumber(num) {
    return Math.random() * num;
  }

  /**
   * 移動轉蛋
   */
  move() {
    this.x += this.speedX;
    this.y += this.speedY;
    if (this.x > canvasEl.value.width - this.r * 2) {
      // 當轉蛋碰到右邊時，X軸座標反轉
      this.speedX = -this.speedX;
    }
    if (this.x < 0) {
      // 當轉蛋碰到左邊時，X軸座標轉正
      this.speedX = Math.abs(this.speedX);
    }
    if (this.y > canvasEl.value.height - this.r * 2) {
      // 當轉蛋碰到下面時，Y軸座標反轉
      this.speedY = -this.speedY;
    }
    if (this.y < 0) {
      // 當轉蛋碰到上面時，Y軸座標轉正
      this.speedY = Math.abs(this.speedY);
    }
    // 繪製移動後轉蛋
    ctx.value.drawImage(this.img, this.x, this.y, 60, 60);
  }
}

/**
 *  渲染轉蛋位置
 */
const renderingMovingBall = () => {
  window.clearInterval(timer.value);
  timer.value = setInterval(function () {
    // 每 20ms 重新渲染轉蛋位置
    ctx.value.clearRect(0, 0, canvasEl.value.width, canvasEl.value.height);
    for (let i = 0; i < awardList.value.length; i++) {
      awardList.value[i].move();
    }
  }, 20);
};

/**
 * 初始化轉蛋機
 */
const init = () => {
  for (let i = 0; i < ballList.value.length; i++) {
    // 把轉蛋全部放到轉蛋機中
    awardList.value[i] = new Ball(i, ballList.value[i]);
  }
  renderingMovingBall();
};

/**
 * 進行轉蛋
 */
const play = async () => {
  if (isGameover.value) {
    return;
  }
  isGameover.value = true; // 只允許轉一次
  document.getElementById("start").style.display = "none"; // 把開始按鈕隱藏
  awardBallEl.value.style.display = "block"; // 顯示轉蛋滾出區塊
  let coupon = await getRandomCoupon(); // 取得折價券
  document.getElementById("coupon" + coupon.index).style.display = "block"; // 顯示折價券
  awardList.value.splice(coupon.index, 1); // 從轉蛋機中取出一顆
  renderingMovingBall();
  awardBallEl.value.setAttribute("class", "dropBall" + (coupon.index + 1)); // 依照不同獎品顯示跳出不同轉蛋
  setTimeout(function () {
    document.getElementById("coupon").style.display = "block"; // 顯示結果
  }, 1100);
};

/**
 * Get coupon
 */
const getRandomCoupon = () => {
  let coupons = [
    { index: 0, no: "001", title: "訂單一律88折", code: "bonus12D88", w: 2 },
    { index: 1, no: "002", title: "訂單一律9折", code: "bonus12D9", w: 8 },
    { index: 2, no: "003", title: "折價券$150", code: "bonus12M150", w: 10 },
    { index: 3, no: "004", title: "折價券$100", code: "bonus12M100", w: 20 },
    { index: 4, no: "005", title: "折價券$50", code: "bonus12M50", w: 50 },
  ];
  // Calculation weight
  const totalWeight = coupons.reduce(function (pre, cur) {
    cur.startW = pre;
    return (cur.endW = pre + cur.w);
  }, 0);
  let random = Math.ceil(Math.random() * totalWeight);
  return coupons.find((coupon) => coupon.startW <= random && coupon.endW >= random);
};

onMounted(() => {
  nextTick(() => {
    if (canvasEl.value) {
      canvasEl.value.width = 200;
      canvasEl.value.height = 200;
      ctx.value = canvasEl.value.getContext("2d");
      ctx.value.beginPath();
      ctx.value.arc(0, 0, 0, 0, 2 * Math.PI);
      ballList.value = [
        ball1El.value,
        ball2El.value,
        ball3El.value,
        ball4El.value,
        ball5El.value,
      ];
      init();
    }
  });
});
</script>

<template>
  <div class="content">
    <div class="text-center">
      <img class="title" src="./assets/title.png" />
    </div>
    <div>
      <div class="machine">
        <div id="start" @click="play">
          <img src="./assets/start-btn.png" />
        </div>
        <div class="text-center">
          <canvas id="ballCanvas" ref="canvasEl"></canvas>
        </div>
      </div>
    </div>
    <div class="award">
      <span id="awardBall" ref="awardBallEl" style="display: none"></span>
    </div>
    <div style="width: 100vw; overflow: hidden">
      <div class="bg-circle"></div>
    </div>
    <div id="coupon" style="display: none">
      <div style="width: 50%; min-width: 340px; max-width: 500px; margin: 0 auto">
        <img id="coupon0" class="coupon-image" src="./assets/coupon1.png" />
        <img id="coupon1" class="coupon-image" src="./assets/coupon2.png" />
        <img id="coupon2" class="coupon-image" src="./assets/coupon3.png" />
        <img id="coupon3" class="coupon-image" src="./assets/coupon4.png" />
        <img id="coupon4" class="coupon-image" src="./assets/coupon5.png" />
      </div>
    </div>
    <div style="display: none">
      <img src="./assets/1.png" id="ball1" class="imgSrc" ref="ball1El" />
      <img src="./assets/2.png" id="ball2" class="imgSrc" ref="ball2El" />
      <img src="./assets/3.png" id="ball3" class="imgSrc" ref="ball3El" />
      <img src="./assets/4.png" id="ball4" class="imgSrc" ref="ball4El" />
      <img src="./assets/5.png" id="ball5" class="imgSrc" ref="ball5El" />
    </div>
  </div>
</template>

<style scoped>
body {
  margin: 0;
  padding: 0;
  border: none;
}
.content {
  background: top / contain no-repeat url("./assets/background.png");
  background-color: #19332b;
  padding: 0rem;
  width: 100vw;
  height: 100vh;
}
.content img {
  width: 100%;
}
.text-center{
  text-align: center;
}
.title {
  max-width: 1000px;
}
.bg-circle {
  background: #fac2fb;
  border-radius: 50%;
  width: 200vw;
  height: 400px;
  margin-left: -50vw;
  margin-top: 300px;
}
.machine {
  background: url("./assets/bg.png") top no-repeat;
  background-size: 100%;
  overflow: hidden;
  position: absolute;
  width: 300px;
  height: 430px;
  margin-left: 50%;
  -webkit-transform: translate(-50%);
  -moz-transform: translate(-50%);
  -ms-transform: translate(-50%);
  -o-transform: translate(-50%);
  transform: translate(-50%);
}
#start {
  position: absolute;
  text-align: center;
  height: 25px;
  width: 150px;
  font-size: 22px;
  margin-top: 332px;
  margin-left: 50%;
  -webkit-transform: translate(-50%);
  -moz-transform: translate(-50%);
  -ms-transform: translate(-50%);
  -o-transform: translate(-50%);
  transform: translate(-50%);
  cursor: pointer;
}
#ballCanvas {
  position: absolute;
  border: none;
  margin-top: 70px;
  -webkit-transform: translate(-50%);
  -moz-transform: translate(-50%);
  -ms-transform: translate(-50%);
  -o-transform: translate(-50%);
  transform: translate(-50%);
}
.imgSrc {
  display: none;
  position: absolute;
}
.award {
  position: absolute;
  border: none;
  width: 60px;
  height: 200px;
  margin-top: 390px;
  margin-left: 50%;
  -webkit-transform: translate(-50%);
  -moz-transform: translate(-50%);
  -ms-transform: translate(-50%);
  -o-transform: translate(-50%);
  transform: translate(-50%);
  z-index: 999;
}

.dropBall1 {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 60px;
  height: 60px;
  display: block;
  background: url("./assets/1.png") no-repeat;
  background-size: contain;
  animation: drop 1s ease-out forwards;
  -webkit-animation: drop 1s ease-out forwards;
}
.dropBall2 {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 60px;
  height: 60px;
  display: block;
  background: url("./assets/2.png") no-repeat;
  background-size: contain;
  animation: drop 1s ease-out forwards;
  -webkit-animation: drop 1s ease-out forwards;
}
.dropBall3 {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 60px;
  height: 60px;
  display: block;
  background: url("./assets/3.png") no-repeat;
  background-size: contain;
  animation: drop 1s ease-out forwards;
  -webkit-animation: drop 1s ease-out forwards;
}
.dropBall4 {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 60px;
  height: 60px;
  display: block;
  background: url("./assets/4.png") no-repeat;
  background-size: contain;
  animation: drop 1s ease-out forwards;
  -webkit-animation: drop 1s ease-out forwards;
}
.dropBall5 {
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 60px;
  height: 60px;
  display: block;
  background: url("./assets/5.png") no-repeat;
  background-size: contain;
  animation: drop 1s ease-out forwards;
  -webkit-animation: drop 1s ease-out forwards;
}

@keyframes drop {
  0% {
    transform: scale(0.7);
    box-shadow: 0 0 30px 15px #fff, 0 0 50px 30px #e2d9ce, 0 0 70px 45px #283934;
  }
  50% {
    box-shadow: 0 0 60px 30px #fff, 0 0 100px 60px #e2d9ce, 0 0 140px 90px #283934;
  }
  51% {
    box-shadow: 0 0 60px 30px #fff, 0 0 100px 60px #e2d9ce, 0 0 140px 90px #283934;
  }
  100% {
    transform: scale(1);
    box-shadow: 0 0 600px 300px #fff, 0 0 1000px 600px #e2d9ce, 0 0 1400px 900px #283934;
  }
}

@-webkit-keyframes drop {
  0% {
    -webkit-transform: scale(0.7);
    -webkit-box-shadow: 0 0 30px 15px #fff, 0 0 50px 30px #e2d9ce, 0 0 70px 45px #283934;
  }
  50% {
    -webkit-box-shadow: 0 0 60px 30px #fff, 0 0 100px 60px #e2d9ce, 0 0 140px 90px #283934;
  }
  51% {
    -webkit-box-shadow: 0 0 60px 30px #fff, 0 0 100px 60px #e2d9ce, 0 0 140px 90px #283934;
  }
  100% {
    -webkit-transform: scale(1);
    -webkit-box-shadow: 0 0 600px 300px #fff, 0 0 1000px 600px #e2d9ce,
      0 0 1400px 900px #283934;
  }
}

/* reflective line */
#awardBall {
  position: absolute;
  width: 60px;
  height: 60px;
  border-radius: 100%;
  overflow: hidden;
  border-radius: 50%;
  background-color: #fff;
}
#awardBall:before {
  content: "";
  display: block;
  position: absolute;
  width: 2px;
  height: 80px;
  top: 0;
  left: 0;
  transform: rotate(45deg);
  background: rgba(255, 255, 255, 0.5);
  animation: sheen2 5s infinite;
}
/* animates orb glow */
@keyframes award {
  0%,
  100% {
    box-shadow: 0 0 5px 2px #8fffcf;
  }
  50% {
    box-shadow: 0 0 24px 4px #8fffcf;
  }
}
/* moves light reflection */
@keyframes sheen2 {
  0%,
  100% {
    top: -75%;
    left: 15%;
    opacity: 0;
    width: 2px;
  }
  1% {
    opacity: 1;
  }
  10% {
    width: 24px;
  }
  11%,
  18% {
    top: 75%;
    left: 50%;
    width: 2px;
    opacity: 0;
  }
  20% {
    top: -75%;
    left: 15%;
    opacity: 0;
    width: 1px;
  }
  21% {
    opacity: 0.75;
  }
  30% {
    width: 6px;
  }
  31%,
  98% {
    top: 75%;
    left: 50%;
    width: 2px;
    opacity: 0;
  }
}

.coupon-image {
  display: none;
}

#coupon {
  width: 100vw;
  top: 8%;
  z-index: 9999;
  position: absolute;
}
</style>
