<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryForm" size="small" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="服务类型编号11" prop="serviceTypeId">
        <!-- <el-input
          v-model="queryParams.serviceTypeId"
          placeholder="请输入服务类型编号"
          clearable
          @keyup.enter.native="handleQuery"/> -->
        <el-select v-model="value" placeholder="请选择服务类型">
            <el-option
              v-for="item in options"
              :key="item.value"
              :label="item.label"
              :value="item.value">
            </el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="服务图片" prop="serviceInfoPic">
        <el-input
          v-model="queryParams.serviceInfoPic"
          placeholder="请输入服务图片"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="服务开始时间" prop="serviceStartTime">
        <el-date-picker clearable
          v-model="queryParams.serviceStartTime"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="请选择服务开始时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="服务结束时间" prop="serviceEndTime">
        <el-date-picker clearable
          v-model="queryParams.serviceEndTime"
          type="date"
          value-format="yyyy-MM-dd"
          placeholder="请选择服务结束时间">
        </el-date-picker>
      </el-form-item>
      <el-form-item label="${comment}" prop="c1">
        <el-input
          v-model="queryParams.c1"
          placeholder="请输入${comment}"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="${comment}" prop="c2">
        <el-input
          v-model="queryParams.c2"
          placeholder="请输入${comment}"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item label="${comment}" prop="c3">
        <el-input
          v-model="queryParams.c3"
          placeholder="请输入${comment}"
          clearable
          @keyup.enter.native="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="el-icon-plus"
          size="mini"
          @click="handleAdd"
          v-hasPermi="['service:info:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="el-icon-edit"
          size="mini"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['service:info:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="el-icon-delete"
          size="mini"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['service:info:remove']"
        >删除</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="el-icon-download"
          size="mini"
          @click="handleExport"
          v-hasPermi="['service:info:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar :showSearch.sync="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="infoList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="服务信息编号11" align="center" prop="serviceInfoId" />
      <el-table-column label="服务类型编号" align="center" prop="serviceTypeId" />
      <el-table-column label="服务内容" align="center" prop="serviceInfoContent" />
      <el-table-column label="服务图片" align="center" prop="serviceInfoPic" />
      <el-table-column label="服务开始时间" align="center" prop="serviceStartTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.serviceStartTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="服务结束时间" align="center" prop="serviceEndTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.serviceEndTime, '{y}-{m}-{d}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="服务状态" align="center" prop="serviceStatus" />
      <el-table-column label="${comment}" align="center" prop="c1" />
      <el-table-column label="${comment}" align="center" prop="c2" />
      <el-table-column label="${comment}" align="center" prop="c3" />
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button
            size="mini"
            type="text"
            icon="el-icon-edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['service:info:edit']"
          >修改</el-button>
          <el-button
            size="mini"
            type="text"
            icon="el-icon-delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['service:info:remove']"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <pagination
      v-show="total>0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改服务信息对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="服务类型编号" prop="serviceTypeId">
          <!-- <el-input v-model="form.serviceTypeId" placeholder="请输入服务类型编号" /> -->
        <el-select v-model="form.serviceTypeId" placeholder="请选择服务类型">
            <el-option
              v-for="item in serviceTypeList"
              :key="item.serviceTypeId"
              :label="item.serviceTypeName"
              :value="item.serviceTypeId">
            </el-option>
        </el-select>

        </el-form-item>
        <el-form-item label="服务内容">
          <editor v-model="form.serviceInfoContent" :min-height="192"/>
        </el-form-item>
        <el-form-item label="服务图片" prop="serviceInfoPic">
          <el-input v-model="form.serviceInfoPic" placeholder="请输入服务图片" />
        </el-form-item>
        <el-form-item label="服务开始时间" prop="serviceStartTime">
          <el-date-picker clearable
            v-model="form.serviceStartTime"
            type="date"
            value-format="yyyy-MM-dd"
            placeholder="请选择服务开始时间">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="服务结束时间" prop="serviceEndTime">
          <el-date-picker clearable
            v-model="form.serviceEndTime"
            type="date"
            value-format="yyyy-MM-dd"
            placeholder="请选择服务结束时间">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="${comment}" prop="c1">
          <el-input v-model="form.c1" placeholder="请输入${comment}" />
        </el-form-item>
        <el-form-item label="${comment}" prop="c2">
          <el-input v-model="form.c2" placeholder="请输入${comment}" />
        </el-form-item>
        <el-form-item label="${comment}" prop="c3">
          <el-input v-model="form.c3" placeholder="请输入${comment}" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { listInfo, getInfo, delInfo, addInfo, updateInfo } from "@/api/service/info";
import { listType } from "@/api/service/type";
export default {
  name: "Info",
  data() {
    return {
      // 服务空数组
   
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 非单个禁用
      single: true,
      // 非多个禁用
      multiple: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 服务信息表格数据
      infoList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 查询参数
      queryParams: {
        pageNum: 1,
        pageSize: 10,
        serviceTypeId: null,
        serviceInfoContent: null,
        serviceInfoPic: null,
        serviceStartTime: null,
        serviceEndTime: null,
        serviceStatus: null,
        c1: null,
        c2: null,
        c3: null
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
      },
         serviceTypeList:[],
    };
  },
  created() {
    this.getList();
    this.listType();
  },
  methods: {

    // 查询服务类型列表
    listType(){
       listType().then(response=>{
        console.log(response)
        this.serviceTypeList =response.rows;
        console.log(serviceTypeList);
       });
    },
    /** 查询服务信息列表 */
    getList() {
      this.loading = true;
      listInfo(this.queryParams).then(response => {
        this.infoList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        serviceInfoId: null,
        serviceTypeId: null,
        serviceInfoContent: null,
        serviceInfoPic: null,
        serviceStartTime: null,
        serviceEndTime: null,
        serviceStatus: null,
        c1: null,
        c2: null,
        c3: null
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNum = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.resetForm("queryForm");
      this.handleQuery();
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.ids = selection.map(item => item.serviceInfoId)
      this.single = selection.length!==1
      this.multiple = !selection.length
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加服务信息";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const serviceInfoId = row.serviceInfoId || this.ids
      getInfo(serviceInfoId).then(response => {
        this.form = response.data;
        this.open = true;
        this.title = "修改服务信息";
      });
    },
    /** 提交按钮 */
    submitForm() {
      this.$refs["form"].validate(valid => {
        if (valid) {
          if (this.form.serviceInfoId != null) {
            updateInfo(this.form).then(response => {
              this.$modal.msgSuccess("修改成功");
              this.open = false;
              this.getList();
            });
          } else {
            addInfo(this.form).then(response => {
              this.$modal.msgSuccess("新增成功");
              this.open = false;
              this.getList();
            });
          }
        }
      });
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const serviceInfoIds = row.serviceInfoId || this.ids;
      this.$modal.confirm('是否确认删除服务信息编号为"' + serviceInfoIds + '"的数据项？').then(function() {
        return delInfo(serviceInfoIds);
      }).then(() => {
        this.getList();
        this.$modal.msgSuccess("删除成功");
      }).catch(() => {});
    },
    /** 导出按钮操作 */
    handleExport() {
      this.download('service/info/export', {
        ...this.queryParams
      }, `info_${new Date().getTime()}.xlsx`)
    }
  }
};
</script>
