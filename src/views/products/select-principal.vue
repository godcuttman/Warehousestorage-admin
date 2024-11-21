<template>
  <el-dialog v-model="visible" :title="title ? title : '选择负责人'" @close="cancel">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
    </el-form>
    <el-table ref="taskTableRef" :data="list" style="width: 100%" @selection-change="handleSelectionChange"
      v-if="title">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="color" label="颜色" header-align="center" align="center">
        <template v-slot="scope">
          {{ dictLabel("color", scope.row.color) }}
        </template>
      </el-table-column>
      <el-table-column prop="style" label="花色" header-align="center" align="center">
        <template v-slot="scope">
          {{ dictLabel("fabric_style", scope.row.style) }}
        </template>
      </el-table-column>
    </el-table>
    <el-table ref="taskTableRef" :data="list" style="width: 100%" @selection-change="handleSelectionChange" v-if="else">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="realName" label="姓名"></el-table-column>
      <el-table-column prop="mobile" label="手机号"></el-table-column>
    </el-table>
    <template v-slot:footer>
      <el-button @click="cancel">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="checkedPrincipal()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
import { checkPermission, getDictLabel } from "@/utils/utils";
import { useAppStore } from "@/store";
const store = useAppStore();

const { t } = useI18n();
const emit = defineEmits(["refreshDataList", "checkedPrincipal", "cancelHandler"]);

const visible = ref(false);
const dataFormRef = ref();
const taskTableRef = ref(); // 表格ref

const dataForm = reactive({
  id: "",
  userId: ""
});
const userList = ref([]);

const rules = ref({});
const title = ref();
const id = ref();

const init = (number: any, str: any) => {
  visible.value = true;
  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }
  if (str) {
    title.value = str;
    id.value = number;
  }
  getList();
};

const list = reactive([]);
// 获取负责人信息
const getList = () => {
  baseService.get(id.value ? "/sys/warehousefabric/page" : "/sys/user/page").then((res) => {
    Object.assign(list, res.data.list);
  });
};

const handleSelectionChange = (row: any) => {
  if (row.length > 1) {
    userList.value = row[1];
    taskTableRef.value.clearSelection();
    taskTableRef.value.toggleRowSelection(row.pop());
  } else {
    userList.value = row;
  }
  console.log(userList.value);
};

const checkedPrincipal = () => {
  if (userList.value.length == 0) {
    return ElMessage.warning({
      message: "请至少选择一项"
    });
  }
  if (title.value && title.value == "指定面料") {
    baseService.put("/sys/warehouseproducts/designate/fabric", { id: id.value, userId: (userList.value[0] as any).id }).then((res) => {
      ElMessage.success({
        message: t("prompt.success"),
        duration: 500,
        onClose: () => {
          visible.value = false;
        }
      });
    });
  }
  emit("checkedPrincipal", userList.value);
};
const cancel = () => {
  emit("cancelHandler");
};
// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/warehouse", dataForm).then((res) => {
      ElMessage.success({
        message: t("prompt.success"),
        duration: 500,
        onClose: () => {
          visible.value = false;
          emit("refreshDataList");
        }
      });
    });
  });
};
const dictLabel = (dictType: string, dictValue: number) => {
  return getDictLabel(store.state.dicts, dictType, dictValue);
};

defineExpose({
  init
});
</script>
<style scoped lang='scss'>
</style>