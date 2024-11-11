<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <!-- <el-form-item label="主键（生成的二维码）" prop="id">
        <el-input v-model="dataForm.id" placeholder="主键（生成的二维码）"></el-input>
      </el-form-item>
      <el-form-item label="外键-归属仓库id" prop="wareHouseId">
        <el-input v-model="dataForm.wareHouseId" placeholder="外键-归属仓库id"></el-input>
      </el-form-item> -->
      <el-form-item label="商品名称（sku）" prop="goodsName">
        <el-input v-model="dataForm.goodsName" placeholder="商品名称（sku）"></el-input>
      </el-form-item>
      <el-form-item label="商品数量" prop="goodsNumber">
        <el-input v-model="dataForm.goodsNumber" placeholder="商品数量"></el-input>
      </el-form-item>
      <el-form-item label="商品单价" prop="goodsAmount">
        <el-input v-model="dataForm.goodsAmount" placeholder="商品单价"></el-input>
      </el-form-item>
      <el-form-item label="宽度" prop="goodsWidth">
        <el-input v-model="dataForm.goodsWidth" placeholder="宽度"></el-input>
      </el-form-item>
      <el-form-item label="长度" prop="goodsLength">
        <el-input v-model="dataForm.goodsLength" placeholder="长度"></el-input>
      </el-form-item>
      <el-form-item label="商品图片（花色样式）" prop="goodsUrl">
        <el-input v-model="dataForm.goodsUrl" placeholder="商品图片（花色样式）"></el-input>
      </el-form-item>
      <el-form-item label="入库负责人" prop="intoUser">
        <el-input v-model="dataForm.intoUser" placeholder="入库负责人"></el-input>
      </el-form-item>
      <el-form-item label="入库时间" prop="intoTime">
        <el-input v-model="dataForm.intoTime" placeholder="入库时间"></el-input>
      </el-form-item>
      <el-form-item label="出库负责人" prop="outUser">
        <el-input v-model="dataForm.outUser" placeholder="出库负责人"></el-input>
      </el-form-item>
      <el-form-item label="出库时间" prop="outTime">
        <el-input v-model="dataForm.outTime" placeholder="出库时间"></el-input>
      </el-form-item>
      <!-- <el-form-item label="货物状态 0:允许出库 1:不允许出库" prop="goodsState"> -->
      <el-form-item label="货物状态" prop="goodsState">
        <el-input v-model="dataForm.goodsState" placeholder="货物状态"></el-input>
      </el-form-item>
      <!-- <el-form-item label="删除标识  0：未删除    1：删除" prop="delFlag">
        <el-input v-model="dataForm.delFlag" placeholder="删除标识  0：未删除    1：删除"></el-input>
      </el-form-item>
      <el-form-item label="创建者-生产者" prop="creator">
        <el-input v-model="dataForm.creator" placeholder="创建者-生产者"></el-input>
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
  wareHouseId: "",
  goodsName: "",
  goodsNumber: "",
  goodsAmount: "",
  goodsWidth: "",
  goodsLength: "",
  goodsUrl: "",
  intoUser: "",
  intoTime: "",
  outUser: "",
  outTime: "",
  goodsState: "",
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
  baseService.get("/sys/syswarehousegoods/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/syswarehousegoods", dataForm).then((res) => {
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