<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="仓库名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="仓库名称"></el-input>
      </el-form-item>
      <el-form-item label="仓库地址" prop="address">
        <el-input v-model="dataForm.address" placeholder="仓库地址"></el-input>
      </el-form-item>
      <el-form-item label="仓库负责人" prop="userId">
        <el-input v-model="dataForm.realName" placeholder="仓库负责人" disabled style="width: 80%;"></el-input>
        <el-button type="primary" @click="selectPrincipal" style="width: 20%;">选择责任人</el-button>
      </el-form-item>
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
  </el-dialog>
</template>

<script lang="ts" setup>
import { reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
const { t } = useI18n();
const emit = defineEmits(["refreshDataList", "refreshPincipalList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  name: "",
  address: "",
  userId: "",
  realName: "",
  delFlag: "",
  creator: "",
  createDate: "",
  updater: "",
  updateDate: "",
  type: "0"
});

const rules = ref({});

const init = (id?: number) => {
  visible.value = true;
  dataForm.id = "";

  // 重置表单数据
  if (dataFormRef.value) {
    dataFormRef.value.resetFields();
  }

  if (id) {
    getInfo(id);
  }
};
const getUserId = (userList: any) => {
  dataForm.userId = userList[0].id;
  dataForm.realName = userList[0].realName;
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/sys/warehouse/" + id).then((res) => {
    Object.assign(dataForm, res.data);
    dataForm.realName = res.data.headerName;
  });
};
const selectPrincipal = () => {
  emit("refreshPincipalList");
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

defineExpose({
  init,
  getUserId
});
</script>