<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>

        <q-toolbar-title class="text-center">
          Ауыл шаруашылығы деректерін талдаудағы сызықтық регрессия алгоритмі
        </q-toolbar-title>

      </q-toolbar>
    </q-header>

    <q-footer reveal elevated>
      <q-toolbar>
        <q-toolbar-title class="flex flex-center">
          © Зеин және К
        </q-toolbar-title>
      </q-toolbar>
    </q-footer>


    <q-page-container>
      <router-view />
    </q-page-container>
  </q-layout>
</template>

<script>
import { defineComponent} from 'vue'


export default defineComponent({
  name: 'MainLayout',


  setup () {
    return {

    }
  }
})
</script>
