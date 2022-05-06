<template>
  <q-layout
    view="hHh Lpr lFf"
  >
    <q-header
      elevated
    >
      <q-toolbar>
        <q-btn
          flat
          @click="drawer = !drawer"
          round
          dense
          icon="menu"
        />
        <!-- <q-avatar icon="bi-bag-check" size="lg"/>-->
        <q-toolbar-title
          class="text-center text-bold"
        >
          WBC
        </q-toolbar-title>

        <div>v0.0.1</div>
      </q-toolbar>
    </q-header>
    <q-drawer
      v-model="drawer"
      :width="350"
      :breakpoint="500"
      bordered
      content-class="bg-grey-3 q-pt-xs"
    >
      <q-scroll-area
        class="fit"
      >
        <!-- <q-list>
          <template
            v-for="(menuItem, index) in menuList"
          >
            <q-item
              :key="index"
              clickable
              v-ripple
              @click="onclick(index)"
            >
              <q-item-section
                avatar
              >
                <q-icon
                  name="bi-forward-fill"
                />
              </q-item-section>
              <q-item-section>
                {{ menuItem.label }}
              </q-item-section>
            </q-item>
          </template>
        </q-list> -->
        <q-tree
          :nodes="nodes"
          node-key="url"
          no-connectors
          dense
          :duration="0"
        >
          <template
            v-slot:default-header="prop"
          >
            <div
              class="row items-center"
            >
              <!-- <q-icon
                :name="prop.node.icon || 'share'"
                color="orange"
                size="28px"
                class="q-mr-sm"
              /> -->
              <div
                class="text-weight-bold text-primary"
              >
                {{ prop.node.label }}
              </div>
              <q-btn
                round
                flat
                dense
                color="primary"
                icon="bi-forward-fill"
                @click.stop="onClick(prop.node.url)"
              >
                <q-tooltip>
                  Скопировать ссылку в строку поиска
                </q-tooltip>
              </q-btn>
            </div>
          </template>
        </q-tree>
      </q-scroll-area>
    </q-drawer>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { magic } from 'boot/magicfile'
export default {
  name: 'MainLayout',
  components: {

  },
  data () {
    return {
      drawer: false,
      nodes: null
      // menuList: null
    }
  },
  methods: {
    parseNodes (nodeData) {
      let result = {
        label: nodeData.name,
        url: nodeData.pageUrl,
        children: []
      }
      if (nodeData.childNodes.length !== 0) {
        for (let i = 0; i < nodeData.childNodes.length; i++) {
          result.children.push(this.parseNodes(nodeData.childNodes[i]))
        }
        result.children = Object.freeze(result.children)
      }
      result = Object.freeze(result)
      return result
    },
    parseTree (treeData) {
      const result = []
      for (let i = 0; i < treeData.length; i++) {
        result.push(this.parseNodes(treeData[i]))
      }
      return result
    },
    async getBurger () {
      // Ругается на CORS, у Wildberries стоит не настроена политика кросс-доменных запросов
      // Нужно будет получать с твоего сервера
      // const response = await fetch('https://napi.wildberries.ru/api/menu/getburger?includeBrands=False')
      // let responseData = await response.json()
      /* let responseData = JSON.parse(magic)
      console.log(responseData)
      responseData = responseData.data.catalog
      this.menuList = []
      for (let i = 0; i < responseData.length; i++) {
        this.menuList.push({
          label: responseData[i].name,
          link: responseData[i].pageUrl
        })
      } */
      let responseData = JSON.parse(magic)
      console.log(responseData)
      responseData = responseData.data.catalog
      this.nodes = this.parseTree(responseData)
      this.nodes = Object.freeze(this.nodes)
      console.log(this.nodes)
    },
    onClick (link) {
      this.$store.commit('mySiteStore/updateCurrentLink', link)
    }
  },
  created () {
    this.getBurger()
  }
}
</script>
