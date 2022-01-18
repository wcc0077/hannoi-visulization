<template>
  <div class="container">
    <h1>Hannoi problem</h1>
    <!-- 控制区 -->
    <div class="control">
      <el-button 
        type="primary" 
        @click="start"
        :disabled="!isDone"
      >
        {{'开始'}}
      </el-button>
      <el-select 
        v-model="panziNum" 
        placeholder="请选择" 
        @change="handleNumChange"
        :disabled="!isDone"
      >
        <el-option
          v-for="item in options"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <div>
        速度：
      </div>
      <el-slider
        v-model="speed"
        :show-tooltip="true"
      >
      </el-slider>
    </div>

    <!-- 展示区 -->
    <div class="show">

      <!-- 底座 -->
      <div class="item" ref="left">
        <!-- <div class="stack"></div> -->
        <div class="label">左</div>
      </div>
      <div class="item" ref="mid">
        <!-- <div class="stack"></div> -->
        <div class="label">中</div>
      </div>
      <div class="item" ref="right">
        <!-- <div class="stack"></div> -->
        <div class="label">右</div>
      </div>

      <!-- 盘子 -->
      
      <PanZi 
        v-for="(panzi,index) in panziList"  
        :key="index"
        :panzi="panzi"
      />
       
    </div>

    <div class="name">by wangcheng</div>
  </div>
</template>

<script>
import PanZi from './PanZi.vue'

export default {
  components:{
    PanZi
  },
  data() {
    return {
      isDone: true,
      speed: 10,
      options: [
        {value: 1, label:1},
        {value: 2, label:2},
        {value: 3, label:3},
        {value: 4, label:4},
        {value: 5, label:5},
        {value: 6, label:6},
        {value: 7, label:7}
      ],
      panziNum: 3,
      panziHeight: 30,
      panziWidthStep: 30,
      firstPanZiWidth: 220,
      deltX : 240,
      panziList: [],
      left:{
        panziList:[],
        origin:{ // 桩底部中心点
          x: '',
          y: ''
        }
      }, 
      right:{
        panziList:[],
        origin:{
          x: '',
          y: ''
        }
      },
      mid:{
        panziList:[],
        origin:{
          x: '',
          y: ''
        }
      }, 
    }
  },

  mounted(){
    this.init() 
  },

  methods:{
    // 初始化盘子,桩
    init(){
      // 获取随机颜色
      function getRandColor(){
        let c = () => ~~(Math.random()*255)
        return `rgb(${c()} ,${c()} , ${c()})`
      }

      let { left, bottom, width } = this.$refs.left.getBoundingClientRect();

      // 初始化桩
      const initZhuang = () => {

        this.left.origin = {
          x: left + width / 2,
          y: bottom - 4
        }

        this.right.origin = {
          x: left + width / 2 + this.deltX * 2,
          y: bottom - 4
        }

        this.mid.origin = {
          x: left + width / 2 + this.deltX,
          y: bottom - 4
        }
      }
      
      // 获取盘子位置与大小
      const getPanZiStyle = (n) => {
        
        let origin = {
          x: left + width / 2 ,
          y: bottom - n * this.panziHeight - 4
        }

        let panziWidth = this.firstPanZiWidth - n * this.panziWidthStep

        return {
          left: origin.x - panziWidth / 2 + 'px',
          top: origin.y  + 'px',
          height: this.panziHeight + 'px',
          width: panziWidth + 'px',
          color: getRandColor(),
          n
        }
      }

      this.panziList = []
      for(let i = 1 ; i <= this.panziNum ; i++){
        this.panziList.push(getPanZiStyle(i))
      }

      this.left.panziList = [...this.panziList]
      
      initZhuang()
    },

    // 递归处理汉诺塔问题
    async hannoi(n, from, to, mid){
      if(n === 1){
        await this.move(from, to)
      }else{
        await this.hannoi(n-1, from, mid, to)
        await this.move(from, to)
        await this.hannoi(n-1, mid, to, from)
      }      
    },

    // 处理盘子数目的改变
    handleNumChange(){
      this.clear()
      this.init()
    },


    async move(from, to){

      const currentPanzi = from.panziList.pop()
      if(!currentPanzi) {
        return
      }

      let fromHeight = from.origin.y - from.panziList.length * this.panziHeight
      let toHeight = to.origin.y - (to.panziList.length + 1) * this.panziHeight
      
      const startPoint = {
        x: from.origin.x,
        y: fromHeight
      }

      const endPoint = {
        x: to.origin.x,
        y: toHeight
      }

      const a = Math.abs(startPoint.x - endPoint.x) // 椭圆半径a
      const b = Math.abs(startPoint.y - endPoint.y)  // 椭圆半径b

      // 椭圆曲线方程
      function y(x){  
        return Math.pow((1 - x**2/(a**2)), 0.5) * b
      }

      const moveOnePanzi = (currentPanzi) => {

        // promise包裹一个异步任务
        return new Promise(resolve => {
          const currentPanziWidth = currentPanzi.width.replace('px', '') * 1
          const step = this.speed
          let count = 0 // 移动的步数

          let rafId = requestAnimationFrame(function animate(){

            if(a - step * count <step){
              currentPanzi.top = endPoint.y 
              currentPanzi.left = endPoint.x - currentPanziWidth / 2

              cancelAnimationFrame(rafId)

              to.panziList.push(currentPanzi)
              resolve()
              return
            }

            const upstep = y( a - step * count) // 向上移动的距离

            const top = startPoint.y - upstep * Math.sign(startPoint.y - endPoint.y)
            const left = startPoint.x - step * count * Math.sign(startPoint.x - endPoint.x) - currentPanziWidth/2
            currentPanzi.top = top + 'px'
            currentPanzi.left = left + 'px'
                        
            count++

            requestAnimationFrame(animate)
          })
        })
        
      }

      await moveOnePanzi(currentPanzi)

    },

    // 开始
    async start(){
      this.clear()
      this.init()
      this.isDone = false
      await this.hannoi(this.panziNum, this.left, this.right, this.mid)
      this.isDone = true
    },

    // 暂停
    pause(){

    },

    // 结束
    terminate(){

    },

    clear(){
      this.isDone = true
      this.left.panziList = []
      this.right.panziList = []
      this.mid.panziList = []
    }

  }
}
</script>

<style scoped>
  .container {
    position: relative;
    width: 800px;
    height: 600px;
    border: 1px solid #ccc;
    /* left: 50%; */
    /* top:50%; */
    /* transform: translate(-50%, -50%); */
    margin: 0 auto;
    padding: 10px 5% 5% 5%;
  }

  .show {
    width: 90%;
    height: 60%;
    /* background: green; */
    display: flex;
    position: relative;
    margin-top: 80px;
  }

  .item {
    flex: 1;
    height: 100%;
    border-bottom: 4px solid rgb(95, 172, 245);
    margin-right: 20px;
  }

  .stack {
    height: 100%;
    width: 50%;
    border-right: 2px solid blue;
  }

  .label {
    width: 15px;
    margin: 0 auto;
  }

  .name {
    position: absolute;
    right: 10px;
    bottom: 10px;
  }
</style>