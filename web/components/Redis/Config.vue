<template>
  <Conf
    ref="conf"
    :type-flag="'redis'"
    :conf="content"
    :file-ext="'conf'"
    :show-commond="true"
    @on-type-change="onTypeChange"
  >
    <template #common>
      <Common :setting="commonSetting" />
    </template>
  </Conf>
</template>

<script lang="ts" setup>
  import { ref, watch, Ref } from 'vue'
  import Conf from '@web/components/Conf/index.vue'
  import Common from '@web/components/Conf/common.vue'
  import type { CommonSetItem } from '@web/components/Conf/setup'
  import { I18nT } from '@shared/lang'
  import { debounce } from 'lodash'

  const conf = ref()
  const commonSetting: Ref<CommonSetItem[]> = ref([])
  const content = ref('')

  import('@web/config/redis.conf.txt?raw').then((res) => {
    content.value = res.default
  })

  const names: CommonSetItem[] = [
    {
      name: 'port',
      value: '6379',
      enable: true,
      tips() {
        return I18nT('host.port')
      }
    },
    {
      name: 'timeout',
      value: '0',
      enable: true,
      tips() {
        return I18nT('redis.timeout')
      }
    },
    {
      name: 'maxclients',
      value: '10000',
      enable: true,
      tips() {
        return I18nT('mysql.max_connections')
      }
    },
    {
      name: 'databases',
      value: '16',
      enable: true,
      tips() {
        return I18nT('redis.databases')
      }
    },
    {
      name: 'requirepass',
      value: '',
      enable: true,
      tips() {
        return I18nT('redis.requirepass')
      }
    },
    {
      name: 'maxmemory',
      value: '0',
      enable: true,
      tips() {
        return I18nT('redis.maxmemory')
      }
    }
  ]
  let editConfig = ''
  let watcher: any

  const onSettingUpdate = () => {
    let config = editConfig
    const list = ['#PhpWebStudy-Conf-Common-Begin#']
    commonSetting.value.forEach((item) => {
      const regex = new RegExp(`([\\s\\n#]?[^\\n]*)${item.name}(.*?)([^\\n])(\\n|$)`, 'g')
      config = config.replace(regex, `\n\n`)
      if (item.enable) {
        list.push(`${item.name} ${item.value}`)
      }
    })
    list.push('#PhpWebStudy-Conf-Common-END#')
    config = config
      .replace(
        /([\s\n]?[^\n]*)#PhpWebStudy-Conf-Common-Begin#([\s\S]*?)#PhpWebStudy-Conf-Common-END#/g,
        ''
      )
      .replace(/\n+/g, '\n\n')
      .trim()
    config = `${list.join('\n')}\n` + config
    conf.value.setEditValue(config)
  }

  const getCommonSetting = () => {
    if (watcher) {
      watcher()
    }
    const arr = names
      .map((item) => {
        const regex = new RegExp(`([\\s\\n#]?[^\\n]*)${item.name}(.*?)([^\\n])(\\n|$)`, 'g')
        const matchs =
          editConfig.match(regex)?.map((s) => {
            const sarr = s
              .trim()
              .split(' ')
              .filter((s) => !!s.trim())
              .map((s) => s.trim())
            const k = sarr.shift()
            const v = sarr.join(' ').replace(';', '').replace('=', '').trim()
            return {
              k,
              v
            }
          }) ?? []
        console.log('getCommonSetting: ', matchs, item.name)
        const find = matchs?.find((m) => m.k === item.name)
        if (!find) {
          item.enable = false
          return item
        }
        item.value = find?.v ?? item.value
        return item
      })
      .filter((item) => item.show !== false)
    commonSetting.value = arr as any
    watcher = watch(commonSetting, debounce(onSettingUpdate, 500), {
      deep: true
    })
  }

  const onTypeChange = (type: 'default' | 'common', config: string) => {
    console.log('onTypeChange: ', type, config)
    if (editConfig !== config) {
      editConfig = config
      getCommonSetting()
    }
  }
</script>
