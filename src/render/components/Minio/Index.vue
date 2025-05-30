<template>
  <div class="soft-index-panel main-right-panel">
    <el-radio-group v-model="tab" class="mt-3">
      <template v-for="(item, index) in tabs" :key="index">
        <el-radio-button :label="item" :value="index"></el-radio-button>
      </template>
    </el-radio-group>
    <div class="main-block">
      <Service v-if="tab === 0" type-flag="minio" title="Minio">
        <template #tool-left>
          <template v-if="isRunning">
            <el-button
              style="color: #01cc74"
              class="button"
              link
              @click.stop="shell.openExternal('http://127.0.0.1:9000')"
            >
              <yb-icon
                style="width: 20px; height: 20px; margin-left: 10px"
                :svg="import('@/svg/http.svg?raw')"
              ></yb-icon>
            </el-button>
          </template>

          <div class="flex items-center gap-1 pl-4">
            <span>{{ I18nT('util.mysqlDataDir') }}: </span>
            <span
              class="cursor-pointer hover:text-yellow-500"
              @click.stop="shell.openPath(DATA_DIR)"
              >{{ DATA_DIR }}</span
            >
            <el-button :disabled="!DATA_DIR" link :icon="Edit" @click.stop="chooseDir"></el-button>
          </div>
        </template>
      </Service>
      <Manager
        v-else-if="tab === 1"
        type-flag="minio"
        title="Minio"
        :has-static="true"
        :show-port-lib="false"
        url="https://github.com/minio/minio"
      >
      </Manager>
    </div>
  </div>
</template>

<script lang="ts" setup>
  import { computed } from 'vue'
  import Service from '../ServiceManager/index.vue'
  import Manager from '../VersionManager/index.vue'
  import { AppModuleSetup } from '@/core/Module'
  import { I18nT } from '@lang/index'
  import { BrewStore } from '@/store/brew'
  import { chooseFolder } from '@/util/File'
  import { MinioSetup } from './setup'
  import { Edit } from '@element-plus/icons-vue'

  const { join } = require('path')
  const { shell } = require('@electron/remote')

  const { tab, checkVersion } = AppModuleSetup('minio')
  const tabs = [I18nT('base.service'), I18nT('base.versionManager')]
  checkVersion()

  const brewStore = BrewStore()

  const isRunning = computed(() => {
    return brewStore.module('minio').installed.some((m) => m.run)
  })

  const currentVersion = computed(() => {
    return brewStore.currentVersion('minio')
  })

  const DATA_DIR = computed({
    get() {
      if (currentVersion?.value?.bin) {
        if (MinioSetup.dir[currentVersion.value.bin]) {
          return MinioSetup.dir[currentVersion.value.bin]
        }
        return join(global.Server.BaseDir!, `minio/data`)
      }
      return I18nT('base.needSelectVersion')
    },
    set(v: string) {
      if (!currentVersion?.value?.bin) {
        return
      }
      MinioSetup.dir[currentVersion.value.bin] = v
      MinioSetup.save()
    }
  })

  const chooseDir = () => {
    chooseFolder()
      .then((path: string) => {
        DATA_DIR.value = path
      })
      .catch()
  }
</script>
