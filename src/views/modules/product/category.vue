<template>

  <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId">
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
      }
    }
  }
</script>

<style scoped>

</style>
