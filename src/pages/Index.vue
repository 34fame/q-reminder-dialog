<template>
   <q-page class="flex flex-center">

      <div class="column justify-center items-center q-gutter-lg">
      <q-btn color="primary" label="Create Reminder" @click="showScheduler = true" />
      <div class="q-mt-sm q-px-xl"><pre>{{ JSON.stringify(data, null, 3) }}</pre></div>
      </div>

      <scheduler
         :closeScheduler="closeScheduler"
         :showScheduler="showScheduler"
         :handleSubmit="(formData) => onSubmit(formData)"
      />
   </q-page>
</template>

<script>
export default {
   data() {
      return {
         showScheduler: false,
         data: {}
      }
   },

   methods: {
      closeScheduler() {
         this.showScheduler = false
      },

      onSubmit(formData) {
         this.data = formData
      },

   },

   components: {
      Scheduler: require('components/Scheduler.vue').default,
   },
}
</script>
