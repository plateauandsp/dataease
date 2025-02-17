<script setup lang="ts">
import { dvMainStoreWithOut } from '@/store/modules/data-visualization/dvMain'
import { lockStoreWithOut } from '@/store/modules/data-visualization/lock'
import { copyStoreWithOut } from '@/store/modules/data-visualization/copy'
import { snapshotStoreWithOut } from '@/store/modules/data-visualization/snapshot'
import { layerStoreWithOut } from '@/store/modules/data-visualization/layer'
import { composeStoreWithOut } from '@/store/modules/data-visualization/compose'
import { storeToRefs } from 'pinia'
import { computed, toRefs } from 'vue'
import { ElDivider } from 'element-plus-secondary'
import eventBus from '@/utils/eventBus'
const dvMainStore = dvMainStoreWithOut()
const copyStore = copyStoreWithOut()
const lockStore = lockStoreWithOut()
const snapshotStore = snapshotStoreWithOut()
const layerStore = layerStoreWithOut()
const composeStore = composeStoreWithOut()

const { areaData } = storeToRefs(composeStore)
const { curComponent } = storeToRefs(dvMainStore)
const emit = defineEmits(['close', 'rename'])
const props = defineProps({
  activePosition: {
    type: String,
    default: 'canvas'
  }
})

const { activePosition } = toRefs(props)

const lock = () => {
  snapshotStore.recordSnapshotCache()
  lockStore.lock()
  menuOpt('lock')
}

const unlock = () => {
  snapshotStore.recordSnapshotCache()
  lockStore.unlock()
  menuOpt('unlock')
}

// 点击菜单时不取消当前组件的选中状态
const handleMouseUp = () => {
  dvMainStore.setClickComponentStatus(true)
  activePosition.value === 'aside' && emit('close')
}

const menuOpt = optName => {
  const param = { opt: optName }
  activePosition.value === 'aside' && emit('close', param)
}

const cut = () => {
  copyStore.cut()
  menuOpt('cut')
}

const copy = () => {
  copyStore.copy()
  menuOpt('copy')
}

const hide = () => {
  snapshotStore.recordSnapshotCache()
  layerStore.hideComponent()
  menuOpt('hide')
}

const show = () => {
  snapshotStore.recordSnapshotCache()
  layerStore.showComponent()
  menuOpt('show')
}

const rename = () => {
  emit('rename')
  menuOpt('rename')
}
const paste = () => {
  copyStore.paste(true)
  snapshotStore.recordSnapshotCache('renderChart')
  menuOpt('paste')
}

const deleteComponent = () => {
  if (curComponent.value) {
    dvMainStore.deleteComponentById(curComponent.value?.id)
  } else if (areaData.value.components.length) {
    areaData.value.components.forEach(component => {
      dvMainStore.deleteComponentById(component.id)
    })
  }
  eventBus.emit('hideArea-canvas-main')
  snapshotStore.recordSnapshotCache()
  menuOpt('deleteComponent')
}

const upComponent = () => {
  layerStore.upComponent()
  snapshotStore.recordSnapshotCache('upComponent')
  menuOpt('upComponent')
}

const downComponent = () => {
  layerStore.downComponent()
  snapshotStore.recordSnapshotCache('downComponent')
  menuOpt('downComponent')
}

const topComponent = () => {
  layerStore.topComponent()
  snapshotStore.recordSnapshotCache('topComponent')
  menuOpt('topComponent')
}

const bottomComponent = () => {
  layerStore.bottomComponent()
  snapshotStore.recordSnapshotCache('bottomComponent')
  menuOpt('bottomComponent')
}

const componentCompose = () => {
  composeStore.compose()
  snapshotStore.recordSnapshotCache('componentCompose')
  menuOpt('componentCompose')
}

const decompose = () => {
  composeStore.decompose()
  snapshotStore.recordSnapshotCache('decompose')
  menuOpt('decompose')
}

// 阻止事件向父级组件传播调用父级的handleMouseDown 导致areaData 被隐藏
const handleComposeMouseDown = e => {
  e.preventDefault()
  e.stopPropagation()
}

const composeDivider = computed(() => {
  return !(!curComponent || curComponent['isLock'] || curComponent['component'] != 'Group')
})
</script>

<template>
  <div class="context-menu-details" @mousedown="handleComposeMouseDown">
    <ul @mouseup="handleMouseUp">
      <template v-if="areaData.components.length">
        <li @mousedown="handleComposeMouseDown" @click="componentCompose">组合</li>
        <el-divider class="custom-divider" />
        <li @click="copy">复制</li>
        <li @click="paste">粘贴</li>
        <li @click="cut">剪切</li>
        <el-divider class="custom-divider" />
        <li @click="deleteComponent">删除</li>
      </template>
      <li
        v-show="!(!curComponent || curComponent['isLock'] || curComponent['component'] != 'Group')"
        @click="decompose()"
      >
        取消组合
      </li>
      <el-divider class="custom-divider" v-show="composeDivider" />
      <template v-if="curComponent">
        <template v-if="!curComponent['isLock']">
          <li @click="upComponent">上移一层</li>
          <li @click="downComponent">下移一层</li>
          <li @click="topComponent">置于顶层</li>
          <li @click="bottomComponent">置于底层</li>
          <el-divider class="custom-divider" />
          <li @click="hide" v-show="curComponent['isShow']">隐藏</li>
          <li @click="show" v-show="!curComponent['isShow']">取消隐藏</li>
          <li @click="lock">锁定</li>
          <el-divider class="custom-divider" />
          <li v-if="activePosition === 'aside'" @click="rename">重命名</li>
          <li @click="copy">复制</li>
          <li @click="paste">粘贴</li>
          <li @click="cut">剪切</li>
          <el-divider class="custom-divider" />
          <li @click="deleteComponent">删除</li>
        </template>
        <li v-else @click="unlock">解锁</li>
      </template>
      <li v-else-if="!curComponent && !areaData.components.length" @click="paste">粘贴</li>
    </ul>
  </div>
</template>

<style lang="less" scoped>
.context-menu-details {
  z-index: 1000;
  border: #434343 1px solid;
  width: 220px;
  ul {
    padding: 4px 0;
    background-color: #292929;
    box-shadow: 0 2px 12px 0 rgba(0, 0, 0, 0.1);
    box-sizing: border-box;
    :deep(.ed-divider) {
      margin: 8px 0;
    }

    li {
      font-size: 14px;
      padding: 0 12px;
      position: relative;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      color: #ebebeb;
      height: 34px;
      line-height: 34px;
      box-sizing: border-box;
      cursor: pointer;

      i {
        position: absolute;
        left: 30px;
        top: 50%;
        transform: translate(-50%, -50%);
      }

      &:hover {
        background-color: #333;
      }
    }
  }
}

.custom-divider {
  border-color: #434343 !important;
}
</style>
