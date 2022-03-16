<template>
  <div class="app-container">
    <div class="tip">
      <p>
        可代替后台管理系统，设置的大量枚举值和配置，统一标准化管理，随时修改或增加
      </p>
    </div>

    <el-row :gutter="20">
      <!--字典数据-->
      <el-col :span="6" :xs="24">
        <div class="head-container">
          <el-input
            v-model="dictType"
            placeholder="请输入字典名称"
            clearable
            size="small"
            prefix-icon="el-icon-search"
            style="margin-bottom: 20px"
          />
        </div>
         <el-row :gutter="10" class="mb8">
          <el-col :span="1.5">
          <el-button
            type="primary"
            icon="el-icon-plus"
            size="mini"
            @click="handleAdd"
            v-hasPermi="['POST:/system/dict/type']"
            >新增</el-button
          >
        </el-col>
        <el-col :span="1.5">
          <el-button
            type="success"
            icon="el-icon-edit"
            size="mini"
            :disabled="single"
            @click="handleUpdate"
            v-hasPermi="['PUT:/system/dict/type']"
            >修改</el-button
          >
        </el-col>
        <el-col :span="1.5">
          <el-button
            type="danger"
            icon="el-icon-delete"
            size="mini"
            :disabled="single"
            @click="handleDelete"
            v-hasPermi="['DELETE:/system/dict/type']"
            >删除</el-button
          >
        </el-col>
        </el-row>
        <div class="head-container">
          <el-tree
            :data="typeList"
            :props="defaultProps"
            :expand-on-click-node="false"
            :highlight-current="true"
            :filter-node-method="filterNode"
            ref="tree"
            default-expand-all
            @node-click="handleNodeClick"
          />
        </div>
      </el-col>
      <el-col :span="18" :xs="24">
        <el-row :gutter="10" class="mb8">
          <el-col :span="1.5">
            <el-button
              type="primary"
              icon="el-icon-plus"
              size="mini"
              @click="handleDataAdd"
              v-hasPermi="['POST:/system/dict/data']"
              >新增</el-button
            >
          </el-col>
          <el-col :span="1.5">
            <el-button
              type="success"
              icon="el-icon-edit"
              size="mini"
              :disabled="dataSingle"
              @click="handleDataUpdate"
              v-hasPermi="['PUT:/system/dict/data']"
              >修改</el-button
            >
          </el-col>
          <el-col :span="1.5">
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              :disabled="multiple"
              @click="handleDataDelete"
              v-hasPermi="['DELETE:/system/dict/data']"
              >删除</el-button
            >
          </el-col>
        </el-row>
        <el-table
          v-loading="loading"
          :data="dataList"
          @selection-change="handleSelectionChange"
        >
          <el-table-column type="selection" width="55" align="center" />
          <el-table-column label="字典标签" align="center" prop="dict_label" />
          <el-table-column label="字典键值" align="center" prop="dict_value" />
          <el-table-column label="字典排序" align="center" prop="dict_sort" />
          <el-table-column
            label="状态"
            align="center"
            prop="status"
            :formatter="statusFormat"
          />
          <el-table-column
            label="备注"
            align="center"
            prop="remark"
            :show-overflow-tooltip="true"
          />
          <el-table-column
            label="创建时间"
            align="center"
            prop="create_time"
            width="180"
          >
            <template slot-scope="scope">
              <span>{{ parseTime(scope.row.create_time) }}</span>
            </template>
          </el-table-column>
          <el-table-column
            label="操作"
            align="center"
            class-name="small-padding fixed-width"
          >
            <template slot-scope="scope">
              <el-button
                size="mini"
                type="text"
                icon="el-icon-edit"
                @click="handleDataUpdate(scope.row)"
                v-hasPermi="['PUT:/system/dict/data']"
                >修改</el-button
              >
              <el-button
                size="mini"
                type="text"
                icon="el-icon-delete"
                @click="handleDataDelete(scope.row)"
                v-hasPermi="['DELETE:/system/dict/data']"
                >删除</el-button
              >
            </template>
          </el-table-column>
        </el-table>
        <pagination
          v-show="total > 0"
          :total="total"
          :page.sync="queryDataParams.pageNum"
          :limit.sync="queryDataParams.pageSize"
          @pagination="getList"
        />
      </el-col>
    </el-row>

    <!-- 添加或修改参数配置对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="500px" append-to-body>
      <el-form ref="form" :model="form" :rules="rules" label-width="80px">
        <el-form-item label="字典名称" prop="dict_name">
          <el-input v-model="form.dict_name" placeholder="请输入字典名称" />
        </el-form-item>
        <el-form-item label="字典类型" prop="dict_type">
          <el-input v-model="form.dict_type" placeholder="请输入字典类型" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio
              v-for="dict in statusOptions"
              :key="dict.dict_value"
              :label="dict.dict_value"
              >{{ dict.dict_label }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="form.remark"
            type="textarea"
            placeholder="请输入内容"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>

    <!-- 添加或修改字典参数配置对话框 -->
    <el-dialog
      :title="dataTitle"
      :visible.sync="dataOpen"
      width="500px"
      append-to-body
    >
      <el-form
        ref="dataForm"
        :model="dataForm"
        :rules="dataRules"
        label-width="80px"
      >
        <el-form-item label="字典类型">
          <el-input v-model="dataForm.dict_type" :disabled="true" />
        </el-form-item>
        <el-form-item label="数据标签" prop="dict_label">
          <el-input
            v-model="dataForm.dict_label"
            placeholder="请输入数据标签"
          />
        </el-form-item>
        <el-form-item label="数据键值" prop="dict_value">
          <el-input
            v-model="dataForm.dict_value"
            placeholder="请输入数据键值"
          />
        </el-form-item>
        <el-form-item label="显示排序" prop="dict_sort">
          <el-input-number
            v-model="dataForm.dict_sort"
            controls-position="right"
            :min="0"
          />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="dataForm.status">
            <el-radio
              v-for="dict in statusOptions"
              :key="dict.dict_value"
              :label="dict.dict_value"
              >{{ dict.dict_label }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="dataForm.remark"
            type="textarea"
            placeholder="请输入内容"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitDataForm">确 定</el-button>
        <el-button @click="dataCancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  optionselect,
  getType,
  delType,
  addType,
  updateType,
} from "@/api/system/dict/type";

import {
  listData,
  getData,
  delData,
  addData,
  updateData,
} from "@/api/system/dict/data";

export default {
  name: "Dict",
  data() {
    return {
      // 遮罩层
      loading: true,
      // 选中数组
      ids: [],
      // 选中数组
      dataIds: [],
      // 非单个禁用
      single: true,
      // 非单个禁用
      dataSingle: true,
      // 非多个禁用
      multiple: true,
      // 总条数
      total: 0,
      // 字典表格数据
      typeList: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 字典表格数据
      dataList: [],
      // 弹出层标题
      dataTitle: "",
      // 是否显示弹出层
      dataOpen: false,
      // 状态数据字典
      statusOptions: [],
      defaultProps: {
        children: "children",
        label: "label",
      },
      dictType: "",
      // 查询参数
      queryDataParams: {
        pageNum: 1,
        pageSize: 10,
        dictName: undefined,
        dictType: undefined,
        status: undefined,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        dict_name: [
          { required: true, message: "字典名称不能为空", trigger: "blur" },
        ],
        dict_type: [
          { required: true, message: "字典类型不能为空", trigger: "blur" },
        ],
      },
      // 表单参数
      dataForm: {},
      // 表单校验
      dataRules: {
        dict_label: [
          { required: true, message: "数据标签不能为空", trigger: "blur" },
        ],
        dict_value: [
          { required: true, message: "数据键值不能为空", trigger: "blur" },
        ],
        dict_sort: [
          { required: true, message: "数据顺序不能为空", trigger: "blur" },
        ],
      },
    };
  },
  watch: {
    // 根据名称筛选部门树
    dictType(val) {
      this.$refs.tree.filter(val);
    },
  },
  created() {
    this.getList();
    this.getDicts("sys_normal_disable").then((response) => {
      this.statusOptions = response.data;
    });
  },
  methods: {
    /** 查询字典类型列表 */
    getList() {
      this.loading = true;
      optionselect().then((response) => {
        // 处理树形
        if (response.data) {
          var trees = [];
          response.data.forEach((item) => {
            trees.push(this.getTree(item));
          });
          this.typeList = trees;
        }
        this.loading = false;
      });
    },
    getTree(item) {
      var tree = {};
      tree.id = item.dict_id;
      tree.label = item.dict_name + ":" + item.dict_type;
      tree.children = [];
      return tree;
    },
    // 筛选节点
    filterNode(value, data) {
      if (!value) return true;
      return data.label.indexOf(value) !== -1;
    },
    // 节点单击事件
    handleNodeClick(data) {
      this.queryDataParams.dictType = data.label.split(":")[1];
      this.getDataList();
      this.single = false
      this.ids = data.id
    },
    // 字典状态字典翻译
    statusFormat(row, column) {
      return this.selectDictLabel(this.statusOptions, row.status);
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.form = {
        dict_id: undefined,
        dict_name: undefined,
        dict_type: undefined,
        status: "0",
        remark: undefined,
      };
      this.resetForm("form");
    },
    /** 新增按钮操作 */
    handleAdd() {
      this.reset();
      this.open = true;
      this.title = "添加字典类型";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.reset();
      const dictId = row.dict_id || this.ids;
      getType(dictId).then((response) => {
        this.form = response.data;
        this.open = true;
        this.title = "修改字典类型";
      });
    },
    /** 提交按钮 */
    submitForm: function () {
      this.$refs["form"].validate((valid) => {
        if (valid) {
          if (this.form.dict_id != undefined) {
            updateType(this.form).then((response) => {
              if (response.code === 0) {
                this.msgSuccess("修改成功");
                this.open = false;
                this.getList();
              } else {
                this.msgError(response.msg);
              }
            });
          } else {
            addType(this.form).then((response) => {
              if (response.code === 0) {
                this.msgSuccess("新增成功");
                this.open = false;
                this.getList();
              } else {
                this.msgError(response.msg);
              }
            });
          }
        }
      });
    },
    /** 删除按钮操作 */
    handleDelete(row) {
      const dictIds = row.dict_id || this.ids;
      this.$confirm(
        '是否确认删除字典?',
        "警告",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      )
        .then(function () {
          return delType(dictIds);
        })
        .then(() => {
          this.getList();
          this.msgSuccess("删除成功");
        })
        .catch(function () {});
    },
    // ====== 字典值 =======
    /** 查询字典数据列表 */
    getDataList() {
      this.loading = true;
      listData(this.queryDataParams).then((response) => {
        this.dataList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 取消按钮
    dataCancel() {
      this.dataOpen = false;
      this.dataReset();
    },
    // 表单重置
    dataReset() {
      this.dataForm = {
        dict_code: undefined,
        dict_label: undefined,
        dict_value: undefined,
        dict_sort: 0,
        status: "0",
        remark: undefined,
      };
      this.resetForm("dataForm");
    },
    /** 提交按钮 */
    submitDataForm: function () {
      this.$refs["dataForm"].validate((valid) => {
        if (valid) {
          if (this.dataForm.dict_code != undefined) {
            updateData(this.dataForm).then((response) => {
              if (response.dataForm === 0) {
                this.msgSuccess("修改成功");
                this.dataOpen = false;
                this.getDataList();
              } else {
                this.msgError(response.msg);
              }
            });
          } else {
            addData(this.dataForm).then((response) => {
              if (response.code === 0) {
                this.msgSuccess("新增成功");
                this.dataOpen = false;
                this.getDataList();
              } else {
                this.msgError(response.msg);
              }
            });
          }
        }
      });
    },
    /** 新增按钮操作 */
    handleDataAdd() {
      this.dataReset();
      this.dataOpen = true;
      this.dataTitle = "添加字典数据";
      this.dataForm.dict_type = this.queryDataParams.dictType;
    },
    // 多选框选中数据
    handleSelectionChange(selection) {
      this.dataIds = selection.map((item) => item.dict_code).join(",");
      this.dataSingle = selection.length != 1;
      this.multiple = !selection.length;
    },
    /** 修改按钮操作 */
    handleDataUpdate(row) {
      this.dataReset();
      const dictCode = row.dict_code || this.dataIds;
      getData(dictCode).then((response) => {
        this.dataForm = response.data;
        this.dataOpen = true;
        this.dataTitle = "修改字典数据";
      });
    },
    /** 删除按钮操作 */
    handleDataDelete(row) {
      const dictCodes = row.dict_code || this.dataIds;
      this.$confirm(
        '是否确认删除字典编码为"' + dictCodes + '"的数据项?',
        "警告",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      )
        .then(function () {
          return delData(dictCodes);
        })
        .then(() => {
          this.getDataList();
          this.msgSuccess("删除成功");
        })
        .catch(function () {});
    },
  },
};
</script>

<style>
.tip {
  padding: 8px 16px;
  background-color: #ecf8ff;
  border-radius: 4px;
  border-left: 5px solid #50bfff;
  margin: 20px 0;
}
</style>