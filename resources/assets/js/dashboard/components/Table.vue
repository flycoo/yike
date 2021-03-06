<template>
  <div :class="tableWrapper">
    <el-row>
      <el-col :span="18" v-if="searchables">
        <el-form :inline="true" :model="query">
          <el-form-item v-for="(placeholder, field) in searchables" :key="field">
            <el-input v-model="query[field]" :placeholder="placeholder"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="onSearch">查询</el-button>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="6">
        <div :class="searchables ? 'float-right' : ''">
          <slot name="right-buttons"></slot>
        </div>
      </el-col>
    </el-row>
    <div class="mb-3">
      <el-tag v-for="(value, key) in query" :key="key" class="mr-1" @close="delete query[key]; fetch()" :closable="true">{{ searchables[key] }}: {{ value }}</el-tag>
      <el-tag v-if="showHideTips" class="mr-1" @close="resetHideQuery" :closable="true">Cancel Hide Query</el-tag>
    </div>
    <el-table :data="items" style="width: 100%">
      <template v-for="column in columns">
        <template v-if="column.name">
          <template v-if="column.name == '__actions'">
            <el-table-column :label="column.label" :width="column.width">
              <template scope="scope">
                <template v-for="action in itemActions">
                  <el-button
                    :size="action.size?action.size:'small'"
                    :type="action.type?action.type:''"
                    @click="callAction(action.name, scope)">{{ action.label }}</el-button>
                </template>
              </template>
            </el-table-column>
          </template>
          <el-table-column v-else :prop="column.prop" :label="column.label" :width="column.width">
            <template scope="scope">
              <slot :name="column.name" :data="scope"></slot>
            </template>
          </el-table-column>
        </template>
        <template v-else>
          <el-table-column :prop="column.prop" :label="column.label" :width="column.width"></el-table-column>
        </template>
      </template>
    </el-table>

    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagination.current_page"
      :page-sizes="[10, 20, 30, 40]"
      :page-size="pagination.per_page"
      layout="total, sizes, prev, pager, next, jumper"
      :total="pagination.total"
      :class="paginationWrapper">
    </el-pagination>
  </div>
</template>

<script>
  export default {
    props: {
      tableWrapper: {
        type: String,
        default: null
      },
      paginationWrapper: {
        type: String,
        default() {
          return 'pagination-wrapper'
        }
      },
      columns: {
        type: Array,
        default() {
          return []
        }
      },
      searchables: {
        type: Object,
        default: undefined
      },
      query: {
        type: Object,
        default() {
          return {}
        }
      },
      hideQuery: {
        type: Object,
        default() {
          return undefined
        }
      },
      api: {
        type: String,
        default: null
      },
      include: {
        type: String,
        default: null
      },
      itemActions: {
        type: Array,
        default() {
          return [
            { name: 'edit-item', label: '编辑' },
            { name: 'delete-item', label: '删除', type: 'danger' }
          ]
        }
      },
    },
    data() {
      return {
        items: [],
        showHideTips: false,
        hideQueryList: this.hideQuery,
        pagination: {
          count: 0,
          current_page: 1,
          per_page: 20,
          total: 0,
          total_pages: 1,
        },
      }
    },
    created() {
      this.pagination.current_page = parseInt(this.$route.query.page) || this.pagination.current_page
      this.pagination.per_page = parseInt(this.$route.query.per_page) || this.pagination.per_page

      this.fetch()
    },
    mounted() {
      this.$parent.$on('reload', () => {
          this.fetch()
      })
    },
    methods: {
      onSearch() {
        this.pagination.current_page = 1
        this.fetch()
      },
      fetch() {
        let that = this
        let query = Object.assign({}, this.query)

        if (this.hideQueryList != undefined) {
          query = Object.assign(query, this.hideQueryList)
          Object.keys(this.hideQueryList).forEach(function(key) {
            if (that.$route.query[key] != undefined) that.showHideTips = true
          });
        }

        query['page'] = this.pagination.current_page
        query['per_page'] = this.pagination.per_page

        this.$router.replace({ name: this.$route.name, query: query })

        query['include'] = this.include

        this.$http.get(this.$endpoints[this.api], { params: query })
            .then(({ data }) => {
              this.items = data.data
              this.pagination = data.meta.pagination
            })
            .catch(response => console.log(response))
      },
      handleSizeChange(val) {
        this.pagination.per_page = val
        this.fetch()
      },
      handleCurrentChange(val) {
        this.pagination.current_page = val
        this.fetch()
      },
      callAction(action, data) {
        this.$emit('table-action', action, data)
      },
      resetHideQuery() {
        this.hideQueryList = undefined
        this.showHideTips = false
        this.fetch()
      },
    }
  }
</script>

<style lang="scss" scoped>
.pagination-wrapper {
  text-align: center;
  margin: 20px 0;
}
.el-form-item {
    margin-bottom: 12px;
}
</style>
