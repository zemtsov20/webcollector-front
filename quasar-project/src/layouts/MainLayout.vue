<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-avatar icon="bi-bag-check" size="lg"/>
        <q-toolbar-title class="text-center">
          WBC
        </q-toolbar-title>

        <div>v0.0.1</div>
      </q-toolbar>
    </q-header>
    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>

export default {
  name: 'MainLayout',
  components: {

  },
  data () {
    return {

    }
  }
}
</script>
