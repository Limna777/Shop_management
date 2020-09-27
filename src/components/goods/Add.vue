<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/goods' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 消息提示 -->
    <el-card>

      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <!-- 步骤条 -->
      <el-steps
        :space="200"
        :active="activeIndex-0"
        align-center
        finish-status="success"
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- 侧边导航 -->
      <template>
        <el-form
          :model="addForm"
          ref="addFormRef"
          :rules="addFormRules"
          label-width="100px"
          label-position="top"
        >
          <el-tabs
            tabPosition="left"
            v-model="activeIndex"
            :before-leave="beforeTabLeave"
            @tab-click="tabClicked"
          >
            <el-tab-pane
              label="基本信息"
              name="0"
            >
              <!-- 基本信息表单 -->
              <el-form-item
                label="商品名称"
                prop="goods_name"
              >
                <el-input v-model="addForm.goods_name"></el-input>
              </el-form-item>
              <el-form-item
                label="商品价格"
                prop="goods_price"
                type="number"
              >
                <el-input v-model="addForm.goods_price"></el-input>
              </el-form-item>
              <el-form-item
                label="商品重量"
                prop="goods_weight"
                type="number"
              >
                <el-input v-model="addForm.goods_weight"></el-input>
              </el-form-item>
              <el-form-item
                label="商品数量"
                prop="goods_number"
                type="number"
              >
                <el-input v-model="addForm.goods_number"></el-input>
              </el-form-item>
              <el-form-item
                label="商品分类"
                prop="goods_cat"
              >
                <el-cascader
                  expand-trigger="hover"
                  :options="catelist"
                  :props="cateProps"
                  v-model="addForm.goods_cat"
                  @change="handleChange"
                >
                </el-cascader>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane
              label="商品参数"
              name="1"
            >
              <!-- 渲染表单的Item项 -->
              <el-form-item
                :label="item.attr_name"
                v-for="item in manyTableData"
                :key="item.attr_id"
              >
                <!-- 复选框组 -->
                <el-checkbox-group v-model="item.attr_vals">
                  <el-checkbox
                    :label="cb"
                    v-for="(cb, i) in item.attr_vals"
                    :key="i"
                    border
                  ></el-checkbox>
                </el-checkbox-group>
              </el-form-item>

            </el-tab-pane>
            <el-tab-pane
              label="商品属性"
              name="2"
            >
              <!-- 商品属性的渲染 -->
              <el-form-item
                :label="item.attr_name"
                v-for="item in onlyTableData"
                :key="item.attr_id"
              >
                <el-input v-model="item.attr_vals"></el-input>
              </el-form-item>
            </el-tab-pane>
            <el-tab-pane
              label="商品图片"
              name="3"
            >
              <!-- 上传商品图片 -->
              <el-upload
                :action="uploadURL"
                :on-remove="handleRemove"
                list-type="picture"
                :headers="headersObj"
                :on-success="handleSuccess"
                :on-preview="handlePreview"
              >
                <el-button
                  size="small"
                  type="primary"
                >点击上传</el-button>
              </el-upload>
            </el-tab-pane>
            <el-tab-pane
              label="商品内容"
              name="4"
            >
              <!-- 富文本编辑器 -->
              <quill-editor v-model="addForm.goods_introduce">
              </quill-editor>
              <!-- 添加商品内容的按钮 -->
              <el-button
                type="primary"
                class="addG"
                @click="add"
              >添加商品</el-button>
            </el-tab-pane>
          </el-tabs>
        </el-form>
      </template>
    </el-card>
    <el-dialog
      title="图片预览"
      :visible.sync="previewVisible"
      width="50%"
    >
      <img :src="previewPath">
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button
          type="primary"
          @click="previewVisible = false"
        >确 定</el-button>
      </span>
    </el-dialog>
  </div>

</template>

<script>
import _ from 'lodash'

export default {
  name: 'Add',
  data() {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_weight: '',
        goods_price: '',
        goods_number: '',
        goods_cat: [],
        pics: [],
        // 商品的详情描述
        goods_introduce: '',
        attrs: []
      },
      addFormRules: {
        goods_name: [
          { required: true, message: '请输入商品名称', trigger: 'blur' }
        ],
        goods_price: [
          { required: true, message: '请输入商品价格', trigger: 'blur' }
        ],
        goods_weight: [
          { required: true, message: '请输入商品重量', trigger: 'blur' }
        ],
        goods_number: [
          { required: true, message: '请输入商品数量', trigger: 'blur' }
        ],
        goods_cat: [
          { required: true, message: '请选择商品分类', trigger: 'blur' }
        ]
      },
      catelist: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      // 动态参数列表数据
      manyTableData: [],
      // 静态属性列表数据
      onlyTableData: [],
      // 上传图片的URL地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      //上传图片的请求头
      headersObj: {
        Authorization:
          window.sessionStorage.getItem('token')
      },
      previewPath: '',
      previewVisible: false

    }
  },
  created() {
    this.getCateList()
  },
  computed: {
    cateId() {
      return this.addForm.goods_cat[2]
    }
  },
  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        this.$message.error('获取商品分类失败')
      }
      console.log(res.data);
      this.catelist = res.data
    },
    handleChange() {
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.addForm.goods_cat.length === 0) {
        this.$message.error('请先选择商品分类！')
        return false
      }
    },
    async tabClicked() {

      // console.log(this.activeIndex)
      // 证明访问的是动态参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'many' }
          }
        )

        if (res.meta.status !== 200) {
          return this.$message.error('获取动态参数列表失败！')
        }

        console.log(res.data)
        res.data.forEach(item => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: { sel: 'only' }
          }
        )

        if (res.meta.status !== 200) {
          return this.$message.error('获取静态属性失败！')
        }

        console.log(res.data)
        this.onlyTableData = res.data
      }
    },
    handlePreview(file) {
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    handleRemove(file) {
      const filePath = file.response.data.tmp_path
      console.log(filePath);
      const i = this.addForm.pics.findIndex(x => {
        x.pic === filePath
      })
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm);
    },
    handleSuccess(res) {
      console.log(res);
      const picInfo = {
        pic: res.data.tmp_path
      }
      this.addForm.pics.push(picInfo)
    },
    add() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项！')
        }
        // 执行添加的业务逻辑
        // lodash   cloneDeep(obj)
        const form = _.cloneDeep(this.addForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.addForm.attrs.push(newInfo)
        })
        form.attrs = this.addForm.attrs
        console.log(form)

        // 发起请求添加商品
        // 商品的名称，必须是唯一的
        const { data: res } = await this.$http.post('goods', form)

        if (res.meta.status !== 201) {
          return this.$message.error('添加商品失败！')
        }

        this.$message.success('添加商品成功！')
        this.$router.push('/goods')
      })
    }
  },

}
</script>

<style scoped>
.el-alert {
  background-color: #e0dede;
  margin-bottom: 20px;
}
.el-steps {
  margin-bottom: 20px;
}
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.el-dialog img {
  width: 100%;
}
.addG {
  margin-top: 20px;
}
</style>
