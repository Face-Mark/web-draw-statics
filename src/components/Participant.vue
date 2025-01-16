<!--参与者表格-->
<template>
    <div class='participant-container' v-loading.fullscreen="isLoading">
        <el-drawer size="100%" v-model="isParticipant" title="参与人员" direction="btt" :before-close="handleClose"
            :with-header="false">
            <h1 class="tit">参与人员</h1>
            <el-icon class="close-btn" @click="handleClose" size="40px" color="#f00">
                <Close />
            </el-icon>
            <el-button class="imp-bt" color="#b79900" @click="downTemp">
                下载模板
            </el-button>
            <el-button class="imp-bt" color="#b79900">
                导入数据
                <input class="inp-xlsx" type="file" accept=".xls,.xlsx" @change="importData" />
            </el-button>
            <el-button class="imp-bt" color="#b79900" @click="deleteTab">
                删除数据
            </el-button>
            <el-table :data="tabDataS" style="width: 90vw;margin: 20px 2vw 0px 2vw;" height="80vh">
                <el-table-column type="index" label="序号" width="auto" min-width="25%" align="center" />
                <el-table-column prop="name" label="姓名" width="auto" min-width="25%" align="center" />
                <el-table-column prop="age" label="性别" width="auto" min-width="25%" align="center" />
                <el-table-column prop="department" label="部门" width="auto" min-width="25%" align="center" />
            </el-table>
        </el-drawer>
    </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus'
import * as XLSX from "xlsx";
import { saveAs } from 'file-saver';
let props = defineProps({
    isParticipant: Boolean,//抽屉状态
}),
    isParticipant = ref(false),
    tabData = ref([]),
    tabDataS = ref([]),//表格数据
    isLoading = ref(false),
    emits = defineEmits(["close", "getNameList"]);

onMounted(() => { });

//监听对话框状态
watch(
    () => props.isParticipant,
    (val) => {
        isParticipant.value = val;
        if (val) {
            tabDataS.value = JSON.parse(localStorage.getItem("tabData"));
        }
    }
)

//关闭对话框
const handleClose = () => {
    emits("close");
}

//下载模板
const downTemp = () => {
    let fileName = "人员模板.xlsx";//文件名
    let fileUrl = "./template/";//文件路径(路径相对index.html)
    saveAs(fileUrl + fileName, fileName);
}

//导入数据
const importData = (e) => {
    isLoading.value = true;
    let file = e.target.files[0]; //获取事件中的file对象
    let fileReader = new FileReader(); //创建文件读取器
    fileReader.onload = (event) => {
        let result = event.target.result; //获取读取的结果
        let workBook = XLSX.read(result, { type: "binary" }); //XLSX读取返回的结果
        let jsonData = XLSX.utils.sheet_to_json(
            workBook.Sheets[workBook.SheetNames[0]]
        ); //将读取结果转换为json
        tabData.value = [];
        jsonData.forEach((j) => {
            tabData.value.push({
                name: j.姓名,
                department: j.部门,
                type: j.类型 // 1-会被选中 2-不会被选中
            });
        }); //处理成需要的数据格式
        localStorage.setItem("tabData", JSON.stringify(tabData.value));//数据存入本地
        tabDataS.value = JSON.parse(localStorage.getItem("tabData"));//取出数据
        emits("getNameList", tabData);
        isLoading.value = false;
    };
    fileReader.readAsBinaryString(file); //开始读取文件
    ((document.getElementsByClassName("inp-xlsx")[0]).value = ""); //置空选中的文件
};

//删除表格
const deleteTab = () => {
    ElMessageBox.confirm(
        '确认删除数据?',
        {
            confirmButtonText: '确认',
            cancelButtonText: '取消',
            type: 'warning',
        }
    )
        .then(() => {
            let a = [];
            localStorage.setItem("tabData", JSON.stringify(a));//数据存入本地
            tabDataS.value = [];
            emits("getNameList", tabDataS);
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
.participant-container {
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

    .imp-bt {
        color: #fff;
        margin: 10px 40px;
        padding: 20px;
        font-weight: 900;
        font-size: 20px;
        position: relative;

        input {
            position: absolute;
            left: 0;
            top: 0;
            z-index: 10;
            width: 100%;
            height: 100%;
            opacity: 0;
            border: 0px;
        }
    }
}
</style>