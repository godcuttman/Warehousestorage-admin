<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="选择仓库" prop="wareHouseName">
        <el-input v-model="dataForm.wareHouseName" placeholder="仓库" style="width: 70%;"></el-input>
        <el-button type="primary" @click="selectStyle('选择仓库')" style="width: 30%;">选择仓库</el-button>
      </el-form-item>
      <el-form-item label="填写SUK/面料" prop="styleType">
        <el-radio-group v-model="dataForm.styleType" @change="selectStyleTyoe">
          <el-radio :label="1" size="large">SKU</el-radio>
          <el-radio :label="2" size="large">面料</el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="SKU" prop="pattern" v-if="dataForm.styleType == 1">
        <el-input v-model="dataForm.pattern" placeholder="花色样式" style="width: 70%;"></el-input>
        <el-button type="primary" @click="selectStyle('选择花色样式')" style="width: 30%;">选择花色样式</el-button>
      </el-form-item>
      <el-form-item label="面料" prop="color">
        <el-input v-model="dataForm.color" placeholder="面料" style="width: 70%;"></el-input>
        <el-button type="primary" @click="selectStyle('选择花色面料')" style="width: 30%;">选择花色面料</el-button>
      </el-form-item>
      <el-form-item label="数量" prop="number">
        <el-input v-model="dataForm.number" placeholder="数量"></el-input>
      </el-form-item>
      <el-form-item label="单价" prop="amount">
        <el-input v-model="dataForm.amount" placeholder="单价"></el-input>
      </el-form-item>
      <!-- <el-form-item label="入库时间" prop="inboundDate">
        <el-date-picker v-model="dataForm.inboundDate" type="date" placeholder="入库时间" />
      </el-form-item> -->
    </el-form>
    <template v-slot:footer>
      <el-button @click="visible = false">{{ $t("cancel") }}</el-button>
      <el-button type="primary" @click="dataFormSubmitHandle()">{{ $t("confirm") }}</el-button>
    </template>
    <!-- 弹窗, 新增 / 修改 -->
    <SelectStyle ref="selectStyleRef" v-if="selectStyleVisiable" @cancelHandler="cancelHandler"
      @selectItem="selectItem"></SelectStyle>
  </el-dialog>

</template>

<script lang="ts" setup>
import { nextTick, reactive, ref } from "vue";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { useAppStore } from "@/store";
import { ElMessage } from "element-plus";
import { checkPermission, getDictLabel } from "@/utils/utils";
import SelectStyle from "./warehouseproducts-select.vue";

const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);
const store = useAppStore();

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  wareHouseId: "",
  wareHouseName: "",
  wareHouseSkuId: "",
  wareHouseFabricId: "",
  number: "",
  amount: "",
  inboundDate: "",
  delFlag: "",
  creator: "",
  createDate: "",
  updater: "",
  updateDate: "",
  pattern: "",
  color: "" as any,
  style: "" as any,
  styleType: 1
});

const selectStyleVisiable = ref(false);

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
const selectStyleTyoe = () => {
  dataForm.wareHouseId = "";
  dataForm.wareHouseSkuId = "";
  dataForm.wareHouseFabricId = "";
  dataForm.pattern = "";
  dataForm.color = "";
};
// 获取信息
const getInfo = (id: number) => {
  baseService.get("/sys/warehouseproducts/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

const selectStyleRef = ref();
const selectTitle = ref();
const selectStyle = (title: any) => {
  selectTitle.value = title;
  selectStyleVisiable.value = true;
  nextTick(() => {
    selectStyleRef.value.init(title);
  });
};
const cancelHandler = () => {
  selectStyleVisiable.value = false;
};
const selectItem = (list: any) => {
  if (selectTitle.value == "选择花色样式") {
    dataForm.wareHouseSkuId = list[0].id;
    dataForm.pattern = list[0].pattern;
  } else if (selectTitle.value == "选择花色面料") {
    dataForm.wareHouseFabricId = list[0].id;
    dataForm.color = dictLabel("color", list[0].color) + " X " + dictLabel("fabric_style", list[0].style);
  } else if (selectTitle.value == "选择仓库") {
    dataForm.wareHouseId = list[0].id;
    dataForm.wareHouseName = list[0].name;
  }
  selectStyleVisiable.value = false;
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/warehouseproducts", dataForm).then((res) => {
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