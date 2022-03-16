<template>
  <div class="app-container">
    <el-row :gutter="20">
      <el-col :span="8" :xs="24">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <el-form :inline="true">
              <el-form-item label="菜单名称">
                <el-input
                  v-model="queryParams.menuName"
                  placeholder="请输入菜单名称"
                  clearable
                  size="small"
                  @keyup.enter.native="handleQuery"
                />
              </el-form-item>
              <el-form-item label="状态">
                <el-select
                  v-model="queryParams.status"
                  placeholder="菜单状态"
                  clearable
                  size="small"
                >
                  <el-option
                    v-for="dict in statusOptions"
                    :key="dict.dict_value"
                    :label="dict.dict_label"
                    :value="dict.dict_value"
                  />
                </el-select>
              </el-form-item>
              <el-form-item>
                <el-button
                  type="primary"
                  icon="el-icon-search"
                  size="mini"
                  @click="handleQuery"
                  >搜索</el-button
                >
                <el-button
                  type="primary"
                  icon="el-icon-plus"
                  size="mini"
                  @click="handleAdd"
                  v-hasPermi="['POST:/system/menu']"
                  >新增</el-button
                >
              </el-form-item>
            </el-form>
          </div>
          <div class="head-container">
            <el-tree
              style="height: 430px; overflow: auto"
              v-loading="loading"
              :data="menuList"
              :props="defaultProps"
              :expand-on-click-node="false"
              :highlight-current="true"
              ref="tree"
              default-expand-all
              @node-click="handleNodeClick"
            >
              <span class="custom-tree-node" slot-scope="{ node, data }">
                <span>{{ node.label }}</span>
                <span @click.stop>
                  <el-button
                    type="text"
                    size="mini"
                    @click="() => handleAdd(data)"
                  >
                    新增
                  </el-button>
                  <el-button
                    type="text"
                    size="mini"
                    style="color:red"
                    @click="() => handleDelete(data)"
                  >
                    删除
                  </el-button>
                </span>
              </span>
            </el-tree>
          </div>
        </el-card>
      </el-col>
      <el-col :span="16" :xs="24">
        <el-card class="box-card">
          <div slot="header" class="clearfix">
            <span>编辑菜单</span>
          </div>
          <div class="tip" v-if="id == undefined">
            <p>从菜单列表选择一项后，进行编辑</p>
          </div>

          <el-form
            ref="form"
            v-loading="formLoading"
            v-if="id"
            :model="form"
            :rules="rules"
            label-width="80px"
          >
            <el-row>
              <el-col :span="24">
                <el-form-item label="上级菜单">
                  <treeselect
                    v-model="form.parent_id"
                    :options="menuOptions"
                    :normalizer="normalizer"
                    :show-count="true"
                    placeholder="选择上级菜单"
                  />
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item label="菜单类型" prop="menu_type">
                  <el-radio-group v-model="form.menu_type">
                    <el-radio label="M">目录</el-radio>
                    <el-radio label="C">菜单</el-radio>
                    <el-radio label="F">按钮</el-radio>
                  </el-radio-group>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item v-if="form.menu_type != 'F'" label="菜单图标">
                  <el-popover
                    placement="bottom-start"
                    width="460"
                    trigger="click"
                    @show="$refs['iconSelect'].reset()"
                  >
                    <IconSelect ref="iconSelect" @selected="selected" />
                    <el-input
                      slot="reference"
                      v-model="form.icon"
                      placeholder="点击选择图标"
                      readonly
                    >
                      <svg-icon
                        v-if="form.icon"
                        slot="prefix"
                        :icon-class="form.icon"
                        class="el-input__icon"
                        style="height: 32px; width: 16px"
                      />
                      <i
                        v-else
                        slot="prefix"
                        class="el-icon-search el-input__icon"
                      />
                    </el-input>
                  </el-popover>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="菜单名称" prop="menu_name">
                  <el-input
                    v-model="form.menu_name"
                    placeholder="请输入菜单名称"
                  />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item
                  v-if="form.menu_type != 'F'"
                  label="路由地址"
                  prop="path"
                >
                  <el-input v-model="form.path" placeholder="请输入路由地址" />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item label="显示排序" prop="order_num">
                  <el-input-number
                    v-model="form.order_num"
                    controls-position="right"
                    :min="0"
                  />
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item v-if="form.menu_type != 'F'" label="是否外链">
                  <el-radio-group v-model="form.is_frame">
                    <el-radio label="0">是</el-radio>
                    <el-radio label="1">否</el-radio>
                  </el-radio-group>
                </el-form-item>
              </el-col>
              <el-col :span="12" v-if="form.menu_type == 'C'">
                <el-form-item label="组件路径" prop="component">
                  <el-input
                    v-model="form.component"
                    placeholder="请输入组件路径"
                  />
                </el-form-item>
              </el-col>
              <el-col :span="24">
                <el-form-item v-if="form.menu_type != 'M'" label="权限标识">
                  <!-- <el-input v-model="form.perms" placeholder="请权限标识" maxlength="50" /> -->
                  <el-input
                    placeholder="请输入权限标识"
                    v-model="form.perms"
                    maxlength="50"
                    class="input-with-select"
                  >
                    <el-select
                      v-model="form.method"
                      slot="prepend"
                      placeholder="请选择"
                    >
                      <el-option label="GET" value="GET"></el-option>
                      <el-option label="POST" value="POST"></el-option>
                      <el-option label="PUT" value="PUT"></el-option>
                      <el-option label="DELETE" value="DELETE"></el-option>
                    </el-select>
                    <!-- <el-button slot="append" icon="el-icon-search"></el-button> -->
                  </el-input>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item v-if="form.menu_type != 'F'" label="显示状态">
                  <el-radio-group v-model="form.visible">
                    <el-radio
                      v-for="dict in visibleOptions"
                      :key="dict.dict_value"
                      :label="dict.dict_value"
                      >{{ dict.dict_label }}</el-radio
                    >
                  </el-radio-group>
                </el-form-item>
              </el-col>
              <el-col :span="12">
                <el-form-item v-if="form.menu_type != 'F'" label="菜单状态">
                  <el-radio-group v-model="form.status">
                    <el-radio
                      v-for="dict in statusOptions"
                      :key="dict.dict_value"
                      :label="dict.dict_value"
                      >{{ dict.dict_label }}</el-radio
                    >
                  </el-radio-group>
                </el-form-item>
              </el-col>
            </el-row>
            <el-form-item>
              <el-button type="primary" @click="submitForm">确 定</el-button>
              <el-button @click="cancel">重 置</el-button>
            </el-form-item>
          </el-form>
        </el-card>
      </el-col>
    </el-row>

    <!-- 添加或修改菜单对话框 -->
    <el-dialog :title="title" :visible.sync="open" width="600px" append-to-body>
      <el-form ref="addForm" :model="form" :rules="rules" label-width="80px">
        <el-row>
          <el-col :span="24">
            <el-form-item label="上级菜单">
              <treeselect
                v-model="form.parent_id"
                :options="menuOptions"
                :normalizer="normalizer"
                :show-count="true"
                placeholder="选择上级菜单"
              />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item label="菜单类型" prop="menu_type">
              <el-radio-group v-model="form.menu_type">
                <el-radio label="M">目录</el-radio>
                <el-radio label="C">菜单</el-radio>
                <el-radio label="F">按钮</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item v-if="form.menu_type != 'F'" label="菜单图标">
              <el-popover
                placement="bottom-start"
                width="460"
                trigger="click"
                @show="$refs['iconSelect'].reset()"
              >
                <IconSelect ref="iconSelect" @selected="selected" />
                <el-input
                  slot="reference"
                  v-model="form.icon"
                  placeholder="点击选择图标"
                  readonly
                >
                  <svg-icon
                    v-if="form.icon"
                    slot="prefix"
                    :icon-class="form.icon"
                    class="el-input__icon"
                    style="height: 32px; width: 16px"
                  />
                  <i
                    v-else
                    slot="prefix"
                    class="el-icon-search el-input__icon"
                  />
                </el-input>
              </el-popover>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item label="菜单名称" prop="menu_name">
              <el-input v-model="form.menu_name" placeholder="请输入菜单名称" />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item
              v-if="form.menu_type != 'F'"
              label="路由地址"
              prop="path"
            >
              <el-input v-model="form.path" placeholder="请输入路由地址" />
            </el-form-item>
          </el-col>
          <!-- <el-col :span="12">
            <el-form-item v-if="form.menu_type != 'M'" label="接口地址" prop="url">
              <el-input v-model="form.url" placeholder="请输入接口地址" />
            </el-form-item>
          </el-col> -->
          <el-col :span="12">
            <el-form-item label="显示排序" prop="order_num">
              <el-input-number
                v-model="form.order_num"
                controls-position="right"
                :min="0"
              />
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item v-if="form.menu_type != 'F'" label="是否外链">
              <el-radio-group v-model="form.is_frame">
                <el-radio label="0">是</el-radio>
                <el-radio label="1">否</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="12" v-if="form.menu_type == 'C'">
            <el-form-item label="组件路径" prop="component">
              <el-input v-model="form.component" placeholder="请输入组件路径" />
            </el-form-item>
          </el-col>
          <el-col :span="24">
            <el-form-item v-if="form.menu_type != 'M'" label="权限标识">
              <!-- <el-input v-model="form.perms" placeholder="请权限标识" maxlength="50" /> -->
              <el-input
                placeholder="请输入权限标识"
                v-model="form.perms"
                maxlength="50"
                class="input-with-select"
              >
                <el-select
                  v-model="form.method"
                  slot="prepend"
                  placeholder="请选择"
                >
                  <el-option label="GET" value="GET"></el-option>
                  <el-option label="POST" value="POST"></el-option>
                  <el-option label="PUT" value="PUT"></el-option>
                  <el-option label="DELETE" value="DELETE"></el-option>
                </el-select>
                <!-- <el-button slot="append" icon="el-icon-search"></el-button> -->
              </el-input>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item v-if="form.menu_type != 'F'" label="显示状态">
              <el-radio-group v-model="form.visible">
                <el-radio
                  v-for="dict in visibleOptions"
                  :key="dict.dict_value"
                  :label="dict.dict_value"
                  >{{ dict.dict_label }}</el-radio
                >
              </el-radio-group>
            </el-form-item>
          </el-col>
          <el-col :span="12">
            <el-form-item v-if="form.menu_type != 'F'" label="菜单状态">
              <el-radio-group v-model="form.status">
                <el-radio
                  v-for="dict in statusOptions"
                  :key="dict.dict_value"
                  :label="dict.dict_value"
                  >{{ dict.dict_label }}</el-radio
                >
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="submitForm">确 定</el-button>
        <el-button @click="cancel">取 消</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  listMenu,
  getMenu,
  delMenu,
  addMenu,
  updateMenu,
} from "@/api/system/menu";
import Treeselect from "@riophae/vue-treeselect";
import "@riophae/vue-treeselect/dist/vue-treeselect.css";
import IconSelect from "@/components/IconSelect";

export default {
  name: "Menu",
  components: { Treeselect, IconSelect },
  data() {
    return {
      id: undefined,
      // 遮罩层
      loading: false,
      formLoading: false,
      // 菜单表格树数据
      menuList: [],
      defaultProps: {
        children: "children",
        label: "menu_name",
        id: "menu_id",
      },
      // 菜单树选项
      menuOptions: [],
      // 弹出层标题
      title: "",
      // 是否显示弹出层
      open: false,
      // 显示状态数据字典
      visibleOptions: [],
      // 菜单状态数据字典
      statusOptions: [],
      // 查询参数
      queryParams: {
        menuName: undefined,
        visible: undefined,
      },
      // 表单参数
      form: {},
      // 表单校验
      rules: {
        menuName: [
          { required: true, message: "菜单名称不能为空", trigger: "blur" },
        ],
        orderNum: [
          { required: true, message: "菜单顺序不能为空", trigger: "blur" },
        ],
        path: [
          { required: true, message: "路由地址不能为空", trigger: "blur" },
        ],
      },
    };
  },
  created() {
    this.getList();
    this.getDicts("sys_show_hide").then((response) => {
      this.visibleOptions = response.data;
    });
    this.getDicts("sys_normal_disable").then((response) => {
      this.statusOptions = response.data;
    });
  },
  methods: {
    // 选择图标
    selected(name) {
      this.form.icon = name;
    },
    /** 查询菜单列表 */
    getList() {
      this.loading = true;
      listMenu(this.queryParams).then((response) => {
        this.menuList = this.handleTree(
          response.data,
          "menu_id",
          "parent_id"
        );
        this.loading = false;
      });
    },
    // 节点单击事件
    handleNodeClick(data) {
      var row = {
        menu_id: data.menu_id,
      };
      this.handleUpdate(row);
    },
    /** 转换菜单数据结构 */
    normalizer(node) {
      if (node.children && !node.children.length) {
        delete node.children;
      }
      return {
        id: node.menu_id,
        label: node.menu_name,
        children: node.children,
      };
    },
    /** 查询菜单下拉树结构 */
    getTreeselect() {
      this.menuOptions = [];
      const menu = { menu_id: 0, menu_name: "顶级菜单", children: [] };
      menu.children = JSON.parse(JSON.stringify(this.menuList));
      this.menuOptions.push(menu);
    },
    // 显示状态字典翻译
    visibleFormat(row, column) {
      if (row.menu_type == "F") {
        return "";
      }
      return this.selectDictLabel(this.visibleOptions, row.visible);
    },
    // 菜单状态字典翻译
    statusFormat(row, column) {
      if (row.menu_type == "F") {
        return "";
      }
      return this.selectDictLabel(this.statusOptions, row.status);
    },
    // 取消按钮
    cancel() {
      this.open = false;
      this.reset();
    },
    // 表单重置
    reset() {
      this.id = undefined;
      this.form = {
        menu_id: undefined,
        parent_id: 0,
        menu_name: undefined,
        icon: undefined,
        menu_type: "M",
        order_num: undefined,
        is_frame: "1",
        visible: "0",
        status: "0",
        method: "",
      };
      this.resetForm("form");
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.getList();
    },
    /** 新增按钮操作 */
    handleAdd(row) {
      this.reset();
      this.getTreeselect();
      if (row != null) {
        this.form.parent_id = row.menu_id;
      }
      this.open = true;
      this.title = "添加菜单";
    },
    /** 修改按钮操作 */
    handleUpdate(row) {
      this.getTreeselect();
      this.formLoading = true;
      getMenu(row.menu_id).then((response) => {
        this.reset();
        this.form = response.data;
        this.form.status = this.form.status + "";
        this.form.visible = this.form.visible + "";
        this.form.is_frame = this.form.is_frame + "";
        // this.open = true;
        this.title = "修改菜单";
        this.id = row.menu_id;
        this.formLoading = false;
      });
    },
    /** 提交按钮 */
    submitForm: function () {
      var form = "form"
      if (this.form.menu_id == undefined) {
        form = "addForm"
      }
      this.$refs[form].validate((valid) => {
        if (valid) {
          if (this.form.menu_id != undefined) {
            updateMenu(this.form).then((response) => {
              if (response.code === 0) {
                this.msgSuccess("修改成功");
                this.open = false;
                this.getList();
              } else {
                this.msgError(response.msg);
              }
            });
          } else {
            addMenu(this.form).then((response) => {
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
      this.$confirm(
        '是否确认删除名称为"' + row.menu_name + '"的数据项?',
        "警告",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      )
        .then(function () {
          return delMenu(row.menu_id);
        })
        .then(() => {
          this.getList();
          this.msgSuccess("删除成功");
        })
        .catch(function () {});
    },
  },
};
</script>
<style>
.input-with-select .el-input-group__prepend {
  background-color: #fff;
}
.el-select .el-input {
  width: 130px;
}
.tip {
  padding: 8px 16px;
  background-color: #ecf8ff;
  border-radius: 4px;
  border-left: 5px solid #50bfff;
  margin: 20px 0;
}
.custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
  }
</style>