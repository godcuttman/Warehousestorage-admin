<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <!-- <el-form-item label="主键" prop="id">
        <el-input v-model="dataForm.id" placeholder="主键"></el-input>
      </el-form-item> -->
      <el-form-item label="仓库名称" prop="name">
        <el-input v-model="dataForm.name" placeholder="仓库名称"></el-input>
      </el-form-item>
      <el-form-item label="仓库地址" prop="address">
        <el-input v-model="dataForm.address" placeholder="仓库地址"></el-input>
      </el-form-item>
      <el-form-item label="仓库负责人" prop="userId">
        <el-input v-model="dataForm.userId" placeholder="仓库负责人"></el-input>
      </el-form-item>
      <!-- <el-form-item label="仓库类型 0:虚拟仓库 1:实体仓库" prop="type"> -->
      <el-form-item label="仓库类型" prop="type">
        <el-input v-model="dataForm.type" placeholder="仓库类型"></el-input>
      </el-form-item>
      <!-- <el-form-item label="删除标识  0：未删除    1：删除" prop="delFlag">
        <el-input v-model="dataForm.delFlag" placeholder="删除标识  0：未删除    1：删除"></el-input>
      </el-form-item>
      <el-form-item label="创建者" prop="creator">
        <el-input v-model="dataForm.creator" placeholder="创建者"></el-input>
      </el-form-item>
      <el-form-item label="创建时间" prop="createDate">
        <el-input v-model="dataForm.createDate" placeholder="创建时间"></el-input>
      </el-form-item>
      <el-form-item label="更新者" prop="updater">
        <el-input v-model="dataForm.updater" placeholder="更新者"></el-input>
      </el-form-item>
      <el-form-item label="更新时间" prop="updateDate">
        <el-input v-model="dataForm.updateDate" placeholder="更新时间"></el-input>
      </el-form-item> -->
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
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  name: "",
  address: "",
  userId: "",
  type: "",
  delFlag: "",
  creator: "",
  createDate: "",
  updater: "",
  updateDate: "",
});

const rules = ref({
});

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

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/sys/syswarehouse/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/syswarehouse", dataForm).then((res) => {
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
  init
});
</script>