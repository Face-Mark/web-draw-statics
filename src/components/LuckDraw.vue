<!--抽奖组件-->
<template>
  <div class='container' v-loading.fullscreen.lock="isStop == false && curMode == modeList[1]"
    element-loading-text="自动抽奖中...">
    <div class="show-name" @click="clickBtn">
      <span v-if="nameList.length == 0">
        {{ noDataTip }}
      </span>
      <span v-else>
        {{ curName }}
      </span>
    </div>
    <!--控制盒子-->
    <el-form class="control-div">
      <!-- <el-form-item label="抽奖模式：">
        <el-radio-group v-model="curMode">
          <el-radio-button v-for="item in modeList" :key="item" :label="item" />
        </el-radio-group>
      </el-form-item> -->
      <!-- <el-form-item label="奖项切换：" v-show="curMode == modeList[0]">
        <el-radio-group v-model="curOpt">
          <el-radio-button v-for="item in optionsData" :key="item" :label="item" />
        </el-radio-group>
      </el-form-item> -->
      <el-form-item class="def-num" label="奖项数量：" v-show="curMode == modeList[1]">
        <span class="def-num-sp">
          胸口碎大石：<el-input class="def-num-input" type="number" v-model="defNum[0]" @change="changeDefNum" />
        </span>
      </el-form-item>
      <el-form-item label="参与人员：">
        <el-button class="btn" color="#b79900" @click="isParticipant = true">参与人员</el-button>
      </el-form-item>
      <el-form-item label="抽奖历史：">
        <el-button class="btn" color="#b79900" @click="isHistoryList = true">抽奖历史</el-button>
      </el-form-item>
    </el-form>
    <!--提示对话框-->
    <el-dialog 
      v-model="isShowMsg"
      :show-close="false"
      width="500px"
      align-center
      class="custom-dialog"
    >
      <div class="dialog-content">
        <!--有数据-->
        <div v-if="curMode == modeList[0]" class="result-box">
          <div v-if="nameList.length === 0" class="warning-text">
            <i class="el-icon-warning"></i>
            <span>剩余抽奖人数不足！</span>
          </div>
          <div v-else class="success-text">
            <div class="congrats">恭喜</div>
            <div class="winner-name">{{ curName }}</div>
            <div class="prize-text">
              获得 <span class="prize-name">{{ curOpt }}</span>
            </div>
          </div>
        </div>
        <div v-if="curMode == modeList[1]" class="multi-result">
          <div v-for="item in curOkList" :key="item.name" class="winner-item">
            <div class="congrats">恭喜</div>
            <div class="winner-info">
              <span class="winner-name">{{ item.name }}</span>
              <span class="prize-text">获得</span>
              <span class="prize-name">{{ item.prize }}</span>
            </div>
          </div>
        </div>
        <el-button class="confirm-btn" @click="isShowMsg = false">确定</el-button>
      </div>
    </el-dialog>
    <!--组件-->
    <div class="com-div">
      <Participant :isParticipant="isParticipant" @close="isParticipant = false" @getNameList="getNameList" />
      <HistoryList :isHistoryList="isHistoryList" @close="isHistoryList = false" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted, toRaw } from 'vue';
import { ElMessage } from 'element-plus'
import Participant from './Participant.vue';
import HistoryList from './HistoryList.vue';
let curName = ref("请开始"),
  optionsData = ref([
    "胸口碎大石",
  ]),//奖项
  modeList = ref(['手动抽奖', '自动抽奖']),
  curMode = ref("手动抽奖"),//当前模式
  curOpt = ref("胸口碎大石"),//当前选中
  defNum = ref([1]),//自动模式默认奖项数量
  curOkList = ref([]),//当前自动模式中奖的人员
  noDataTip = "无数据",
  isStop = ref(true),//默认停止
  isShowMsg = ref(false),
  isParticipant = ref(false),//人员弹窗状态
  isHistoryList = ref(false),
  nameList = ref([]);//名字列表

onMounted(() => {
  let tabList = JSON.parse(localStorage.getItem("tabData"));
  tabList && (nameList.value = tabList);
});

const getNameList = (data) => {
  nameList = data;
}

//点击按钮
const clickBtn = () => {
  if (curMode.value === modeList.value[1]) {
    autoHandle();
    return;
  }
  if (isStop.value) {
    handleStart();
  } else {
    handleStop();
  }
}

//修改奖项数量
const changeDefNum = (val) => {
  let allNum = defNum.value.reduce((a, b) => Number(a) + Number(b));
  if (!/^[1-9]\d*$/.test(val) || (allNum > nameList.value.length)) {
    ElMessage.error('输入错误！必须输入正整数且奖项总数不能大于总参与人数！！');
    return;
  }
}

//自动处理
const autoHandle = () => {
  let tabData = JSON.parse(localStorage.getItem("tabData"));//取出总人员
  let availableCount = tabData.filter(person => person.type !== 2).length; // 计算可选人数
  let allNum = defNum.value.reduce((a, b) => Number(a) + Number(b));//计算总奖项数量
  
  if (!/^[1-9]\d*$/.test(defNum.value[0])) {
    ElMessage.error('奖项数量设置错误！');
    return;
  }
  if (availableCount < allNum) {
    ElMessage.error('可选人数不足！');
    return;
  }
  
  handleStart();
  setTimeout(() => {
    let luckDrawHis = JSON.parse(localStorage.getItem("luckDrawHis")) ? JSON.parse(localStorage.getItem("luckDrawHis")) : [];
    let sureTime = new Date().toLocaleString();
    let winners = [];
    let shuffledList = shuffle([...tabData]);
    
    // 选出足够的获奖者
    for (let i = 0; winners.length < allNum && i < shuffledList.length; i++) {
      if (shuffledList[i].type !== 2) {
        winners.push(shuffledList[i]);
      }
    }
    
    curOkList.value = winners.map(winner => ({
      prize: optionsData.value[0],
      name: winner.name,
      department: winner.department,
      time: sureTime
    }));
    
    // 更新历史记录和剩余名单
    luckDrawHis.push(...curOkList.value);
    let tempList = tabData.filter(person => 
      !winners.some(w => w.name === person.name)
    );
    
    localStorage.setItem("luckDrawHis", JSON.stringify(luckDrawHis));
    localStorage.setItem("tabData", JSON.stringify(tempList));
    nameList.value = tempList;
    isStop.value = true;
    isShowMsg.value = true;
  }, 2000);
}

//点击开始
const handleStart = () => {
  if (nameList.value.length == 0) {
    isShowMsg.value = true;
    // showMsg.value = "请先导入参与人员数据！";
  } else {
    isStop.value = false;
    forNameList(nameList.value);//循环数组
  }
}

//洗牌算法(乱序数组)
const shuffle = (arr) => {
  let l = arr.length
  let index, temp
  while (l > 0) {
    index = Math.floor(Math.random() * l)
    temp = arr[l - 1]
    arr[l - 1] = arr[index]
    arr[index] = temp
    l--
  }
  return arr;
}

//循环列表
const forNameList = (list) => {
  const shuffledList = shuffle([...list]);// 乱序数组的副本
  
  for (let i = 0; i < shuffledList.length; i++) {
    setTimeout(() => {
      if (!isStop.value) {
        curName.value = shuffledList[i].name;
        (i == shuffledList.length - 1) && (forNameList(nameList.value));//数组耗尽循环
      }
    }, 50 * i);
  }
}

//停止
const handleStop = () => {
  let tabData = JSON.parse(localStorage.getItem("tabData"));//取出总人员
  let luckDrawHis = JSON.parse(localStorage.getItem("luckDrawHis")) ? JSON.parse(localStorage.getItem("luckDrawHis")) : [];
  
  // 检查当前选中的人是否可以被抽中
  const currentPerson = tabData.find(t => t.name === curName.value);
  if (currentPerson?.type === 2) {
    // 如果是不可选的人，继续抽奖并顺延到下一个可选的人
    forNameListOnce(nameList.value);
    return;
  }

  // 记录中奖历史
  tabData.forEach(t => {
    if (t.name == curName.value) {
      luckDrawHis.push({
        prize: curOpt.value,//奖项
        name: curName.value,//姓名
        department: t.department,//部门
        time: new Date().toLocaleString()//时间
      })
    }
  })
  localStorage.setItem("luckDrawHis", JSON.stringify(luckDrawHis));//数据存入本地
  
  // 从名单中移除中奖者
  let tempList = [];//临时列表
  nameList.value.forEach(n => {
    if (n.name !== curName.value) {
      tempList.push(n);
    }
  })
  nameList.value = tempList;//重新赋值姓名列表
  localStorage.setItem("tabData", JSON.stringify(nameList.value));//数据存入本地
  isStop.value = true;//停止循环
  isShowMsg.value = true;
}

// 新增一个只执行一次的抽奖函数
const forNameListOnce = (list) => {
  const shuffledList = shuffle([...list]);// 乱序数组的副本
  let nextValidPerson = null;
  
  // 从当前名字开始，找到下一个可选的人
  let currentIndex = shuffledList.findIndex(person => person.name === curName.value);
  if (currentIndex === -1) currentIndex = 0;
  
  // 从当前位置往后找
  for (let i = currentIndex + 1; i < shuffledList.length; i++) {
    if (shuffledList[i].type !== 2) {
      nextValidPerson = shuffledList[i];
      break;
    }
  }
  
  // 如果没找到，从头开始找
  if (!nextValidPerson) {
    for (let i = 0; i < currentIndex; i++) {
      if (shuffledList[i].type !== 2) {
        nextValidPerson = shuffledList[i];
        break;
      }
    }
  }
  
  if (nextValidPerson) {
    curName.value = nextValidPerson.name;
    // 递归调用 handleStop 处理新选中的人
    handleStop();
  } else {
    // 如果没有可选的人了，显示提示
    ElMessage.error('抽奖人数超出预定人数，请重新导入名单并清理中奖历史');
    isStop.value = true;
  }
}
</script>

<style lang="scss">
:deep(.el-form-item__label) {
  color: #fff;
}

:deep(.el-radio-button__inner) {
  background-color: #b79900;
  font-weight: 900;
  border: 0px solid #f00;
  color: #fff;

  &:hover {
    border: 0px solid #f00;
    color: #fff
  }
}

:deep(.el-radio-button__original-radio:checked+.el-radio-button__inner) {
  background-color: #fd3c3c;
  color: #fff;
  border: 0px solid #f00;
}

.container {
  height: 100vh;
  width: 100vw;
  background-image: url(../assets/img/bg1.png);
  background-size: cover;
  background-position: center;
  display: flex;

  .show-name {
    height: 50vh;
    width: 50vh;
    background-color: rgba(255, 255, 255, .2);
    box-shadow: 0px 0px 20px black;
    margin: auto;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 15vh;
    font-weight: 900;
    color: #fff;
    border-radius: 100%;
    user-select: none;

    &:hover {
      background-color: rgba(255, 255, 255, .3);
      box-shadow: 0px 0px 50px black;
    }
  }

  .control-div {
    padding: 2vh;
    // height: 25vh;
    width: 30vw;
    position: absolute;
    right: 0;
    bottom: 0;
    background-color: rgba(0, 0, 0, .2);

    .option-div {
      width: 100%;
      display: flex;
      justify-content: left;
      margin: 10vh auto;
    }

    .def-num {
      width: 100%;
      color: #ffffff;
      display: flex;

      .def-num-sp {
        background-color: #b79900;
        width: calc(50% - 10px);
        padding: 5px;

        .def-num-input {
          width: 45%;
        }
      }
    }

    .btn {
      color: #fff;
    }
  }
}

.custom-dialog {
  border-radius: 20px;
  :deep(.el-dialog) {
    background: rgba(255, 255, 255, 0.95);
    backdrop-filter: blur(10px);
    
    box-shadow: 0 0 20px rgba(0, 0, 0, 0.2);
    
    .el-dialog__body {
      padding: 30px;
    }
  }
  
  .dialog-content {
    text-align: center;
    
    .result-box {
      padding: 20px 0;
      
      .warning-text {
        color: #ff4949;
        font-size: 24px;
        font-weight: bold;
        
        i {
          margin-right: 10px;
        }
      }
      
      .success-text {
        .congrats {
          font-size: 28px;
          color: #333;
          margin-bottom: 20px;
        }
        
        .winner-name {
          font-size: 36px;
          color: #b79900;
          font-weight: bold;
          margin: 15px 0;
        }
        
        .prize-text {
          font-size: 24px;
          color: #333;
          
          .prize-name {
            color: #b79900;
            font-weight: bold;
            margin-left: 10px;
          }
        }
      }
    }
    
    .multi-result {
      max-height: 400px;
      overflow-y: auto;
      padding: 10px 0;
      
      .winner-item {
        margin-bottom: 20px;
        padding: 15px;
        background: rgba(183, 153, 0, 0.1);
        border-radius: 10px;
        
        &:last-child {
          margin-bottom: 0;
        }
        
        .congrats {
          font-size: 20px;
          color: #333;
          margin-bottom: 10px;
        }
        
        .winner-info {
          display: flex;
          justify-content: center;
          align-items: center;
          gap: 10px;
          
          .winner-name {
            font-size: 24px;
            color: #b79900;
            font-weight: bold;
          }
          
          .prize-text {
            font-size: 20px;
            color: #333;
          }
          
          .prize-name {
            font-size: 24px;
            color: #b79900;
            font-weight: bold;
          }
        }
      }
    }
    
    .confirm-btn {
      margin-top: 30px;
      padding: 12px 40px;
      font-size: 18px;
      background-color: #b79900;
      border-color: #b79900;
      color: white;
      border-radius: 25px;
      transition: all 0.3s;
      
      &:hover {
        background-color: darken(#b79900, 10%);
        transform: translateY(-2px);
        box-shadow: 0 5px 15px rgba(183, 153, 0, 0.3);
      }
      
      &:active {
        transform: translateY(0);
      }
    }
  }
}

// 添加滚动条样式
.multi-result {
  &::-webkit-scrollbar {
    width: 6px;
  }
  
  &::-webkit-scrollbar-track {
    background: #f1f1f1;
    border-radius: 3px;
  }
  
  &::-webkit-scrollbar-thumb {
    background: #b79900;
    border-radius: 3px;
    
    &:hover {
      background: darken(#b79900, 10%);
    }
  }
}
</style>