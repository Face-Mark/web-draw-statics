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
    <el-dialog v-model="isShowMsg">
      <!--有数据-->
      <span v-if="curMode == modeList[0]">
        <span v-if="nameList.length === 0">
          <span style="color:#f00">剩余抽奖人数不足！</span>
        </span>
        <span v-else>
          恭喜
          <span style="color:#f00">{{ curName }}</span>
          获得
          <span style="color:#f00">{{ curOpt }}</span>
        </span>
      </span>
      <span v-if="curMode == modeList[1]">
        <span v-for="item in curOkList" :key="item.name">
          恭喜
          <span style="color:#f00">{{ item.name }}</span>
          获得
          <span style="color:#f00">{{ item.prize }}</span>
          <br />
        </span>
      </span>
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
  let allNum = defNum.value.reduce((a, b) => Number(a) + Number(b));//计算总奖项数量
  if (!/^[1-9]\d*$/.test(defNum.value[0])) {
    ElMessage.error('奖项数量设置错误！');
    return;
  }
  if (tabData.length < allNum) {
    ElMessage.error('剩余抽奖人数不足！');
    return
  }
  handleStart();
  setTimeout(() => {
    let luckDrawHis = JSON.parse(localStorage.getItem("luckDrawHis")) ? JSON.parse(localStorage.getItem("luckDrawHis")) : [];// 获取抽奖历史
    let sureTime = new Date().toLocaleString()//中将时间
    let list = shuffle(tabData);// 乱序数组
    curOkList.value = [];//置空
    list.forEach((l, index) => {
      if (index < allNum) {
        let prizeName = '';//根据index、defNum与optionsData确定奖项
        if (index < Number(defNum.value[0]) + Number(defNum.value[1]) + Number(defNum.value[2]) + Number(defNum.value[3])) {
          prizeName = optionsData.value[3];
        }
        if (index < Number(defNum.value[0]) + Number(defNum.value[1]) + Number(defNum.value[2])) {
          prizeName = optionsData.value[2];
        }
        if (index < Number(defNum.value[0]) + Number(defNum.value[1])) {
          prizeName = optionsData.value[1];
        }
        if (index < Number(defNum.value[0])) {
          prizeName = optionsData.value[0];
        }
        curOkList.value.push({
          prize: prizeName,//奖项
          name: l.name,//姓名
          department: l.department,//部门
          time: sureTime
        })
      }
    })
    luckDrawHis = luckDrawHis.concat(curOkList.value);//将luckDrawHis与curOkList合并
    localStorage.setItem("luckDrawHis", JSON.stringify(luckDrawHis));//数据存入本地
    let tempList = [];//临时列表
    nameList.value.forEach(n => {
      if (!luckDrawHis.map(l => l.name).includes(n.name)) {
        tempList.push(n);// 将不在luckDrawHis中的人员添加到临时列表
      }
    })
    nameList.value = tempList;//重新赋值姓名列表
    localStorage.setItem("tabData", JSON.stringify(nameList.value));//数据存入本地
    isStop.value = true;//停止循环
    isShowMsg.value = true;//显示信息
  }, 200 * allNum);
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
  list = shuffle(list);// 乱序数组
  for (let i = 0; i < list.length; i++) {
    setTimeout(() => {
      if (!isStop.value) {
        curName.value = list[i].name;
        (i == list.length - 1) && (forNameList(nameList.value));//数组耗尽循环
      }
    }, 50 * i);
  }
}

//停止
const handleStop = () => {
  let tabData = JSON.parse(localStorage.getItem("tabData"));//取出总人员
  let luckDrawHis = JSON.parse(localStorage.getItem("luckDrawHis")) ? JSON.parse(localStorage.getItem("luckDrawHis")) : [];
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
</script>

<style lang="scss" scoped>
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
</style>