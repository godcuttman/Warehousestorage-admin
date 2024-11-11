<template>
  <el-dialog v-model="visible" :title="!dataForm.id ? $t('add') : $t('update')" :close-on-click-modal="false" :close-on-press-escape="false">
    <el-form :model="dataForm" :rules="rules" ref="dataFormRef" @keyup.enter="dataFormSubmitHandle()" label-width="120px">
      <!-- <el-form-item label="主键" prop="id">
        <el-input v-model="dataForm.id" placeholder="主键"></el-input>
      </el-form-item> -->
      <el-form-item label="花色样式" prop="pattern">
        <el-input v-model="dataForm.pattern" placeholder="花色样式"></el-input>
      </el-form-item>
      <el-form-item label="花色图片" prop="skuUrl">
        <!-- <el-input v-model="dataForm.skuUrl" placeholder="花色图片"></el-input> -->
        <el-upload class="avatar-uploader" ref="nailRef" :action="uploadUrl" :show-file-list="false" :on-success="handleImgSuccess" :before-upload="beforeImgUpload">
          <img v-if="dataForm.profilePhoto" :src="dataForm.profilePhoto" class="avatar" />
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
        </el-upload>
      </el-form-item>
      <el-form-item label="宽度" prop="width">
        <el-input v-model="dataForm.width" placeholder="宽度"></el-input>
      </el-form-item>
      <el-form-item label="长度" prop="length">
        <el-input v-model="dataForm.length" placeholder="长度"></el-input>
      </el-form-item>
      <el-form-item label="片数" prop="piece">
        <ren-select v-model="dataForm.piece" dict-type="piece" placeholder="请选择片数"></ren-select>
      </el-form-item>
      <el-form-item label="颜色" prop="color">
        <ren-select v-model="dataForm.color" dict-type="color" placeholder="请选择颜色"></ren-select>
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
import app from "@/constants/app";
import { getToken } from "@/utils/cache";
import baseService from "@/service/baseService";
import { useI18n } from "vue-i18n";
import { ElMessage } from "element-plus";
import type { UploadInstance, UploadProps,UploadFile } from "element-plus";
const { t } = useI18n();
const emit = defineEmits(["refreshDataList"]);

const visible = ref(false);
const dataFormRef = ref();
const nailRef = ref<UploadInstance>();

const dataForm = reactive({
  id: "",
  pattern: "",
  skuUrl: "",
  width: "",
  length: "",
  delFlag: "",
  creator: "",
  createDate: "",
  updater: "",
  updateDate: "",
  piece:"",
  color: "",  // 颜色
});
const uploadUrl = ref(`${app.api}/oss/file/upload?access_token=${getToken()}`); // 图片上传地址

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
  baseService.get("/sys/syswarehousesku/" + id).then((res) => {
    Object.assign(dataForm, res.data);
  });
};
// 头像上传之前的回调
const beforeImgUpload: UploadProps["beforeUpload"] = (rawFile) => {    
  const type = ["image/jpeg", "image/jpg", "image/png", "image/svg"];
  const isJPG = type.includes(rawFile.type);
  if (!isJPG) {
    ElMessage.error("请上传jpg/png格式的图片！!");
    return false;
  } else if (rawFile.size / 1024 / 1024 > 5) {
    ElMessage.error("上传图片不能超过5MB!");
    return false;
  }
  return true;
};
// 头像上传成功后的回调
const handleImgSuccess: UploadProps["onSuccess"] = (response) => {
  dataForm.skuUrl = response.data.url;
  nailRef.value?.clearFiles();
};

// 表单提交
const dataFormSubmitHandle = () => {
  dataFormRef.value.validate((valid: boolean) => {
    if (!valid) {
      return false;
    }
    (!dataForm.id ? baseService.post : baseService.put)("/sys/syswarehousesku", dataForm).then((res) => {
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
<style lang="less" scoped>
.avatar-uploader .avatar {
  width: 200px;
  display: block;
}
// 缩略图上传样式调整
:deep(.el-form-item__content) .upload-demo {
  width: 100%;
}
:deep(.avatar-uploader) .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}
:deep(.avatar-uploader) .el-upload:hover {
  border-color: var(--el-color-primary);
}
:deep(.el-icon.avatar-uploader-icon) {
  font-size: 28px;
  color: #8c939d;
  width: 72px;
  height: 72px;
  text-align: center;
}
</style>