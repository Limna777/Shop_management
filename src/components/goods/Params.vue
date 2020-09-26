<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/params' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <!-- 头部的警告头 -->
      <el-alert
        title="注意：只允许为第三级分类设置相关参数！"
        show-icon
        :closable="false"
        type="warning"
      >
      </el-alert>
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 选择商品分类的级联选择框 -->
          <el-cascader
            expand-trigger="hover"
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cateProps"
            @change="handleChangeTab"
          ></el-cascader>
        </el-col>
      </el-row>
      <!-- tab页签区域 -->
      <el-tabs
        v-model="activeName"
        @tab-click="handleTabClick"
      >
        <!-- 添加动态参数的面板 -->
        <el-tab-pane
          label="动态参数"
          name="many"
        >
          <el-button
            type="primary"
            size="small"
            :disabled="isBtnDisabled"
            @click="showAddDialogVisible"
          >添加参数</el-button>
          </el-button>
          <!-- 动态参数的表格 -->
          <el-table
            :data="manyTableData"
            border
            stripe
          >
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  @close="handleClose(i,scope.row)"
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                >{{item}}</el-tag>
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加的按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引 -->
            <el-table-column type="index">
            </el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            >
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  size="small"
                  icon="el-icon-edit"
                  @click="showEditdialog(scope.row.attr_id)"
                >修改</el-button>
                <el-button
                  type="danger"
                  size="small"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <!-- 添加静态属性的面板 -->
        <el-tab-pane
          label="静态属性"
          name="only"
        >
          <el-button
            @click="showAddDialogVisible"
            type="primary"
            size="small"
            :disabled="isBtnDisabled"
          >添加属性</el-button>
          </el-button>
          <el-table
            :data="onlyTableData"
            border
            stripe
          >
            <!-- 展开行 -->
            <el-table-column type="expand">
              <template slot-scope="scope">
                <!-- 循环渲染tag标签 -->
                <el-tag
                  @close="handleClose(i,scope.row)"
                  v-for="(item,i) in scope.row.attr_vals"
                  :key="i"
                  closable
                >{{item}}</el-tag>
                <!-- 输入文本框 -->
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)"
                >
                </el-input>
                <!-- 添加的按钮 -->
                <el-button
                  v-else
                  class="button-new-tag"
                  size="small"
                  @click="showInput(scope.row)"
                >+ New Tag</el-button>
              </template>
            </el-table-column>
            <!-- 索引 -->
            <el-table-column type="index">
            </el-table-column>
            <el-table-column
              label="参数名称"
              prop="attr_name"
            >
            </el-table-column>
            <el-table-column label="操作">
              <template slot-scope="scope">
                <el-button
                  type="primary"
                  size="small"
                  icon="el-icon-edit"
                  @click="showEditdialog(scope.row.attr_id)"
                >修改</el-button>
                <el-button
                  type="danger"
                  size="small"
                  icon="el-icon-delete"
                  @click="removeParams(scope.row.attr_id)"
                >删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数的对话框 -->
    <el-dialog
      @close="addDialogClosed"
      :title="'添加'+ titleText"
      :visible.sync="addDialogVisible"
      width="50%"
    >
      <el-form
        ref="addFormRef"
        :model="addForm"
        :rules="addFormRules"
        label-width="100px"
      >
        <el-form-item
          :label="titleText"
          prop="attr_name"
        >
          <el-input v-model="addForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="addParams"
        >确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数的对话框 -->
    <el-dialog
      @close="editDialogClosed"
      :title="'修改'+ titleText"
      :visible.sync="editDialogVisible"
      width="50%"
    >
      <el-form
        ref="editFormRef"
        :model="editForm"
        :rules="editFormRules"
        label-width="100px"
      >
        <el-form-item
          :label="titleText"
          prop="attr_name"
        >
          <el-input v-model="editForm.attr_name"></el-input>
        </el-form-item>
      </el-form>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="editParams"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: 'Params',
  data() {
    return {
      catelist: [],
      //级联选择框的配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        //父子节点的嵌套属性
        children: 'children'
      },
      // 选择框双向绑定选择的数组
      selectedCateKeys: [],
      //被激活的页签的名称
      activeName: 'many',
      //动态表格的数据
      manyTableData: [],
      //静态获取得到数据
      onlyTableData: [],
      //添加属性的对话框的显示喝隐藏
      addDialogVisible: false,
      //添加表单的数据对象
      addForm: {
        attr_name: ''
      },
      //添加表单的规则对象
      addFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      },
      //修改参数的对话框的显示与隐藏
      editDialogVisible: false,
      editForm: {
      },
      editFormRules: {
        attr_name: [{ required: true, message: '请输入参数名称', trigger: 'blur' }]
      },
      inputVisible: false,
      inputValue: ''


    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.catelist = res.data
    },
    //获取级联选择框的数据的渲染
    handleChangeTab() {
      this.getParamsData()
    },
    handleTabClick() {
      if (this.selectedCateKeys.length === 0) {
        this.$message.warning('请先选择商品分类')
      } else {
        this.getParamsData()
      }
    },
    //获取参数列表的数据
    async getParamsData() {
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, {
        params: {
          sel: this.activeName
        }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取动态参数失败！')
      }
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        item.inputVisible = false
        item.inputValue = ''
      });
      //判断获取的数据是动态的还是静态的
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    //控制添加属性或者参数的对话框
    showAddDialogVisible() {
      this.addDialogVisible = true
    },
    //监听添加参数对话框的关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    //添加参数或者属性
    addParams() {
      //先进行预验证
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          })
        if (res.meta.status !== 201) {
          this.$message.error('添加' + this.titleText + '失败')
        }
        this.$message.success('添加' + this.titleText + '成功')
        this.getParamsData()
        this.addDialogVisible = false
      })
    },
    //点击修改按钮展示对话框
    async showEditdialog(attr_id) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(
        `categories/${this.cateId}/attributes/${attr_id}`,
        {
          params: { attr_sel: this.activeName }
        }
      )
      console.log(res);
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数信息失败！')
      }

      this.editForm = res.data
      this.editDialogVisible = true
    },
    //关闭修改参数对话框的监听,重置表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    //点击确定按钮修改参数
    editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(
          `categories/${this.cateId}/attributes/${this.editForm.attr_id}`,
          { attr_name: this.editForm.attr_name, attr_sel: this.activeName }
        )
        if (res.meta.status !== 200) {
          this.$message.error('修改' + this.titleText + '失败')
        }
        this.$message.success('修改' + this.titleText + '成功')
        this.editDialogVisible = false;
        this.getParamsData()
      })
    },
    // 删除参数的点击事件
    async removeParams(attr_id) {
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(error => error)
      if (confirmResult == 'confirm') {
        const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
        if (res.meta.status !== 200) {
          this.$message.error('删除' + this.titleText + '失败')
        }
        this.$message.success('删除' + this.titleText + '成功')
        this.getParamsData();
      } else {
        this.$message.info('已取消删除')
      }
    },
    //文本框失去焦点或者回车键
    handleInputConfirm(row) {
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return
      }
      //输入了真实的内容，需要做后续处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      //需要发起请求保存到后端
      this.saveAttrVals(row)
    },
    //需要发起请求保存到后端
    async saveAttrVals(row) {
      // 需要发起请求，保存这次操作
      const { data: res } = await this.$http.put(
        `categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_name: row.attr_name,
          attr_sel: row.attr_sel,
          attr_vals: row.attr_vals.join(' ')
        }
      )

      if (res.meta.status !== 200) {
        return this.$message.error('删除' + this.titleText + '项失败！')
      }

      this.$message.success('修改' + this.titleText + '项成功！')
    },
    //点击按钮转换成文本框
    showInput(row) {
      row.inputVisible = true
      //让文本框自动获得焦点
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus();
      });
    },
    //删除tag
    handleClose(i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrVals(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true，否则返回false
    isBtnDisabled() {
      console.log(this.selectedCateKeys.length);
      if (this.selectedCateKeys.length === 0) {
        return true
      }
      return false
    },
    cateId() {
      return this.selectedCateKeys[2]
    },
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  },

}
</script>

<style scoped>
.cat_opt {
  margin: 20px 0;
}
.el-tag {
  margin: 10px;
}
.input-new-tag {
  width: 90px;
}
</style>
