<template>
  <div class="app-container">
    <el-form :inline="true" :model="formInline" class="demo-form-inline">
      <el-form-item label="场地名称">
        <el-input v-model="formInline.user" placeholder="审批人"></el-input>
      </el-form-item>
      <el-form-item label="状态">
        <el-select v-model="formInline.region" placeholder="活动区域">
          <el-option label="上架" value="shanghai"></el-option>
          <el-option label="下架" value="beijing"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="onSubmit">查询</el-button>
      </el-form-item>
      <el-form-item>
        <el-button type="" @click="onSubmit">清空</el-button>
      </el-form-item>
    </el-form>
    <el-button type="primary" @click="dialogFormVisible = true"
      >新增场地
    </el-button>

    <el-table
      v-loading="listLoading"
      :data="list"
      element-loading-text="Loading"
      border
      fit
      highlight-current-row
    >
      <el-table-column align="center" label="ID" width="95">
        <template slot-scope="scope">
          {{ scope.$index }}
        </template>
      </el-table-column>
      <el-table-column label="Title">
        <template slot-scope="scope">
          {{ scope.row.title }}
        </template>
      </el-table-column>
      <el-table-column label="Author" width="110" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.author }}</span>
        </template>
      </el-table-column>
      <el-table-column label="Pageviews" width="110" align="center">
        <template slot-scope="scope">
          {{ scope.row.pageviews }}
        </template>
      </el-table-column>
      <el-table-column
        class-name="status-col"
        label="Status"
        width="110"
        align="center"
      >
        <template slot-scope="scope">
          <el-tag :type="scope.row.status | statusFilter">{{
            scope.row.status
          }}</el-tag>
        </template>
      </el-table-column>
      <el-table-column
        align="center"
        prop="created_at"
        label="Display_time"
        width="200"
      >
        <template slot-scope="scope">
          <i class="el-icon-time" />
          <span>{{ scope.row.display_time }}</span>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="currentPage4"
      :page-sizes="[100, 200, 300, 400]"
      :page-size="100"
      layout="total, sizes, prev, pager, next, jumper"
      :total="400"
    >
    </el-pagination>

    <el-dialog width="960px" title="新增场地" :visible.sync="dialogFormVisible">
      <el-form :model="courtInfo">
        <el-form-item
          label="场地名称"
          :rules="[{ required: true, message: '场地名称不能为空' }]"
          :label-width="formLabelWidth"
        >
          <el-input v-model="courtInfo.name" autocomplete="off"></el-input>
        </el-form-item>

        <el-form-item label="时段价格" :label-width="formLabelWidth">
          <el-row
            :style="{ marginBottom: '10px' }"
            :key="index"
            v-for="(item, index) in courtInfo.timeArray"
          >
            <el-time-select
              :style="{ width: '120px' }"
              v-model="courtInfo.timeArray[index].startTime"
              :picker-options="{
                start: '08:00',
                step: '01:00',
                end: '24:00',
              }"
              placeholder="选择时间"
            >
            </el-time-select>
            —
            <el-time-select
              :style="{ width: '120px', marginRight: '10px' }"
              v-model="courtInfo.timeArray[index].endTime"
              :picker-options="{
                start: '08:00',
                step: '01:00',
                end: '24:00',
              }"
              placeholder="选择时间"
            >
            </el-time-select>
            原价：
            <el-input
              :style="{ width: '120px' }"
              v-model="courtInfo.timeArray[index].price"
              autocomplete="off"
            ></el-input>
            <span :style="{ marginLeft: '10px' }">售价：</span>
            <el-input
              :style="{ width: '120px' }"
              v-model="courtInfo.timeArray[index].off_price"
              autocomplete="off"
            ></el-input>
            <el-checkbox
              v-model="courtInfo.timeArray[index].onsell"
              :style="{ marginLeft: '10px' }"
              >设为特价</el-checkbox
            >
            <el-button
              v-if="index != 0"
              style="
                margin-left: 5px;
                font-size: 20px;
                border: none;
                background: #fff;
                color: #606266;
              "
              icon="el-icon-delete"
              @click="delDays(index)"
            ></el-button>
            <el-button
              v-if="index == 0"
              style="
                margin-left: 5px;
                font-size: 20px;
                border: none;
                background: #fff;
                color: #606266;
              "
              icon="el-icon-circle-plus-outline"
              @click="addDays()"
            ></el-button>
          </el-row>
        </el-form-item>
        <el-form-item label="状态" prop="region" :label-width="formLabelWidth">
          <el-select
            :style="{ width: '100%' }"
            v-model="courtInfo.status"
            placeholder="请选择状态"
          >
            <el-option label="上架" :value="1"></el-option>
            <el-option label="下架" :value="0"></el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveCourtInfo"
          >确 定</el-button
        >
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { getList } from "@/api/table";

export default {
  filters: {
    statusFilter(status) {
      const statusMap = {
        published: "success",
        draft: "gray",
        deleted: "danger",
      };
      return statusMap[status];
    },
  },
  data() {
    return {
      list: null,
      listLoading: true,
      currentPage4: 4,
      formInline: {
        user: "",
        region: "",
      },
      dialogFormVisible: false,
      formLabelWidth: "100px",
      courtInfo: {
        name: "",
        timeArray: [{}],
        status: 1, //1上架，0下架
      },
      form: {
        name: "",
        region: "",
        date1: "",
        date2: "",
        delivery: false,
        type: [],
        resource: "",
        desc: "",
      },
    };
  },
  created() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      this.listLoading = true;
      getList().then((response) => {
        this.list = response.data.items;
        this.listLoading = false;
      });
    },
    addDays() {
      this.courtInfo.timeArray.push({});
    },
    delDays(index) {
      this.courtInfo.timeArray.splice(index, 1);
    },
    saveCourtInfo() {
      console.log('this.courtInfo',JSON.stringify(this.courtInfo))
    },
    onSubmit() {
      console.log("submit!");
    },
    handleSizeChange(val) {
      console.log(`每页 ${val} 条`);
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
    },
  },
};
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}
</style>
