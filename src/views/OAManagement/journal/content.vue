<template>
  <div class="content">
    <div class="select-box">
      <div class="select-group"
           v-if="selectAuthority">
        <label>发起人</label>
        <el-select v-model="fromData.createUserId"
                   size="small"
                   @change="selectChange"
                   placeholder="请选择">
          <el-option v-for="item in nameOptions"
                     :key="item.userId"
                     :label="item.realname"
                     :value="item.userId">
          </el-option>
        </el-select>
      </div>
      <div class="select-group">
        <label>提交时间</label>
        <el-date-picker @change="selectChange"
                        v-model="fromData.createTime"
                        type="date"
                        value-format="yyyy-MM-dd"
                        placeholder="选择日期">
        </el-date-picker>
      </div>
      <div class="select-group">
        <label>类型</label>
        <el-select v-model="fromData.categoryId"
                   @change="selectChange"
                   size="small"
                   placeholder="请选择">
          <el-option v-for="item in modelOptions"
                     :key="item.key"
                     :label="item.label"
                     :value="item.key">
          </el-option>
        </el-select>
      </div>
    </div>
    <div class="list-box"
         :id="'list-box' + activeName"
         v-loading="journalLoading">
      <journal-cell v-for="(item, index) in journalData"
                    :key="index"
                    :logIndex="index"
                    :data="item"
                    class="list-cell"
                    @on-handle="jourecallCellHandle"></journal-cell>
      <slot name="load"></slot>
      <div ref="blankClick"></div>
    </div>
    <!-- 相关业务页面 -->
    <c-r-m-all-detail :visible.sync="showRelatedDetail"
                      :crmType="relatedCRMType"
                      :listenerIDs="['workbench-main-container']"
                      :noListenerIDs="['journal-list-box']"
                      :id="relatedID"
                      class="d-view">
    </c-r-m-all-detail>
  </div>
</template>

<script>
// API
import { journalDelete } from '@/api/oamanagement/journal'
import CRMAllDetail from '@/views/customermanagement/components/CRMAllDetail'
import JournalCell from './journalCell'

export default {
  components: {
    CRMAllDetail,
    JournalCell
  },
  data() {
    return {
      // 筛选数据
      fromData: {},
      modelOptions: [
        { label: '全部', key: '0' },
        { label: '日报', key: '1' },
        { label: '周报', key: '2' },
        { label: '月报', key: '3' }
      ],
      // 相关详情的查看
      relatedID: '',
      relatedCRMType: '',
      showRelatedDetail: false
    }
  },
  watch: {
    activeName: function(val) {
      this.fromData = {}
    }
  },
  computed: {
    selectAuthority() {
      return this.activeName != 2 // 我发出的日志 的不展示
    }
  },
  props: {
    // 数据
    journalData: Array,
    depOptions: Array,
    nameOptions: Array,
    journalLoading: Boolean,
    activeName: String
  },
  methods: {
    selectChange(val, key) {
      this.$emit('selectChange', this.fromData)
    },
    jourecallCellHandle(data) {
      // 编辑按钮
      if (data.type == 'edit') {
        this.$emit('editBtn', data.data.item)
        this.$refs.blankClick.click()
        // 删除按钮
      } else if (data.type == 'delete') {
        this.$confirm('确定删除?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        })
          .then(() => {
            journalDelete({ logId: data.data.item.logId }).then(res => {
              this.$message({
                type: 'success',
                message: '删除成功!'
              })
              for (let i in this.journalData) {
                if (this.journalData[i].logId == data.data.item.logId) {
                  this.journalData.splice(i, 1)
                  break
                }
              }
            })
          })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          })
        // 相关详情
      } else if (data.type == 'related-detail') {
        this.relatedID = data.data.item.key
        this.relatedCRMType = data.data.type
        this.showRelatedDetail = true
      }
    }
  }
}
</script>

<style scoped lang="scss">
@import '../styles/content.scss';
.content {
  flex: 1;
  display: flex;
  flex-direction: column;
  min-height: 0;
  .select-box {
    margin: 10px 0 20px;
    max-width: 710px;
    .select-group {
      margin-right: 20px;
      display: inline-block;
      margin-bottom: 10px;
      label {
        @include color9;
        margin-right: 5px;
      }
      .el-select,
      .el-date-editor {
        width: 140px;
        height: 30px;
      }
    }
    .group-btn {
      float: right;
      margin-right: 10px;
    }
  }
  .list-box {
    flex: 1;
    overflow-y: auto;
    padding-right: 30px;
    margin-right: -30px;
    position: relative;
  }
}

.list-cell {
  border: 1px solid #e6e6e6;
  margin-bottom: 20px;
  border-radius: 4px;
}

.d-view {
  position: fixed;
  width: 926px;
  top: 60px;
  bottom: 0px;
  right: 0px;
}
</style>
