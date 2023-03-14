<template>
  <div class="part-table">
    <div class="title-content">
      <h3>表格多选、多层级：多选时父子级不关联 {{checkStrictly}}</h3>
      <el-button type="primary" @click="handleClick">点击获取选中id</el-button>
    </div>
      <el-table
        ref="multipleTableRef"
        :data="tableData"
        style="width: 100%; height: 100%"
        row-key="id"
        default-expand-all
        @select="handleSelect"
        @select-all="handleSelectAll"
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          type="selection"
          width="30"
        />
        <el-table-column
          label="测试数据1"
          width="260"
          class-name="part-column-class"
          show-overflow-tooltip
        >
          <template #default="scope">{{ scope.row.componentName }}</template>
        </el-table-column>
        <el-table-column
          property="curTemplate"
          label="测试数据2"
          width="120"
        />
        <el-table-column
          property="relatedPartyRelationship"
          label="测试数据3"
        />
        <el-table-column
          property="withinScope"
          label="测试数据4"
        />
        <el-table-column
          property="projectName"
          label="测试数据5"
        />
        <el-table-column
          property="principal"
          label="测试数据6"
        />
        <el-table-column
          property="projectTeamMembers"
          label="测试数据7"
        />
        <el-table-column
          property="isGetData"
          label="测试数据8"
        />
        <el-table-column
          property="dataUpate"
          label="测试数据9"
          fixed="right"
          width="100"
        >
          <template #default="{ row }">
            <div class="no-update">
              <el-tooltip
                content="暂无数据"
                :show-after="200"
              >
                <el-button
                  link
                  type="primary"
                  class="data-update-text"
                  @click="handleCheck(row)"
                >
                  查看
                </el-button>
              </el-tooltip>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>
</template>

<script lang="ts">
import {
 defineComponent,
 ref,
 getCurrentInstance
} from 'vue'

export default defineComponent({
 name: 'MultipleLevelTable'
})
</script>

<script lang="ts" setup>
 import { ElTable } from 'element-plus'

const props = defineProps({
  modelValue: {
    type: Array,
    default: () => {
      return []
    }
  },
  tableData: {
    type: Array,
    default: () => {
      return []
    }
  },
  checkStrictly: {
    type: Boolean,
    default: false
  }
})

const emits = defineEmits(['select', 'selectAll', 'selectionChange'])

const proxy = getCurrentInstance()

const multipleTableRef = ref<InstanceType<typeof ElTable>>()
// 多选id
const selectListId = ref<Array<any>>([])

// 处理表格多选，父级选中不影响子级
const handleSelect = (row: any, isSelected: any) => {
  // console.log('---', tableData.value, row, isSelected)
  proxy?.emit('select', row, isSelected)
  if(!props.checkStrictly) return

  const idIndex = selectListId.value.findIndex((item) => isSelected.id === item)
  // console.log('idIndex', idIndex)
  if (idIndex < 0) {
    selectListId.value.push(isSelected.id)
  } else {
    selectListId.value.splice(idIndex, 1)
  }

  // 遍历row：包括选中父级默认勾选的子级
  // 对比当前手动选中的selectListId，找出row中多余的id，手动去除勾选
  const differentList = row.filter((item: any) => selectListId.value.every((val: any) => val !== item.id))
  
  // traverseData(tableData.value)：拍平的tableData
  // 遍历tableData中的数据，取消differentList中的id选中（取消父级选中时默认选中的子级）
  if (differentList.length) {
    differentList.forEach((item: any) => {
      traverseData(props.tableData).some((val: any) => {
        if (val.id === item.id) {
          multipleTableRef.value!.toggleRowSelection(val, false)
        }
      })
    })
  }

  // 遍历tableData中的数据，添加父级取消时被取消选中的子级
  if (selectListId.value) {
    selectListId.value.forEach((item: any) => {
      traverseData(props.tableData).some((val: any) => {
        if (item === val.id) {
          multipleTableRef.value!.toggleRowSelection(val, true)
        }
      })
    })
  }
}

// 递归遍历
const traverseData = (arr: any) => {
  const res = ref<Array<any>>([])
  const recursive = (arr: any) => {
    arr.forEach((item: any) => {
      res.value.push(item)
      if (item.children) {
        recursive(item.children)
      }
    })
  }
  recursive(arr)
  return res.value
}


// 全选事件
const handleSelectAll = (val: any) => {
  proxy?.emit('selectAll', val)
  if(!props.checkStrictly) return
  traverseData(props.tableData).forEach((item: any) => {
    if (!val.length) {
      multipleTableRef.value!.toggleRowSelection(item, false)
    } else {
      multipleTableRef.value!.toggleRowSelection(item, true)
    }
  })
  selectListId.value = val.map((item: any) => item.id)
}

const handleSelectionChange = (val: any) => {
  proxy?.emit('selectionChange', val)
  if(props.checkStrictly) return
  selectListId.value = val.map((item: any) => item.id)
}


// 点击查看
const handleCheck = (row: any) => {
  console.log('点击查看', row)
}

// 获取选中id
const handleClick = () => {
  proxy?.emit('update:modelValue', selectListId.value)
  console.log('获取选中id', selectListId.value)
}

</script>

<style scoped lang="scss">
.part-table {
    display: flex;
    flex-direction: column;
    height: calc(100% - 32px);
    padding: 16px;
    min-height: 0;
    background-color: #fff;
    .title-content {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }

    .new-tip-img {
      width: 38px;
      height: 16px;
      margin-left: 2px;
    }

    :deep() {
      .el-table__header {
        tr th {
          background-color: #eef0f4;
        }
      }

      .part-column-class > .cell{
        padding-right: 0;
      }
      // 自定义展开、收起图标
      .el-table__expand-icon > .el-icon {
        position: absolute;
        right: 1px;
        &::before {
          content: "";
          background: url('/src/assets/svg/unfold.svg') no-repeat center;
          background-size: 14px 14px;
          font-style: normal;
          width: 14px;
          height: 14px;
        }

        &:hover {

          &::before {
            content: "";
            background: url('/src/assets/svg/unfold-light.svg') no-repeat center;
            background-size: 14px 14px;
          }
        }

        svg {
          display: none!important;
        }
      }

      [class*="el-table__row--level"] .el-table__expand-icon {
        transform: rotate(0);
      }

      .el-table__expand-icon--expanded .el-icon {
        position: absolute;
        right: 1px;

        &::before {
          content: "";
          background: url('/src/assets/svg/fold.svg') no-repeat center;
          background-size: 14px 14px;
          width: 14px;
          height: 14px;
        }

        &:hover {

          &::before {
            content: "";
            background: url('/src/assets/svg/fold-light.svg') no-repeat center;
            background-size: 14px 14px;
          }
        }
      }
    }
  }
</style>
