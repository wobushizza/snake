<template>
  <div>
    <div :style="gameBarStyle" @click="startGame">{{ status }}</div>
    <div class="a" :style="contianerStyle">
      <div v-for="(row, rowIndex) in map" :key="rowIndex" :style="rowStyle">
        <div
          v-for="col in map[rowIndex]"
          :key="col.key"
          :style="{ ...colStyle, backgroundColor: getColorByType(col.type) }"
        >
          <!-- ({{ rowIndex  }},{{ colIndex  }}) -->
        </div>
      </div>
    </div>
  </div>
</template>

<script>
// 游戏颜色定义
const colorMap = {
  food: "black",
  snakeHead: "black",
  snakeBody: "pink",
  block: "white",
};
// 游戏状态
const GAMESTATUS = {
  TO_BE_STATED: "未开始",
  STARTING: "游戏中",
  PAUSE: "暂停",
  GAME_OVER: "游戏结束",
};
// 方向
const DIR = {
  LEFT: "0",
  RIGHT: "1",
  UP: "2",
  DOWN: "3",
};
// 不同的方向对蛇头坐标的改变方法
const DIR_GOFUNC = {
  [DIR.LEFT]: ([x, y]) => [x, y - 1],
  [DIR.RIGHT]: ([x, y]) => [x, y + 1],
  [DIR.UP]: ([x, y]) => [x - 1, y],
  [DIR.DOWN]: ([x, y]) => [x + 1, y],
};
// 键盘keyCode 到方向的映射
const KEY_DIR = {
  37: DIR.LEFT,
  38: DIR.UP,
  39: DIR.RIGHT,
  40: DIR.DOWN,
};
    
export default {
  // 组件名称
  name: "tcs",
  // 组件参数 接收来自父组件的数据
  props: {
    mapSize: 0,
    boxSize: 0,
  },
  
  // 局部注册的组件
  components: {},
  // 组件状态值
  data() {
    return {
      status: GAMESTATUS.TO_BE_STATED,
      direction: DIR.LEFT,
      snake: [
        [5, 7],
        [5, 8],
        [5, 9],
        [5, 10],
      ],
      map: [],
      food: {},
      sudu:120
    };
  },
  // 计算属性
  computed: {
    gameContainerWidth() {
      const { mapSize, boxSize } = this.$props;
      return boxSize * mapSize + 'px';
    },
    // 地图行样式
    rowStyle: function () {
      const { boxSize } = this.$props;
      return {
        width: this.gameContainerWidth,
        height: boxSize + "px",
        outline: "1px solid black",
      };
    },
    // 地图列基本样式
    colStyle: function () {
      const { boxSize } = this.$props;
      return {
        width: boxSize + "px",
        height: boxSize + "px",
        outline: "1px solid black",
        float: "left",
      };
    },
    // 游戏头部
    gameBarStyle() {
      return {
        width: this.gameContainerWidth,
        height: "30px",
        backgroundColor: "pink",
        border: "1px solid deeppink",
        borderBottom: "0px",
      };
    },
    // 地图盒子
    contianerStyle() {
      return {
        width: this.gameContainerWidth,
        height: this.gameContainerWidth,
        border: "1px solid black",
      };
    },
  },
  // 侦听器
  watch: {},
  // 组件方法
  methods: {
    // 初始化游戏
    initGame() {
      this.paint();
    },
    getRandom() {
      // console.log(Math.random());
      return Math.floor(Math.random() * (this.mapSize - 1));
    },
    //随机食物的方法
    randomFood() {
      this.food = {
        x: this.getRandom(),
        y: this.getRandom(),
      };
      console.log(this.food.x);
      console.log(this.food.y);
      this.paint();
    },
    // 绘图
    paint() {
      this.map = [];
      const { mapSize } = this.$props;
      for (let rowIndex = 0; rowIndex < mapSize; rowIndex++) {
        const row = [];
        this.map.push(row);
        for (let colIndex = 0; colIndex < mapSize; colIndex++) {
          //蛇头
          const [headX, headY] = this.snake[0];
          const isSnakeHead = headX === rowIndex && headY === colIndex;
          //初始化一条蛇
          const isSnake = this.snake.some(([x, y]) => {
            return x === rowIndex && y === colIndex;
          });
          const key = `${rowIndex}-${colIndex}`;
          const { x: foodX, y: foodY } = this.food;
          const isFood = foodX === rowIndex && foodY === colIndex;

          //判断食物
          if (isFood) {
            row.push({ key, type: "food" });
          }
          //判断是否是蛇头
          else if (isSnakeHead) {
            row.push({ key, type: "snakeHead" });
          }
          //判断是否是蛇身
          else if (isSnake) {
            row.push({ key, type: "snakeBody" });
          } else {
            row.push({ key, type: "block" });
          }
        }
      }
      
    },
    
    go(direction) {
      console.log('整条蛇:',this.snake);
      const { mapSize } = this.$props;
      // 拿到老的蛇头
      const snakeHead = this.snake[0];
      // 根据老的蛇头和要去的方向，算出新的蛇头
      const newSnakeHead = DIR_GOFUNC[direction](snakeHead);
      this.snake.unshift(newSnakeHead);
      const [headX, headY] = this.snake[0];
      const { x, y } = this.food;
      if (headX === x && headY === y) {
        this.randomFood();
      } else {
        // 删除最后一个元素
        this.snake.pop();
      }
      //拿到蛇身
        const snakebody= this.snake.slice(1)
        // console.log('整条蛇:',this.snake);
        // console.log('蛇身:',snakebody);
        const issnakebody=snakebody.some(([bodyX,bodyY])=>{
          return bodyX===headX&&bodyY===headY 
        })
      //撞到墙或者自己就游戏结束
      if (headX <= -1 || headX >= mapSize || headY <= -1 || headY >= mapSize  ||issnakebody) {
        clearInterval(this.timer);
        this.status = GAMESTATUS.GAME_OVER;
        return;
      } 
      this.paint();
    },
   
    //开始游戏
    startGame() {
      console.log("游戏开始");
      clearInterval(this.timer);
      this.timer = setInterval(() => {
        this.go(this.direction);
      }, this.sudu);
      this.status = GAMESTATUS.STARTING;
    },
    // 根据type 计算不同物体的颜色
    getColorByType(type) {
      return colorMap[type];
    },
    
    keyup() {
      document.onkeyup = (e) => {
        // 监听空格
        if (e.key == " ") {
          console.log('status:',this.status)
          // 正在游戏中就去停止游戏
          if (this.status === GAMESTATUS.STARTING) {
            console.log("游戏暂停");
            clearInterval(this.timer);
            this.status = GAMESTATUS.PAUSE;
          }
          // 游戏停止状态或者未开启状态就去开启游戏
          else if (
            this.status === GAMESTATUS.PAUSE ||
            this.status === GAMESTATUS.TO_BE_STATED
          ) {
            this.startGame();
          }
          else if(this.status === GAMESTATUS.GAME_OVER){
            location.reload();
          }
        }
        // 只有在游戏中并且按方向键时才能改变方向
        if (KEY_DIR[e.keyCode] && this.status === GAMESTATUS.STARTING) {
          this.direction = KEY_DIR[e.keyCode];
        }
      };
    },
  },
  // 以下是生命周期钩子   注：没用到的钩子请自行删除
  /**
   * 组件实例创建完成，属性已绑定，但DOM还未生成，$ el属性还不存在
   */
  created() {
    this.initGame();
    this.keyup();
    
  },
  /**
   * el 被新创建的 vm.$ el 替换，并挂载到实例上去之后调用该钩子。
   * 如果 root 实例挂载了一个文档内元素，当 mounted 被调用时 vm.$ el 也在文档内。
   */
  mounted() {
    this.randomFood();
  },
};
</script> 

<style scoped lang="less">
div {
  color: gray;
}
</style>