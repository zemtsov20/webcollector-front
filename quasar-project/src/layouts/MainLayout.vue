<template>
  <q-layout view="hHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn flat @click="drawer = !drawer" round dense icon="menu" />
<!--        <q-avatar icon="bi-bag-check" size="lg"/>-->
        <q-toolbar-title class="text-center text-bold">
          WBC
        </q-toolbar-title>

        <div>v0.0.1</div>
      </q-toolbar>
    </q-header>
    <q-drawer
      v-model="drawer"
      show-if-above
      :width="200"
      :breakpoint="500"
      bordered
      content-class="bg-grey-3"
    >
      <q-scroll-area class="fit">
        <q-list>
          <template v-for="(menuItem, index) in menuList">
            <q-item :key="index" clickable v-ripple @click="onclick(index)">
              <q-item-section avatar>
                <q-icon :name="'bi-caret-right-fill'" />
              </q-item-section>
              <q-item-section>
                {{ menuItem.label }}
              </q-item-section>
            </q-item>
          </template>
        </q-list>
      </q-scroll-area>
    </q-drawer>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { eventbus } from 'boot/eventbus'

export default {
  name: 'MainLayout',
  components: {

  },
  data () {
    return {
      drawer: false,
      menuList: [
        {
          link: 'etu.ru',
          label: 'Inbox'
        },
        {
          label: 'Outbox'
        },
        {
          label: 'Trash'
        },
        {
          icon: 'error',
          label: 'Spam'
        },
        {
          label: 'Settings'
        },
        {
          icon: 'feedback',
          label: 'Send Feedback'
        },
        {
          icon: 'help',
          iconColor: 'primary',
          label: 'Help'
        }
      ]
    }
  },
  methods: {
    async getBurger () {
      const response = await fetch('https://napi.wildberries.ru/api/menu/getburger?includeBrands=False')
      const responseData = await response.json().data.catalog
      const menuList = []
      for (let i = 0; i < responseData.length; i++) {
        menuList.push(responseData.name)
      }
      return responseData
    },
    onclick (buttonIdx) {
      console.log(buttonIdx, eventbus)
      eventbus.$emit('clicked', this.menuList)
    }
  },
  created () {
    // this.menuList = this.getBurger()
  }
}
</script>
