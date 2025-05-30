<template>
  <el-drawer
    ref="host-edit-drawer"
    v-model="show"
    size="500px"
    :close-on-click-modal="false"
    :destroy-on-close="true"
    class="host-edit-drawer"
    :with-header="false"
    @closed="closedFn"
  >
    <div class="host-edit">
      <div class="nav">
        <div class="left" @click="show = false">
          <yb-icon :svg="import('@/svg/delete.svg?raw')" class="top-back-icon" />
          <span class="ml-15">{{ isEdit ? I18nT('base.edit') : I18nT('base.add') }}</span>
        </div>
        <el-button :loading="running" :disabled="running" class="shrink0" @click="doSave">{{
          I18nT('base.save')
        }}</el-button>
      </div>

      <div class="main-wapper">
        <div class="main">
          <input
            v-model.trim="item.name"
            type="text"
            :class="'input' + (errs['name'] ? ' error' : '')"
            :placeholder="I18nT('host.placeholderName')"
          />
          <textarea
            v-model.trim="item.alias"
            type="text"
            class="input-textarea"
            :placeholder="I18nT('host.placeholderAlias')"
          ></textarea>
          <input
            v-model.trim="item.mark"
            style="margin: 15px 0 10px"
            class="input"
            :placeholder="I18nT('host.placeholderComment')"
          />
          <div class="path-choose mt-20 mb-20">
            <input
              v-model.trim="item.root"
              type="text"
              :class="'input' + (errs['root'] ? ' error' : '')"
              :placeholder="I18nT('host.placeholderRootPath')"
            />
            <div class="icon-block" @click="chooseRoot('root')">
              <yb-icon
                :svg="import('@/svg/folder.svg?raw')"
                class="choose"
                width="18"
                height="18"
              />
            </div>
          </div>
          <div class="park">
            <div class="title">
              <span>{{ I18nT('base.parkTitle') }}</span>
              <el-popover placement="top" trigger="hover" width="auto">
                <template #reference>
                  <yb-icon
                    :svg="import('@/svg/question.svg?raw')"
                    width="12"
                    height="12"
                    style="margin-left: 5px"
                  ></yb-icon>
                </template>
                <template #default>
                  <p>
                    {{ I18nT('base.parkTips') }}
                  </p>
                </template>
              </el-popover>
            </div>
            <el-switch v-model="park" :before-change="onParkChange"></el-switch>
          </div>
        </div>

        <div class="plant-title">{{ I18nT('base.phpVersion') }}</div>
        <div class="main">
          <div class="port-set">
            <el-select
              v-model="item.phpVersion"
              class="w-p100"
              :placeholder="I18nT('base.selectPhpVersion')"
            >
              <el-option :value="undefined" :label="I18nT('host.staticSite')"></el-option>
              <template v-for="(v, i) in phpVersions" :key="i">
                <el-option :value="v.num" :label="v.num"></el-option>
              </template>
            </el-select>
          </div>
        </div>

        <div class="plant-title">{{ I18nT('host.port') }}</div>
        <div class="main">
          <div class="port-set mb-20">
            <div class="port-type"> Nginx </div>
            <input
              v-model.number="item.port.nginx"
              type="number"
              :class="'input' + (errs['port_nginx'] ? ' error' : '')"
              placeholder="Default: 80"
            />
          </div>

          <div class="port-set mb-20">
            <div class="port-type"> Caddy </div>
            <input
              v-model.number="item.port.caddy"
              type="number"
              :class="'input' + (errs['port_caddy'] ? ' error' : '')"
              placeholder="Default: 80"
            />
          </div>

          <div class="port-set mb-20">
            <div class="port-type"> Apache </div>
            <input
              v-model.number="item.port.apache"
              type="number"
              :class="'input' + (errs['port_apache'] ? ' error' : '')"
              placeholder="Default: 80"
            />
          </div>
        </div>
        <div class="plant-title">{{ I18nT('host.hostSSL') }}</div>
        <div class="main">
          <div class="ssl-switch">
            <span>SSL</span>
            <el-switch v-model="item.useSSL"></el-switch>
          </div>

          <div v-if="item.useSSL" class="ssl-switch" style="margin-top: 12px">
            <span>{{ I18nT('host.autoSSL') }}</span>
            <el-switch v-model="item.autoSSL"></el-switch>
          </div>

          <template v-if="item.useSSL && !item.autoSSL">
            <div class="path-choose mt-20">
              <input
                v-model.trim="item.ssl.cert"
                type="text"
                :class="'input' + (errs['cert'] ? ' error' : '')"
                placeholder="cert"
              />
              <div class="icon-block" @click="chooseRoot('cert', true)">
                <yb-icon
                  :svg="import('@/svg/folder.svg?raw')"
                  class="choose"
                  width="18"
                  height="18"
                />
              </div>
            </div>

            <div class="path-choose mt-20 mb-20">
              <input
                v-model.trim="item.ssl.key"
                type="text"
                :class="'input' + (errs['certkey'] ? ' error' : '')"
                placeholder="cert key"
              />
              <div class="icon-block" @click="chooseRoot('certkey', true)">
                <yb-icon
                  :svg="import('@/svg/folder.svg?raw')"
                  class="choose"
                  width="18"
                  height="18"
                />
              </div>
            </div>
          </template>

          <template v-if="item.useSSL">
            <div class="ssl-switch mb-20 mt-20">
              <span>Port</span>
            </div>
            <div class="port-set port-ssl mb-20">
              <div class="port-type"> Nginx </div>
              <input
                v-model.number="item.port.nginx_ssl"
                type="number"
                :class="'input' + (errs['port_nginx_ssl'] ? ' error' : '')"
                placeholder="Default: 443"
              />
            </div>
            <div class="port-set port-ssl mb-20">
              <div class="port-type"> Caddy </div>
              <input
                v-model.number="item.port.caddy_ssl"
                type="number"
                :class="'input' + (errs['port_caddy_ssl'] ? ' error' : '')"
                placeholder="Default: 443"
              />
            </div>
            <div class="port-set port-ssl mb-20">
              <div class="port-type"> Apache </div>
              <input
                v-model.number="item.port.apache_ssl"
                type="number"
                :class="'input' + (errs['port_apache_ssl'] ? ' error' : '')"
                placeholder="Default: 443"
              />
            </div>
          </template>
        </div>

        <div class="plant-title">
          <span>Nginx Url Rewrite</span>
          <el-popover placement="top" :title="I18nT('base.attention')" width="auto" trigger="hover">
            <template #reference>
              <yb-icon
                :svg="import('@/svg/question.svg?raw')"
                width="12"
                height="12"
                style="margin-left: 5px"
              ></yb-icon>
            </template>
            <p>{{ I18nT('base.nginxRewriteTips') }}</p>
          </el-popover>
        </div>
        <div class="main">
          <el-select
            v-model="rewriteKey"
            filterable
            :placeholder="I18nT('base.commonTemplates')"
            class="w-p100"
            @change="rewriteChange"
          >
            <el-option v-for="key in rewrites" :key="key" :label="key" :value="key"> </el-option>
          </el-select>

          <div ref="input" class="input-textarea nginx-rewrite"></div>
        </div>

        <div class="plant-title flex items-center justify-between">
          <span>{{ I18nT('host.reverseProxy') }}</span>
          <el-button link :icon="Plus" @click.stop="addReverseProxy"></el-button>
        </div>
        <div class="main flex flex-col gap-3">
          <template v-if="item.reverseProxy.length === 0">
            <div class="flex justify-center">{{ I18nT('base.none') }}</div>
          </template>
          <template v-else>
            <template v-for="(proxy, index) in item.reverseProxy" :key="index">
              <div class="flex items-center justify-between gap-2">
                <el-button link :icon="Delete" @click.stop="delReverseProxy(index)"></el-button>
                <el-input v-model="proxy.path" class="w-28 ml-2"></el-input>
                <el-input v-model="proxy.url"></el-input>
              </div>
            </template>
          </template>
        </div>
        <div class="py-5"></div>
      </div>
    </div>
  </el-drawer>
</template>

<script lang="ts" setup>
  import { computed, nextTick, ref, watch, Ref, onMounted, onUnmounted, reactive } from 'vue'
  import { AppStore } from '@web/store/app'
  import { BrewStore } from '@web/store/brew'
  import { editor } from 'monaco-editor/esm/vs/editor/editor.api.js'
  import { I18nT } from '@shared/lang'
  import { RewriteAll } from '@web/components/Host/store'
  import { AsyncComponentSetup } from '@web/fn'
  import { merge } from 'lodash'
  import { EditorConfigMake, EditorCreate } from '@web/fn'
  import { ElMessageBox } from 'element-plus'
  import { Plus, Delete } from '@element-plus/icons-vue'

  const { show, onClosed, onSubmit, closedFn } = AsyncComponentSetup()

  const props = defineProps<{
    isEdit: boolean
    edit: any
  }>()
  const input = ref()
  const running = ref(false)
  const park = ref(false)
  const item = ref({
    id: new Date().getTime(),
    type: 'php',
    name: '',
    alias: '',
    useSSL: false,
    autoSSL: false,
    ssl: {
      cert: '',
      key: ''
    },
    port: {
      nginx: 80,
      nginx_ssl: 443,
      apache: 80,
      apache_ssl: 443,
      caddy: 80,
      caddy_ssl: 443,
      tomcat: 80,
      tomcat_ssl: 443
    },
    nginx: {
      rewrite: ''
    },
    url: '',
    root: '',
    mark: '',
    phpVersion: undefined,
    reverseProxy: []
  })
  const errs = ref({
    name: false,
    root: false,
    cert: false,
    certkey: false,
    port_nginx: false,
    port_caddy: false,
    port_apache: false,
    port_nginx_ssl: false,
    port_apache_ssl: false,
    port_caddy_ssl: false,
    port_tomcat: false,
    port_tomcat_ssl: false
  })
  merge(item.value, props.edit)
  const rewrites: Ref<Array<string>> = ref([])
  const rewriteKey = ref('')
  const appStore = AppStore()
  const brewStore = BrewStore()
  const hosts = computed(() => {
    return appStore.hosts
  })
  const php = computed(() => {
    return brewStore.module('php')
  })
  const phpVersions = computed(() => {
    const set: Set<number> = new Set()
    return (
      php?.value?.installed?.filter((p) => {
        if (p.version && p.num) {
          if (!set.has(p.num)) {
            set.add(p.num)
            return true
          }
          return false
        }
        return false
      }) ?? []
    )
  })

  watch(
    phpVersions,
    (v) => {
      if (props?.isEdit) {
        return
      }
      if (v && v[0] && !item.value.phpVersion) {
        item.value.phpVersion = v[0].num as any
      }
    },
    {
      immediate: true
    }
  )

  watch(
    item,
    () => {
      let k: keyof typeof errs.value
      for (k in errs.value) {
        errs.value[k] = false
      }
    },
    {
      immediate: true,
      deep: true
    }
  )

  const itemName = computed(() => {
    return item.value.name
  })

  watch(itemName, (name) => {
    for (let h of hosts.value) {
      if (h.name === name && h.id !== item.value.id) {
        errs.value['name'] = true
        break
      }
    }
  })

  const addReverseProxy = () => {
    const d = reactive({
      path: '/',
      url: 'http://127.0.0.1:3000'
    })
    const arr: any[] = item.value.reverseProxy as any
    arr.unshift(d)
  }

  const delReverseProxy = (index: number) => {
    const arr: any[] = item.value.reverseProxy as any
    arr.splice(index, 1)
  }

  const onParkChange = () => {
    if (!park.value) {
      return ElMessageBox.confirm(I18nT('base.parkConfirm'), undefined, {
        confirmButtonText: I18nT('base.confirm'),
        cancelButtonText: I18nT('base.cancel'),
        closeOnClickModal: false,
        customClass: 'confirm-del',
        type: 'warning'
      })
    }
    return true
  }

  let monacoInstance: editor.IStandaloneCodeEditor | null

  const initEditor = () => {
    if (!monacoInstance) {
      if (!input?.value?.style) {
        return
      }
      const config = EditorConfigMake(item.value.nginx.rewrite, false, 'off')
      Object.assign(config, {
        minimap: {
          enabled: false
        },
        lineNumbers: 'off',
        padding: {
          top: 8,
          bottom: 8
        }
      })
      monacoInstance = EditorCreate(input.value, config)
    } else {
      monacoInstance.setValue(item.value.nginx.rewrite)
    }
  }

  const rewriteChange = (flag: any) => {
    item.value.nginx.rewrite = RewriteAll[flag]
    monacoInstance?.setValue(item.value.nginx.rewrite)
  }

  const loadRewrite = () => {
    rewrites.value.splice(0)
    rewrites.value.push(...Object.keys(RewriteAll))
  }

  const chooseRoot = (flag: string) => {
    switch (flag) {
      case 'root':
        item.value.root = '/Users/XXX/Desktop/WWW/xxxx'
        break
      case 'cert':
        item.value.ssl.cert = '/Users/XXX/Desktop/WWW/xxxx.cert'
        break
      case 'certkey':
        item.value.ssl.key = '/Users/XXX/Desktop/WWW/xxxx.key'
        break
    }
  }

  const checkItem = () => {
    if (!Number.isInteger(item.value.port.nginx)) {
      errs.value['port_nginx'] = true
    }
    if (!Number.isInteger(item.value.port.apache)) {
      errs.value['port_apache'] = true
    }
    if (!Number.isInteger(item.value.port.caddy)) {
      errs.value['port_caddy'] = true
    }
    if (!Number.isInteger(item.value.port.tomcat)) {
      errs.value['port_tomcat'] = true
    }

    if (item.value.useSSL) {
      if (!Number.isInteger(item.value.port.nginx_ssl)) {
        errs.value['port_nginx_ssl'] = true
      }
      if (!Number.isInteger(item.value.port.apache_ssl)) {
        errs.value['port_apache_ssl'] = true
      }
      if (!Number.isInteger(item.value.port.caddy_ssl)) {
        errs.value['port_caddy_ssl'] = true
      }
      if (!Number.isInteger(item.value.port.tomcat_ssl)) {
        errs.value['port_tomcat_ssl'] = true
      }
    }

    errs.value['name'] = item.value.name.length === 0
    errs.value['root'] = item.value.root.length === 0
    if (item.value.useSSL && !item.value.autoSSL) {
      errs.value['cert'] = item.value.ssl.cert.length === 0
      errs.value['certkey'] = item.value.ssl.key.length === 0
    }
    for (let h of hosts.value) {
      if (h.name === item.value.name && h.id !== item.value.id) {
        errs.value['name'] = true
        break
      }
    }
    let k: keyof typeof errs.value
    for (k in errs.value) {
      if (errs.value[k]) {
        return false
      }
    }
    return true
  }

  const doSave = () => {
    if (!checkItem()) {
      return
    }
    running.value = true
    item.value.nginx.rewrite = monacoInstance?.getValue() ?? ''
    if (props.isEdit) {
      const find = appStore.hosts.findIndex((h) => h.id === props.edit.id)
      if (find >= 0) {
        appStore.hosts.splice(find, 1, JSON.parse(JSON.stringify(item.value)))
      }
    } else {
      appStore.hosts.unshift(JSON.parse(JSON.stringify(item.value)))
    }
    running.value = false
    show.value = false
  }

  loadRewrite()
  onMounted(() => {
    nextTick().then(() => {
      initEditor()
    })
  })
  onUnmounted(() => {
    monacoInstance && monacoInstance.dispose()
    monacoInstance = null
  })

  defineExpose({
    show,
    onSubmit,
    onClosed
  })
</script>
