<!--抽奖历史记录-->
<template>
    <div class='history-container'>
        <el-drawer size="100%" v-model="isHistoryList" title="抽奖历史" direction="btt" :before-close="handleClose"
            :with-header="false">
            <h1 class="tit">抽奖历史</h1>
            <el-icon class="close-btn" @click="handleClose" size="40px" color="#f00">
                <Close />
            </el-icon>
            <el-button class="det-btn" color="#b79900" @click="deleteLog">
                删除数据
            </el-button>
            <el-table :data="hisTab" style="width: 90vw;margin: 20px 2vw 0px 2vw;" height="80vh">
                <el-table-column type="index" label="序号" width="auto" min-width="20%" align="center" />
                <el-table-column prop="prize" label="奖项" width="auto" min-width="20%" align="center" />
                <el-table-column prop="name" label="姓名" width="auto" min-width="20%" align="center" />
                <el-table-column prop="department" label="部门" width="auto" min-width="20%" align="center" />
                <el-table-column prop="time" label="时间" width="auto" min-width="20%" align="center" />
            </el-table>
        </el-drawer>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus'
let props = defineProps({
    isHistoryList: Boolean,//抽屉状态
}),
    isHistoryList = ref(false),
    hisTab = ref([]),//表格数据
    emits = defineEmits(["close", "getNameList"]);

onMounted(() => { });

//监听对话框状态
watch(
    () => props.isHistoryList,
    (val) => {
        isHistoryList.value = val;
        if (val) {
            hisTab.value = JSON.parse(localStorage.getItem("luckDrawHis"));
        }
    }
)

//关闭对话框
const handleClose = () => {
    emits("close");
}

//删除记录数据
const deleteLog = () => {
    ElMessageBox.confirm(
        '确认删除数据?',
        {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'warning',
        }
    )
        .then(() => {
            localStorage.removeItem("luckDrawHis");//数据存入本地
            hisTab.value = [];
            ElMessage({
                type: 'success',
                message: '删除成功',
            })
        })
        .catch(() => {
            ElMessage({
                type: 'info',
                message: '取消删除',
            })
        })
}

</script>

<style lang="scss" scoped>
.history-container {
    height: 100%;
    width: 100%;
    text-align: center;

    .tit {
        position: absolute;
        left: 0;
        top: 0;
        margin: 10px;
    }


    .close-btn {
        cursor: pointer;
        position: absolute;
        margin: 10px;
        right: 0;
        top: 0;
    }

    .det-btn {
        color: #fff;
        margin: 10px 40px;
        padding: 20px;
        font-weight: 900;
        font-size: 20px;
        position: relative;
    }
}
</style>