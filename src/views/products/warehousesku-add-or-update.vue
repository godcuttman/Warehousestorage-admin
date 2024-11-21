<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false"
    :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()"
      label-width="120px">
      <el-form-item label="花色样式" prop="pattern">
        <el-input v-model="dataForm.pattern" placeholder="花色样式"></el-input>
      </el-form-item>
      <el-form-item label="花色图片" prop="skuUrl">
        <el-upload class="avatar-uploader" action="http://192.168.110.84:8080/oss/file/upload" :show-file-list="false"
          :on-success="handleAvatarSuccess" :before-upload="beforeAvatarUpload">
          <img v-if="dataForm.skuUrl" :src="dataForm.skuUrl" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon">
            <Plus />
          </el-icon>
        </el-upload>
      </el-form-item>
      <el-form-item label="大类型" prop="type">
        <ren-select v-model="dataForm.type" dict-type="sku_type" placeholder="大类型"></ren-select>
      </el-form-item>
      <el-form-item label="宽度" prop="width">
        <el-input v-model="dataForm.width" placeholder="宽度"></el-input>
      </el-form-item>
      <el-form-item label="长度" prop="length">
        <el-input v-model="dataForm.length" placeholder="长度"></el-input>
      </el-form-item>
      <el-form-item label="片数" prop="piece">
        <ren-select v-model="dataForm.piece" dict-type="piece" placeholder="片数"></ren-select>
      </el-form-item>
      <el-form-item label="颜色" prop="color">
        <ren-select v-model="dataForm.color" dict-type="color" placeholder="颜色"></ren-select>
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
import { Plus } from "@element-plus/icons-vue";
import type { UploadProps } from "element-plus";
const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();

const dataForm = reactive({
  id: "",
  pattern: "",
  skuUrl: "",
  type: "",
  width: "",
  length: "",
  piece: "",
  color: "",
  delFlag: "",
  creator: "",
  createDate: "",
  updater: "",
  updateDate: ""
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

const handleAvatarSuccess: UploadProps["onSuccess"] = (response, uploadFile) => {
  dataForm.skuUrl = URL.createObjectURL(uploadFile.raw!);
};

const beforeAvatarUpload: UploadProps["beforeUpload"] = (rawFile) => {
  const imgType = ["image/jpeg", "image/jpg", "image/png", "image/gif"];
  if (imgType.indexOf(rawFile.type) == -1) {
    ElMessage.error("图片格式需为：jpg/jpeg/png/gif");
    return false;
  } else if (rawFile.size / 1024 / 1024 > 2) {
    ElMessage.error("图片需大于2M");
    return false;
  }
  return true;
};

// 获取信息
const getInfo = (id: number) => {
  baseService.get("/sys/warehousesku/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/warehousesku", dataForm).then((res) => {
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

<style>
.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  text-align: center;
}
img {
  width: 178px;
  height: 178px;
}
</style>