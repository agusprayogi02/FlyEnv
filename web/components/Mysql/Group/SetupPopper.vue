<template>
  <el-popover
    :show-arrow="false"
    effect="dark"
    popper-class="host-list-poper"
    placement="bottom-end"
    width="auto"
  >
    <ul v-poper-fix class="host-list-menu">
      <li @click.stop="action('edit')">
        <span class="ml-15">{{ $t('base.edit') }}</span>
      </li>
      <li @click.stop="action('socket')">
        <span class="ml-15"> {{ $t('util.mysqlPopperSocket') }} </span>
      </li>
      <li @click.stop="action('cnf')">
        <span class="ml-15">{{ $t('base.configFile') }}</span>
      </li>
      <li @click.stop="action('log')">
        <span class="ml-15">{{ $t('base.log') }}</span>
      </li>
      <li @click.stop="action('log-slow')">
        <span class="ml-15">{{ $t('base.slowLog') }}</span>
      </li>
      <li @click.stop="action('del')">
        <span class="ml-15">{{ $t('base.del') }}</span>
      </li>
    </ul>
    <template #reference>
      <div class="more">
        <yb-icon :svg="import('@/svg/more1.svg?raw')" width="22" height="22" />
      </div>
    </template>
  </el-popover>
</template>

<script lang="ts" setup>
  import { I18nT } from '@shared/lang'
  import { AsyncComponentShow } from '@web/fn'
  import { MessageSuccess } from '@/util/Element'
  import type { MysqlGroupItem } from '@shared/app'
  import { MysqlStore } from '../mysql'
  import { ElMessageBox } from 'element-plus'

  const props = defineProps<{
    item: MysqlGroupItem
  }>()

  const mysqlStore = MysqlStore()

  const copyString = (): void => {
    MessageSuccess(I18nT('base.copySuccess'))
  }

  const doDel = (data: MysqlGroupItem) => {
    ElMessageBox.confirm(I18nT('base.areYouSure'), undefined, {
      confirmButtonText: I18nT('base.confirm'),
      cancelButtonText: I18nT('base.cancel'),
      closeOnClickModal: false,
      customClass: 'confirm-del',
      type: 'warning'
    }).then(() => {
      const index = mysqlStore.all.findIndex((f) => f === data)
      if (index >= 0) {
        mysqlStore.stop(data).then(() => {
          mysqlStore.all.splice(index, 1)
          mysqlStore.save().then()
          MessageSuccess(I18nT('base.success'))
        })
      }
    })
  }

  let EditVM: any
  import('./Add.vue').then((res) => {
    EditVM = res.default
  })

  let ConfigVM: any
  import('./Config.vue').then((res) => {
    ConfigVM = res.default
  })

  let LogVM: any
  import('./Logs.vue').then((res) => {
    LogVM = res.default
  })

  const action = (flag: string) => {
    switch (flag) {
      case 'edit':
        AsyncComponentShow(EditVM, {
          item: props.item
        }).then()
        break
      case 'socket':
        copyString()
        break
      case 'cnf':
        AsyncComponentShow(ConfigVM, {
          item: props.item
        }).then()
        break
      case 'log':
        AsyncComponentShow(LogVM, {
          item: props.item,
          flag: 'log'
        }).then()
        break
      case 'log-slow':
        AsyncComponentShow(LogVM, {
          item: props.item,
          flag: 'slow-log'
        }).then()
        break
      case 'del':
        doDel(props.item)
        break
    }
  }
</script>
