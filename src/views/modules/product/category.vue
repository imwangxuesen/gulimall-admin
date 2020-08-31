<template>
  <el-tree
    :data="menus"
    :props="defaultProps"
    :expand-on-click-node="false"
    show-checkbox
    node-key="catId"
    :default-expanded-keys="defaultExpandedKeys"
    draggable="true"
    :allow-drop="allowDrop">
    <span class="custom-tree-node" slot-scope="{ node, data }">
      <span>{{ node.label }}</span>
      <span>
        <el-button
          v-if="node.level <= 2"
          type="text"
          size="mini"
          @click="() => append(node, data)">
          添加
        </el-button>
        <el-button
          v-if="node.childNodes.length === 0"
          type="text"
          size="mini"
          @click="() => remove(node, data)">
          删除
        </el-button>
        <el-button
          type="text"
          size="mini"
          @click="() => edit(node, data)">
          修改
        </el-button>
        <!-- Form -->
        <el-dialog :title="title" :visible.sync="dialogFormVisible">
          <el-form :model="category">
            <el-form-item label="名称" label-width="130px">
              <el-input v-model="category.name" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="是否展示" label-width="130px">
              <el-radio v-model="category.showStatus" label="1">展示</el-radio>
              <el-radio v-model="category.showStatus" label="0">隐藏</el-radio>
            </el-form-item>
            <el-form-item label="单位" label-width="130px">
              <el-input
                v-model="category.productUnit"
                autocomplete="off"></el-input>
            </el-form-item>
          </el-form>
          <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="commitCategoryInfoAction()">确 定</el-button>
          </div>
        </el-dialog>
      </span>
    </span>
  </el-tree>
</template>

<script>
export default {
  name: 'category',
  data () {
    return {
      maxLevel: 0,
      title: '',
      dialogType: '',
      dialogFormVisible: false,
      category: {
        name: '',
        showStatus: '1',
        productUnit: '',
        catLevel: 0,
        parentCid: 0,
        sort: 0,
        catId: null
      },
      menus: [],
      defaultExpandedKeys: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    }
  },
  mounted () {
    console.log('请求三级菜单')
    this.getMenus()
  },
  methods: {
    getMenus () {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get'
      }).then(data => {
        console.log(data)
        this.menus = data.data.data
      })
    },
    // 提交分类信息
    commitCategoryInfoAction () {
      var url = this.dialogType === 'add' ? '/product/category/save' : '/product/category/update'
      var { catId, name, icon, productUnit } = this.category
      var data = {catId, name, icon, productUnit}
      this.dialogFormVisible = false
      this.$http({
        url: this.$http.adornUrl(url),
        method: 'post',
        data: this.$http.adornData(data, false)
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.$message({
            message: '保存成功',
            type: 'success'
          })
          this.defaultExpandedKeys = [this.category.parentCid]
          this.getMenus()
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    edit (node, data) {
      this.title = '修改分类'
      this.dialogType = 'edit'

      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get'
      }).then(({ data }) => {
        if (data && data.code === 0) {
          this.category = data.data
          this.dialogFormVisible = true
        } else {
          this.$message.error(data.msg)
        }
      })
    },
    append (node, data) {
      this.title = '添加分类'
      this.dialogType = 'add'
      this.dialogFormVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.category.catId = null
      this.category.name = ''
      this.category.icon = ''
      this.category.productUnit = ''
      this.category.sort = 0
      this.category.showStatus = 1
    },
    remove (node, data) {
      var name = data.name
      this.$confirm(`是否确认删除${name}分类`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        var ids = [data.catId]
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({ data }) => {
          if (data && data.code === 0) {
            this.defaultExpandedKeys = [node.parent.data.catId]
            this.$message({
              message: '操作成功',
              type: 'success',
              duration: 1500,
              onClose: () => {
                this.getMenus()
              }
            })
          } else {
            this.$message.error(data.msg)
          }
        })
      })
    },
    // 是否允许拖拽
    allowDrop (draggingNode, dropNode, type) {
      // 被拖动的当前节点，以及所在的父节点总层数不能大于3
      // 1）被拖动的当前节点层数(本节点及以下层级有几层)
      this.currentNodeDeep(draggingNode.data)
      let level = this.maxLevel - draggingNode.data.catLevel + 1

      if (type === 'inner') {
        return (level + dropNode.data.catLevel) <= 3
      } else {
        return (level + dropNode.parent.level) <= 3
      }
    },
    currentNodeDeep (node) {
      if (node.children || node.children.length > 0) {
        for (let i = 0; i < node.children.length; i++) {
          if (node.children[i].catLevel > this.maxLevel) {
            this.maxLevel = node.children[i].catLevel
          }
          this.currentNodeDeep(node.children[i])
        }
      }
    }
  }
}
</script>

<style scoped></style>
