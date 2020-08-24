<template>

  <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId" :default-expanded-keys="defaultExpandedKeys">
     <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"

            type="text"
            size="mini"
            @click="() => append(data)">
            添加
          </el-button>
          <el-button
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
  </el-tree>

</template>

<script>
  export default {
    name: 'category',
    data () {
      return {
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
        }).then((data) => {
          console.log(data)
          this.menus = data.data.data
        })
      },
      append (data) {
        console.log('添加节点:', data)
      },
      remove (node, data) {
        console.log('删除节点:', data, node)
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
          }).then(({data}) => {
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
      }

    }
  }
</script>

<style scoped>

</style>
