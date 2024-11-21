<template>
  <div class="mod-sys__warehouse">
    <el-form :inline="true" :model="state.dataForm" @keyup.enter="state.getDataList()">
      <el-form-item>
        <el-button @click="state.getDataList()">{{ $t("query") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="info" @click="state.exportHandle()">{{ $t("export") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:warehouse:save')" type="primary"
          @click="addOrUpdateHandle()">{{ $t("add") }}</el-button>
      </el-form-item>
      <el-form-item>
        <el-button v-if="state.hasPermission('sys:warehouse:delete')" type="danger"
          @click="state.deleteHandle()">{{ $t("deleteBatch") }}</el-button>
      </el-form-item>
    </el-form>
    <el-table v-loading="state.dataListLoading" :data="state.dataList" border
      @selection-change="state.dataListSelectionChangeHandle" style="width: 100%">
      <el-table-column type="selection" header-align="center" align="center" width="50"></el-table-column>
      <el-table-column prop="name" label="仓库名称" header-align="center" align="center"></el-table-column>
      <el-table-column prop="address" label="仓库地址" header-align="center" align="center"></el-table-column>
      <el-table-column prop="headerName" label="仓库负责人" header-align="center" align="center"></el-table-column>
      <!-- <el-table-column prop="type" label="仓库类型  0:虚拟仓库 1:实体仓库" header-align="center" align="center"></el-table-column> -->
      <el-table-column :label="$t('handle')" fixed="right" header-align="center" align="center" width="150">
        <template v-slot="scope">
          <el-button v-if="state.hasPermission('sys:warehouse:update')" type="primary" link
            @click="addOrUpdateHandle(scope.row.id)">{{ $t("update") }}</el-button>
          <el-button v-if="state.hasPermission('sys:warehouse:delete')" type="primary" link
            @click="state.deleteHandle(scope.row.id)">{{ $t("delete") }}</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination :current-page="state.page" :page-sizes="[10, 20, 50, 100]" :page-size="state.limit"
      :total="state.total" layout="total, sizes, prev, pager, next, jumper" @size-change="state.pageSizeChangeHandle"
      @current-change="state.pageCurrentChangeHandle"> </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update :key="addKey" ref="addOrUpdateRef" @refreshDataList="state.getDataList"
      @refreshPincipalList="selectPrincipal"></add-or-update>
    <!-- 选择责任人 -->
    <SelectPrincial v-if="state.principalVisiable" ref="principalRef" @checkedPrincipal="checkedPrincipal"
      @cancelHandler="cancelHandler">
    </SelectPrincial>
  </div>
</template>

<script lang="ts" setup>
import useView from "@/hooks/useView";
import { nextTick, reactive, ref, toRefs, watch } from "vue";
import AddOrUpdate from "./warehousevirtual-add-or-update.vue";
import SelectPrincial from "./select-principal.vue";

const view = reactive({
  getDataListURL: "/sys/warehouse/page",
  getDataListIsPage: true,
  exportURL: "/sys/warehouse/export",
  deleteURL: "/sys/warehouse",
  deleteIsBatch: true,
  dataForm: {
    type: "0"
  },
  principalVisiable: false
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

const principalRef = ref();
const selectPrincipal = () => {
  state.principalVisiable = true;
  nextTick(() => {
    principalRef.value.init();
  });
};
const checkedPrincipal = (userList: any) => {
  console.log(2);

  state.principalVisiable = false;
  nextTick(() => {
    addOrUpdateRef.value.getUserId(userList);
  });
};
const cancelHandler = () => {
  state.principalVisiable = false;
};
</script>