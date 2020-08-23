<template>
   <q-layout view="lHh Lpr lFf">
      <q-header elevated>
         <q-toolbar class="q-pa-lg">
            <q-icon name="fas fa-calendar-check" size="40px"/>
            <q-toolbar-title>
               Reminder Dialog
            </q-toolbar-title>

            <div>Quasar v{{ $q.version }}</div>
         </q-toolbar>
      </q-header>

      <q-page-container>
         <router-view/>
      </q-page-container>
   </q-layout>
</template>

<script>
export default {}
</script>
