<template>
  <div class="mod-sys__syswarehousegoods">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:syswarehousegoods:save')" type="primary" @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:syswarehousegoods:delete')" type="danger" @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <!-- <el-table-column prop="id" label="主键（生成的二维码）" header-align="center" align="center"></el-table-column> -->
      <!-- <el-table-column prop="wareHouseId" label="外键-归属仓库id" header-align="center" align="center"></el-table-column> -->
      <!-- <el-table-column prop="goodsName" label="商品名称（sku）" header-align="center" align="center"></el-table-column> -->
      <el-table-column prop="goodsName" label="商品名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="goodsNumber" label="商品数量" header-align="center" align="center"></el-table-column>
      <el-table-column prop="goodsAmount" label="商品单价" header-align="center" align="center"></el-table-column>
      <el-table-column prop="goodsWidth" label="宽度" header-align="center" align="center"></el-table-column>
      <el-table-column prop="goodsLength" label="长度" header-align="center" align="center"></el-table-column>
      <el-table-column prop="goodsUrl" label="商品图片（花色样式）" header-align="center" align="center"></el-table-column>
      <el-table-column prop="intoUser" label="入库负责人" header-align="center" align="center"></el-table-column>
      <el-table-column prop="intoTime" label="入库时间" header-align="center" align="center"></el-table-column>
      <el-table-column prop="outUser" label="出库负责人" header-align="center" align="center"></el-table-column>
      <el-table-column prop="outTime" label="出库时间" header-align="center" align="center"></el-table-column>
      <!-- <el-table-column prop="goodsState" label="货物状态 0:允许出库 1:不允许出库" header-align="center" align="center"></el-table-column> -->
      <el-table-column prop="goodsState" label="货物状态" header-align="center" align="center"></el-table-column>
      <!-- <el-table-column prop="delFlag" label="删除标识  0：未删除    1：删除" header-align="center" align="center"></el-table-column>
      <el-table-column prop="creator" label="创建者-生产者" header-align="center" align="center"></el-table-column> -->
      <el-table-column prop="createDate" label="创建时间" header-align="center" align="center"></el-table-column>
      <!-- <el-table-column prop="updater" label="更新者" header-align="center" align="center"></el-table-column> -->
      <el-table-column prop="updateDate" label="更新时间" header-align="center" align="center"></el-table-column>
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('sys:syswarehousegoods:update')" type="primary" link @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('sys:syswarehousegoods:delete')" type="primary" link @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit" :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle" @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"></add-or-update>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./syswarehousegoods-add-or-update.vue";

const view = reactive({
  getDataListURL: "/sys/syswarehousegoods/page",
  getDataListIsPage: true,
  exportURL: "/sys/syswarehousegoods/export",
  deleteURL: "/sys/syswarehousegoods",
  deleteIsBatch: true,
  dataForm: {
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