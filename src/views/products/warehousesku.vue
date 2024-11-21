<template>
  <div class="mod-sys__warehousesku">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item prop="pattern">
        <el-input v-model="state.dataForm.pattern" placeholder="花色样式"></el-input>
      </el-form-item>
      <el-form-item prop="type">
        <ren-select v-model="state.dataForm.type" dict-type="sku_type" placeholder="大类型"></ren-select>
      </el-form-item>
      <el-form-item prop="width">
        <el-input v-model="state.dataForm.width" placeholder="宽度"></el-input>
      </el-form-item>
      <el-form-item prop="length">
        <el-input v-model="state.dataForm.length" placeholder="长度"></el-input>
      </el-form-item>
      <el-form-item prop="piece">
        <ren-select v-model="state.dataForm.piece" dict-type="piece" placeholder="片数"></ren-select>
      </el-form-item>
      <el-form-item prop="color">
        <ren-select v-model="state.dataForm.color" dict-type="color" placeholder="颜色"></ren-select>
      </el-form-item>
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:warehousesku:save')" type="primary"
          @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:warehousesku:delete')" type="danger"
          @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="pattern" label="花色样式" header-align="center" align="center"></el-table-column>
      <el-table-column prop="skuUrl" label="花色图片" header-align="center" align="center">
        <template v-slot="scope">
          <img :src="scope.row.skuUrl" alt="" style="width: 50px;height: 50px;">
        </template>
      </el-table-column>
      <el-table-column prop="type" label="大类型" header-align="center" align="center">
        <template v-slot="scope">
          {{ state.getDictLabel("sku_type", scope.row.type) }}
        </template>
      </el-table-column>
      <el-table-column prop="width" label="尺寸" header-align="center" align="center">
        <template v-slot="scope">
          {{scope.row.width + 'X' + scope.row.length}}
        </template>
      </el-table-column>
      <el-table-column prop="width" label="宽度" header-align="center" align="center"></el-table-column>
      <el-table-column prop="length" label="长度" header-align="center" align="center"></el-table-column>
      <el-table-column prop="piece" label="片数" header-align="center" align="center">
        <template v-slot="scope">
          {{ state.getDictLabel("piece", scope.row.piece) }}
        </template>
      </el-table-column>
      <el-table-column prop="color" label="颜色" header-align="center" align="center">
        <template v-slot="scope">
          {{ state.getDictLabel("color", scope.row.color) }}
        </template>
      </el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('sys:warehousesku:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('sys:warehousesku:delete')" type="primary" link
            @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./warehousesku-add-or-update.vue";

const view = reactive({
  getDataListURL: "/sys/warehousesku/page",
  getDataListIsPage: true,
  exportURL: "/sys/warehousesku/export",
  deleteURL: "/sys/warehousesku",
  deleteIsBatch: true,
  dataForm: {
    pattern: "",
    type: "",
    width: "",
    length: "",
    piece: "",
    color: ""
  }
});

const state = reactive({ ...useView(view), ...toRefs(view) });

const addKey = ref(0);
const addOrUpdateRef = ref();
const addOrUpdateHandle = (id?: number) => {
  addKey.value++;
  nextTick(() => {
    addOrUpdateRef.value.init(id);
  });
};
</script>