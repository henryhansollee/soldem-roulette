<template>
  <div class="result-template d-flex flex-column align-items-center">
    <div class="d-flex">
      <div class="d-flex justify-content-center">
        <div>
          <canvas class="canvas-tag" id="canvas" width="460" height="460" />
        </div>
      </div>
      <div class="d-flex flex-column">
        <div class="input-group input-group">
          <input class="form-control result-font1" type="text" placeholder="항목을 입력하세요 :)" v-model="new_option" v-on:keyup.enter="addOptions">
          <button class="btn btn-secondary result-font1" v-on:click="addOptions">항목추가</button>
        </div>
        <b-card>
          <b-card-body
            class="p-0"
            id="nav-scroller"
            ref="content"
            style="position:relative; height:350px; overflow-y:scroll;"
          >
            <div class="column is-one-quarter" v-for="option in options" :key="option">
              <b-list-group>
                <b-list-group-item class="d-flex justify-content-between result-list-group">
                  <small class="result-font1 option-text">{{option}}</small>
                  <button class="btn btn-danger btn-sm result-font1 delete-button" v-on:click="removeOptions(option)">x</button>
                </b-list-group-item>
              </b-list-group>
            </div>
          </b-card-body>
        </b-card>
        <div class="d-flex">
          <button class="btn btn-primary result-font1 w-100 ml-1" v-on:click="spin">돌려버려</button>
          <button class="btn btn-info result-font1 w-100" @click="initRoulette">다시하기</button>
          <a class="btn btn-warning result-font1 w-100 mr-1 go-main" href="/">처음으로</a>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import VueConfetti from 'vue-confetti'
import VueRouter from 'vue-router'

Vue.use(VueRouter)
Vue.use(VueConfetti)

export default {
  name: 'ResultView',
  data() {
    return {
      options: ['꽝★', '당첨♣', '다시♥'],
      new_option: '',
      startAngle: 0,
      startAngleStart: 0,
      spinTimeout: null,
      spinArcStart: 10,
      spinTime: 0,
      spinTimeTotal: 0,
      ctx: ''
    } 
  },
  computed:{
    arc: function () {
      return Math.PI / (this.options.length / 2);
    } 
  },
  methods: {
    start() {
        this.$confetti.start({
        particles: [
          {
            type: 'heart',
            dropRate: '5',
            size: '7'
          },
          {
            type: 'circle',
            dropRate: '5',
            size: '7'
          },
        ],
        defaultColors: [
          'red',
          'pink',
          'yellow',
          // '#ba0000'
        ],
        
      })
    },
    stop() {
      this.$confetti.stop();
    },
    love() {
      this.$confetti.update({
        particles: [
          {
            type: 'heart',
          },
        ],
        defaultColors: [
          'red',
          'pink',
          '#ba0000'
        ],
        dropRate: 1
      });
    },
    byte2Hex: function (n) {
      var nybHexString = "0123456789ABCDEF";
      return String(nybHexString.substr((n >> 4) & 0x0F,1)) + nybHexString.substr(n & 0x0F,1);
    },
    RGB2Color: function (r,g,b) {
      return '#' + this.byte2Hex(r) + this.byte2Hex(g) + this.byte2Hex(b);
    },
    getColor: function (item, maxitem) {
      var phase = 0;
      var center =500;
      var width = 100;
      var frequency = Math.PI*2/maxitem;
      
      var red   = Math.sin(frequency*item+2+phase) * width + center;
      var green = Math.sin(frequency*item+0+phase) * width + center;
      var blue  = Math.sin(frequency*item+4+phase) * width + center;
      
      return this.RGB2Color(red,green,blue);
    },

    addOptions: function () {
      this.options.push(this.new_option);
      this.new_option = '';
      this.drawRouletteWheel()
    },
    removeOptions: function (option) {
      let idx = this.options.indexOf(option) || 0;
      this.options.splice(idx, 1);
      this.drawRouletteWheel()
    },

    drawRouletteWheel: function () {
      var canvas = document.getElementById("canvas");
      if (canvas.getContext) {
        var outsideRadius = 200;
        var textRadius = 160;
        var insideRadius = 125;

        this.ctx = canvas.getContext("2d");
        this.ctx.clearRect(0,0,500,500);

        // 돌림판 테두리
        this.ctx.strokeStyle = "lightgray";
        this.ctx.lineWidth = 3;

        this.ctx.font = '100% CookieRunOTF-Bold';

        for(var i = 0; i < this.options.length; i++) {
          var angle = this.startAngle + i * this.arc;
          //this.ctx.fillStyle = colors[i];
          this.ctx.fillStyle = this.getColor(i, this.options.length);

          this.ctx.beginPath();
          this.ctx.arc(250, 250, outsideRadius, angle, angle + this.arc, false);
          this.ctx.arc(250, 250, insideRadius, angle + this.arc, angle, true);
          this.ctx.stroke();
          this.ctx.fill();

          this.ctx.save();
          this.ctx.shadowOffsetX = -1;
          this.ctx.shadowOffsetY = -1;
          this.ctx.shadowBlur    = 0;
          this.ctx.shadowColor   = "rgb(220,220,220)";
          this.ctx.fillStyle = "black";
          this.ctx.translate(250 + Math.cos(angle + this.arc / 2) * textRadius, 
                        250 + Math.sin(angle + this.arc / 2) * textRadius);
          this.ctx.rotate(angle + this.arc / 2 + Math.PI / 2);
          var text = this.options[i];
          this.ctx.fillText(text, -this.ctx.measureText(text).width / 2, 0);
          this.ctx.restore();
        } 

        //Arrow
        this.ctx.fillStyle = "black";
        this.ctx.beginPath();
        this.ctx.moveTo(250 - 4, 250 - (outsideRadius + 5));
        this.ctx.lineTo(250 + 4, 250 - (outsideRadius + 5));
        this.ctx.lineTo(250 + 4, 250 - (outsideRadius - 5));
        this.ctx.lineTo(250 + 9, 250 - (outsideRadius - 5));
        this.ctx.lineTo(250 + 0, 250 - (outsideRadius - 13));
        this.ctx.lineTo(250 - 9, 250 - (outsideRadius - 5));
        this.ctx.lineTo(250 - 4, 250 - (outsideRadius - 5));
        this.ctx.lineTo(250 - 4, 250 - (outsideRadius + 5));
        this.ctx.fill();
      }
    },

    spin: function () {
      this.spinAngleStart = Math.random() * 50 + 50;
      this.spinTime = 0;
      this.spinTimeTotal = Math.random() * 3 + 4 * 2000;
      this.rotateWheel();
      setTimeout(
        this.start, 8400
      )
    },

    rotateWheel: function () {
      this.spinTime += 30;
      if(this.spinTime >= this.spinTimeTotal) {
        this.stopRotateWheel();
        return;
      }
      var spinAngle = this.spinAngleStart - this.easeOut(this.spinTime, 0, this.spinAngleStart, this.spinTimeTotal);
      this.startAngle += (spinAngle * Math.PI / 180);
      this.drawRouletteWheel();

      let _this = this
      this.spinTimeout = setTimeout(function() {
        _this.rotateWheel()
      }, 30);
    },

    stopRotateWheel: function () {
      clearTimeout(this.spinTimeout);
      var degrees = this.startAngle * 180 / Math.PI + 90;
      var arcd = this.arc * 180 / Math.PI;
      var index = Math.floor((360 - degrees % 360) / arcd);
      this.ctx.save();
      this.ctx.font = '40px SDSamliphopangche_Outline';
      var text = this.options[index]
      console.log(index, text, this.options)
      this.ctx.fillText(text, 250 - this.ctx.measureText(text).width / 2, 250 + 10);
      this.ctx.restore();
    },

    easeOut: function (t, b, c, d) {
      var ts = (t/=d)*t;
      var tc = ts*t;
      return b+c*(tc + -3*ts + 3*t);
    },
  initRoulette() {
      this.options = ['꽝★', '당첨♣', '다시♥']
      this.$confetti.stop();
      this.drawRouletteWheel()
    },
  },

  mounted() {
    this.drawRouletteWheel();
  }
}
</script>

<style>
@font-face {
  font-family: 'SDSamliphopangche_Outline';
  src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts-20-12@1.0/SDSamliphopangche_Outline.woff') format('woff');
  font-weight: normal;
  font-style: normal;
}
@font-face {
  font-family: 'CookieRunOTF-Bold';
  src: url('https://cdn.jsdelivr.net/gh/projectnoonnu/noonfonts_twelve@1.0/CookieRunOTF-Bold00.woff') format('woff');
  font-weight: normal;
  font-style: normal;
}
.result-template {
  background-color: #FFFFF3;
  margin-top: 4%;
  padding-top: 3%;
  padding-bottom: 3%;
}
.result-font1 {
  font-family: 'CookieRunOTF-Bold';
}
.result-list-group {
  font-size: 30px;
}
.delete-button {
  width: 15%;
}
.canvas-tag {
  margin-right: 190px;
}
.option-text {
  font-size: 1rem;
}
.go-main {
  color: white;
}
</style>