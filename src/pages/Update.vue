<template>
    <v-container fluid py-0 px-0>
        <v-row>
            <v-col class="col-12 col-md-6 pt-0 pt-md-3">
                <update-panel></update-panel>
            </v-col>
        </v-row>
    </v-container>
</template>
<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import BaseMixin from '@/components/mixins/base'
import UpdatePanel from '@/components/panels/Machine/UpdatePanel.vue'

@Component({
    components: {
        UpdatePanel
    },
})
export default class PageUpdate extends Mixins(BaseMixin) {}
</script>
