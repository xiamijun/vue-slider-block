<!-- @format -->

<template>
  <div class="timeStrategyArea clearfix">
    <div class="setChooseArea">
      <RadioGroup v-model="radio">
        <Radio class="radioBtn greenBlock" label="1">选中</Radio>
        <Radio class="radioBtn" label="2">未选中</Radio>
      </RadioGroup>
    </div>
    <div class="sliderTimeArea clearfix">
      <ul class="weekDay">
        <li>时间段</li>
        <li>周一</li>
        <li>周二</li>
        <li>周三</li>
        <li>周四</li>
        <li>周五</li>
        <li>周六</li>
        <li>周日</li>
      </ul>
      <ul class="dayTime">
        <li :key='index' v-for="(dayTime,index) in dataObj.dayTimeCount">
          {{dayTime - 1}}
        </li>
      </ul>
      <ul class="chooseTimeArea">
        <li :key='index'
            v-for="(chooseBlock,index) in dataObj.chooseCount"
            :class="{ green: chooseBlock.isGreenClass }"
            :id="chooseBlock.id"
            @mousedown="clickBlock(chooseBlock.id)"
            @mouseup="clickUpBlock(chooseBlock.id)"
            ondragstart="return false">
        </li>
      </ul>
      <div class="btns">
        <button class='sureBtn' @click="statisticsTime()">确定</button>
        <button @click="removeSetTime()">清空</button>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';

const dataObj = {};
const chooseArr = [];
const numChooseObj = {};
const weeklyName = ['Mon', 'Tue', 'Wed', 'Thu', 'Fri', 'Sat', 'Sun'];
let currentLiId = '';
let index = 0;
let clickInitId = '';
for (let i = 1; i < 8; i++) {
  for (let j = 1; j < 25; j++) {
    let objStr = '' + i + j;
    let chooseObj = {};
    chooseObj.id = 'chooseBlockId' + i + j;
    chooseObj.isGreenClass = false;
    chooseArr.push(chooseObj);
    numChooseObj[objStr] = index;
    index++;
  }
}
export default {
  name: 'sliderTime',
  data() {
    return {
      dataObj,
      radio: '1'
    };
  },
  created() {
    Vue.set(this.dataObj, 'dayTimeCount', 24);
    Vue.set(this.dataObj, 'chooseCount', chooseArr);
    Vue.set(this.dataObj, 'isChooseData', chooseArr);
  },
  methods: {
    clickBlock(data) {
      let drawFlag = '';
      if (this.radio == 1) {
        drawFlag = true;
      } else if (this.radio == 2) {
        drawFlag = false;
      }
      let currentIndex = numChooseObj[data.slice(13)];
      let _this = this;
      currentLiId = data;
      clickInitId = data.slice(13);
      chooseArr[currentIndex].isGreenClass = drawFlag;
      let targetClassDom = document.getElementsByClassName('chooseTimeArea')[0];
      targetClassDom.onmousemove = function(e) {
        if (e.which == 1) {
          if (currentLiId != e.target.getAttribute('id')) {
            _this.drawBlocks(clickInitId, e.target.getAttribute('id').slice(13));
            currentLiId = e.target.getAttribute('id');
          }
        }
      };
      targetClassDom.onmouseleave = function(e) {
        targetClassDom.removeEventListener('mousemove', null, false);
      };
    },
    clickUpBlock(data) {
      let targetClassDom = document.getElementsByClassName('chooseTimeArea')[0];
      targetClassDom.removeEventListener('mousemove', null, false);
    },
    /**
     * 绘制蓝色区域
     * @param start 鼠标按下的开始地方
     * @param end 滑动时鼠标所在区域
     */
    drawBlocks(start, end) {
      let drawFlag = '';
      if (this.radio == 1) {
        drawFlag = true;
      } else if (this.radio == 1) {
        drawFlag = false;
      }
      let startX = '',
        startY = '',
        endX = '',
        endY = '';
      let smallX = '',
        smallY = '',
        bigX = '',
        bigY = '';

      startX = start.slice(0, 1) - 0;
      startY = start.slice(1) - 0;
      endX = end.slice(0, 1) - 0;
      endY = end.slice(1) - 0;
      if (startX < endX) {
        smallX = startX;
        bigX = endX;
      } else {
        smallX = endX;
        bigX = startX;
      }
      if (startY < endY) {
        smallY = startY;
        bigY = endY;
      } else {
        smallY = endY;
        bigY = startY;
      }
      for (let i = smallX; i <= bigX; i++) {
        for (let j = smallY; j <= bigY; j++) {
          let drawIndex = numChooseObj['' + i + j];
          chooseArr[drawIndex].isGreenClass = drawFlag;
        }
      }
    },
    /**
     * 处理获取的时间点
     */
    statisticsTime: function() {
      let currentRow = '';
      let idObj = {};
      let arrObj = {};
      let timeObj = {};
      let timeAllData = {};
      /**
       * 将同一行的时间点统一到一起,赋值为idObj
       */

      for (let i in chooseArr) {
        let v = chooseArr[i];
        let idStr = '';
        if (v.isGreenClass) {
          idStr = v.id.slice(13);
          if (currentRow == '') {
            currentRow = idStr.slice(0, 1);
            idObj[weeklyName[currentRow - 1]] = [];
          }
          if (currentRow != idStr.slice(0, 1)) {
            currentRow = idStr.slice(0, 1);
            idObj[weeklyName[currentRow - 1]] = [];
            idObj[weeklyName[currentRow - 1]].push(idStr);
          } else {
            idObj[weeklyName[currentRow - 1]].push(idStr);
          }
        }
      }

      for (let i in idObj) {
        let _i = i;
        let v = idObj[i];
        let currentV = v[0].slice(1);
        let conArr = [];
        arrObj[_i] = [];
        conArr.push(currentV);
        for (let key in v) {
          let _v = v[key];
          let thisV = _v.slice(1);
          if (i != 0) {
            if (thisV - currentV <= 1) {
              conArr.push(thisV);
              currentV = thisV;
            } else {
              arrObj[_i].push(conArr);
              conArr = [];
              conArr.push(thisV);
              currentV = thisV;
            }
          }
        }
        arrObj[_i].push(conArr);
      }

      for (let i in arrObj) {
        let _i = i;
        let thisVobj = arrObj[i];
        timeObj[_i] = [];
        for (let key in thisVobj) {
          let v = thisVobj[key];
          let str = v[0] - 1 < 10 ? '0' + (v[0] - 1) + ':00-' : v[0] - 1 + ':00-';
          str += v[v.length - 1] < 10 ? '0' + v[v.length - 1] + ':00' : v[v.length - 1] + ':00';
          timeObj[_i].push(str);
        }
      }
      console.log(timeObj);
      return timeObj;
    },
    /**
     * 清空时间策略
     */
    removeSetTime() {
      for (let i = 1; i <= 7; i++) {
        for (let j = 1; j <= 24; j++) {
          let drawIndex = numChooseObj['' + i + j];
          chooseArr[drawIndex].isGreenClass = false;
        }
      }
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.timeStrategyArea {
  background-color: #f9fbff;
  margin-top: 0.2rem;
  height: 500px;
  width: 100%;
  position: relative;
  padding-left: 1.5rem;
  padding-top: 0.44rem;
  box-sizing: border-box;
  border: 0.01rem solid #dce5f1;
}
.sliderTimeArea {
  position: relative;
  width: 100%;
  height: 100%;
}
.title {
  position: absolute;
  top: 1.4rem;
  left: 0.54rem;
  height: 0.96rem;
  width: 0.31rem;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.05rem;
  background-color: #1fbf85;
  color: #ffffff;
  font-size: 0.14rem;
  z-index: 99;
}
.title:after {
  content: '';
  width: 0;
  height: 0;
  position: absolute;
  left: 0.27rem;
  top: 0.39rem;
  border-top: solid 0.1rem transparent;
  border-left: solid 0.1rem #1fbf85;
  border-bottom: solid 0.1rem transparent;
}
.isOnlyShow {
  background-color: #f5f5f5 !important;
  color: #999;
}
.isOnlyShow:after {
  border-left: solid 0.1rem #f5f5f5;
}
.timeStrategyTitle:before {
  content: '';
  display: inline-block;
  position: absolute;
  top: 0rem;
  left: 0.7rem;
  height: 100%;
  width: 1px;
  background-color: #dce5f1;
  z-index: 50;
}
ul {
  margin: 0;
  padding: 0;
  list-style: none;
}

.weekDay {
  position: absolute;
  text-align: center;
}

.weekDay li {
  height: 0.3rem;
  width: 0.6rem;
  line-height: 0.3rem;
  border: 0.01rem solid #dbdbdb;
  border-top: none;
  color: #333333;
  box-sizing: border-box;
}

.weekDay li:first-of-type {
  border-top: 1px solid #dbdbdb;
  background: #f5f5f5;
}

.dayTime {
  position: absolute;
  left: 0.6rem;
}

.dayTime li {
  float: left;
  width: 0.3rem;
  height: 0.3rem;
  line-height: 0.3rem;
  border: 1px solid #dbdbdb;
  background-color: #f5f5f5;
  border-left: none;
  color: #666666;
  box-sizing: border-box;
  text-align: center;
}

.chooseTimeArea {
  position: absolute;
  width: 7.2rem;
  height: 2.1rem;
  left: 0.6rem;
  top: 0.3rem;
}

.chooseTimeArea li {
  width: 0.3rem;
  height: 0.3rem;
  box-sizing: border-box;
  border: 1px solid #dbdbdb;
  float: left;
  border-left: none;
  border-top: none;
}

.chooseTimeArea li.green {
  background-color: #41b883;
}

button {
  position: absolute;
  bottom: -0.2rem;
}
.setChooseArea {
  position: absolute;
  top: 0.2rem;
}
.radioBtn {
  margin-left: 0.2rem;
  position: relative;
}
.radioBtn:after {
  content: '';
  width: 0.12rem;
  height: 0.05rem;
  position: absolute;
  top: 0.02rem;
  right: -0.15rem;
  border: 0.01rem solid #dce5f1;
  background-color: #fff;
}
.greenBlock:after {
  background-color: #41b883;
}
.setChooseArea span {
  display: inline-block;
  position: relative;
  color: #fff;
  width: 0.44rem;
  height: 0.18rem;
  line-height: 0.18rem;
  border-radius: 0.09rem;
  text-align: center;
}
.wholeLevel {
  background-color: #f5ae39;
}
.schoolLevel {
  background-color: #3eb9ef;
}
.classLevel {
  background-color: #6fce84;
}
.userLevel {
  background-color: #7291d6;
}
.starChoose {
  color: #ff9933;
  font-size: 0.14rem;
}
.mess {
  color: #9f9fa0;
  margin: 0.2rem 0;
  margin-right: 0.2rem;
}
.radioArea {
  position: absolute;
  width: 100%;
  height: 100%;
  top: 0;
  left: 0.55rem;
  z-index: 400;
}
.btns button {
  position: absolute;
  width: 50px;
  left: 0;
}
.btns button:last-child {
  left: 80px;
}
</style>
