<template>
    <v-container fluid py-0 px-0>
        <v-row>
            <v-col class="col-12 col-md-6 pb-0 pb-md-3">
                <config-files-panel></config-files-panel>
            </v-col>
            <v-col class="col-12 col-md-6 pt-0 pt-md-3">
                <klippy-state-panel></klippy-state-panel>
                <system-panel></system-panel>
                <update-panel></update-panel>
                <v-row>
                    <v-col v-if="klipperState === 'ready'" class="col-12 col-sm-6 pb-0 pb-md-3">
                        <endstop-panel></endstop-panel>
                    </v-col>
                    <v-col :class="klipperState !== 'ready' ? 'col-12' : 'col-12 col-sm-6 pt-0 pt-sm-3'">
                        <logfiles-panel></logfiles-panel>
                    </v-col>
                </v-row>
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
