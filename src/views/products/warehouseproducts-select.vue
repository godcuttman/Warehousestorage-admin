<template>
  <el-dialog v-model="visible" :title="title" @close="cancel">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
    </el-form>
    <el-table ref="taskTableRef" :data="list" style="width: 100%" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55"></el-table-column>
      <el-table-column prop="pattern" label="花色样式" v-if="title == '选择花色样式'"></el-table-column>
      <el-table-column prop="color" label="颜色" header-align="center" align="center" v-if="title == '选择花色面料'">
        <template v-slot="scope">
          {{ dictLabel("color", scope.row.color) }}
        </template>
      </el-table-column>
      <el-table-column prop="style" label="花色" header-align="center" align="center" v-if="title == '选择花色面料'">
        <template v-slot="scope">
          {{ dictLabel("fabric_style", scope.row.style) }}
        </template>
      </el-table-column>
      <el-table-column prop="name" label="仓库名称" v-if="title == '选择仓库'"></el-table-column>
      <!-- <el-table-column prop="headerName" label="仓库负责人" v-if="title == '选择仓库'"></el-table-column> -->
      <el-table-column prop="address" label="仓库地址" v-if="title == '选择仓库'"></el-table-column>
    </el-table>
    <template v-slot:footer>
      <el-button @click="cancel">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="checkedPrincipal()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { checkPermission, getDictLabel } from "@/utils/utils";
import { useAppStore } from "@/store";
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
const { t } = useI18n();
const emit = defineEmits(["refreshDataList", "selectItem", "cancelHandler"]);
const store = useAppStore();

const visible = ref(false);
const dataFormRef = ref();
const taskTableRef = ref(); // 表格ref

const dataForm = reactive({
  id: ""
});
const selectList = ref([]);

const rules = ref({});

const title = ref();
const init = (date: any) => {
  console.log(title);

  visible.value = true;
  title.value = date;
  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }
  getList();
};

const list = reactive([]);
// 获取负责人信息
const getList = () => {
  baseService.get(title.value == "选择花色样式" ? "/sys/warehousesku/page" : title.value == "选择花色面料" ? "/sys/warehousefabric/page" : title.value == "选择仓库" ? "/sys/warehouse/page" : "").then((res) => {
    Object.assign(list, res.data.list);
  });
};

const handleSelectionChange = (row: any) => {
  if (row.length > 1) {
    selectList.value = row[1];
    taskTableRef.value.clearSelection();
    taskTableRef.value.toggleRowSelection(row.pop());
  } else {
    selectList.value = row;
  }
  console.log(selectList.value);
};

const checkedPrincipal = () => {
  if (selectList.value.length == 0) {
    return ElMessage.warning({
      message: "请选择" + title
    });
  }
  emit("selectItem", selectList.value);
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