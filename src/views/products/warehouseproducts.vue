<template>
  <div class="mod-sys__warehouseproducts">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:warehouseproducts:save')" type="primary"
          @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:warehouseproducts:delete')" type="danger"
          @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <!-- <el-table-column prop="id" label="主键（生成的二维码）" header-align="center" align="center"></el-table-column>
      <el-table-column prop="wareHouseId" label="外键-归属仓库id" header-align="center" align="center"></el-table-column>
      <el-table-column prop="wareHouseSkuId" label="外键-skuId" header-align="center" align="center"></el-table-column>
      <el-table-column prop="wareHouseFabricId" label="外键-面料id" header-align="center" align="center"></el-table-column> -->
      <el-table-column prop="wareHouseName" label="仓库名" header-align="center" align="center"></el-table-column>
      <el-table-column prop="number" label="产品" header-align="center" align="center">
        <template v-slot="scope">
          {{ scope.row.fabricStyle ? scope.row.fabricStyle : scope.row.pattern }}
        </template>
      </el-table-column>
      <el-table-column prop="number" label="数量" header-align="center" align="center"></el-table-column>
      <el-table-column prop="amount" label="单价" header-align="center" align="center"></el-table-column>
      <el-table-column prop="createTime" label="最新入库时间" header-align="center" align="center"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="scope.row.wareHouseSkuId" type="primary" link
            @click="selectLining(scope.row.id,'指定面料')">指定面料</el-button>
          <el-button v-if="state.hasPermission('sys:warehouseproducts:delete')" type="primary" link
            @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
    <!-- 指定面料 -->
    <select-table :key="addKey" v-if="state.selectVisible" ref="selectRef" @refreshDataList="state.getDataList"
      @checkedPrincipal="checkedMianliao"></select-table>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./warehouseproducts-add-or-update.vue";
import selectTable from "./select-principal.vue";

const view = reactive({
  getDataListURL: "/sys/warehouseproducts/page",
  getDataListIsPage: true,
  exportURL: "/sys/warehouseproducts/export",
  deleteURL: "/sys/warehouseproducts",
  deleteIsBatch: true,
  dataForm: {},
  selectVisible: false
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
const selectRef = ref();
const selectLining = (id: number, title: string) => {
  state.selectVisible = true;
  nextTick(() => {
    selectRef.value.init(id, title);
  });
};
const checkedMianliao = (data: any) => {
  state.selectVisible = false;
  state.getDataList();
};
</script>