<template>
  <div class="update-page">
    <UpdatePanel />
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import UpdatePanel from '@/components/panels/Machine/UpdatePanel.vue'

export default defineComponent({
  name: 'UpdatePage',
  components: {
    UpdatePanel
  }
})
</script>

<style scoped>
.update-page {
  padding: 1rem;
}
</style>
