<template>
<div class="main-content">
  <!-- check-strictly：是否遵循父子不互相关联的做法，默认为false -->
  <MultipleLevelTable 
    v-model="selectId"
    :table-data="tableDataValue"
    :check-strictly="true"
    @select="handleSelect"
    @select-all="handleSelectAll"
    @selection-change="handleSelectionChange"
  />
</div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import MultipleLevelTable from './components/MultipleLevelTable.vue'
const selectId = ref([])

const tableDataValue = ref<Array<any>>([])
// 获取列表数据
const getTableList = () => {
  //  模拟接口数据
  const mockResponse = {
    error: 0,
    msg: 'ok',
    data: {
      results: new Array(10).fill('').map((item, index) => {
        return {
          id: index + 1,
          componentName: '总公司',
          curTemplate: '模板',
          relatedPartyRelationship: '一级',
          withinScope: '是',
          projectName: '描述',
          principal: '张三',
          projectTeamMembers: '张敏、李四、王五',
          isGetData: '通过',
          dataUpate: Math.random() <= 0.5 ? 1 : 0, // 1: 更新，0：暂无更新
          children: [
            {
              id: 100 * (index + 1) + 1,
              componentName: '总公司-子公司-子公司-子公司-子公司',
              curTemplate: '模板',
              relatedPartyRelationship: '二级',
              withinScope: '是',
              projectName: '描述',
              principal: '张三',
              projectTeamMembers: '张敏、李四、王五',
              isGetData: '通过',
              dataUpate: 1, // 1: 更新，0：暂无更新
              children: [
                {
                  id: 100000 * (index + 1) + 1,
                  componentName: '分公司-分公司-分公司分公司分公司',
                  curTemplate: '模板',
                  relatedPartyRelationship: '三级',
                  withinScope: '是',
                  projectName: '描述',
                  principal: '张三',
                  projectTeamMembers: '张敏、李四、王五',
                  isGetData: '通过',
                  dataUpate: 0 // 1: 更新，0：暂无更新
                }
              ]
            }
          ]
        }
      })
    }
  }

  const { data, error } = mockResponse

  if (error) return

  tableDataValue.value = data.results
}
getTableList()

// 手动勾选事件
const handleSelect = (row: any, isSelected: any) => {
  console.log('手动勾选事件', row, isSelected)
}

// 全选事件事件
const handleSelectAll = (val: any) => {
  console.log('全选事件事件', val)
}

// 当选择项发生变化时会触发该事件
const handleSelectionChange = (val: any) => {
  console.log('当选择项发生变化时会触发该事件', val)
}

</script>


<style scoped lang="scss">
.main-content {
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  overflow: hidden;
}
</style>
