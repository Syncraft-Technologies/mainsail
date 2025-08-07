<template>
    <div>
        <v-row v-if="isMobile">
           <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <webcam-panel></webcam-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3">                
                <status-panel></status-panel>
            </v-col>
           
            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <temperature-panel></temperature-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <toolhead-control-panel></toolhead-control-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <extruder-control-panel></extruder-control-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <miscellaneous-panel></miscellaneous-panel>
            </v-col>
        </v-row>
        
        <v-row v-else-if="isTablet">
            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <webcam-panel></webcam-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3">                
                <status-panel></status-panel>
            </v-col>
           
            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <temperature-panel></temperature-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <toolhead-control-panel></toolhead-control-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <extruder-control-panel></extruder-control-panel>
            </v-col>

            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <miscellaneous-panel></miscellaneous-panel>
            </v-col>
        </v-row>

        <v-row v-else-if="isDesktop">
            <v-col class="col-6 col-md-6 pb-0 pb-md-3"> 
                <webcam-panel></webcam-panel>
            </v-col>

            <v-col class="col-6 col-md-6 pb-0 pb-md-3">                
                <status-panel></status-panel>
            </v-col>
           
            <v-col class="col-6 col-md-6 pb-0 pb-md-3"> 
                <temperature-panel></temperature-panel>
            </v-col>

            <v-col class="col-6 col-md-6 pb-0 pb-md-3"> 
                <toolhead-control-panel></toolhead-control-panel>
            </v-col>

            <v-col class="col-6 col-md-6 pb-0 pb-md-3"> 
                <extruder-control-panel></extruder-control-panel>
            </v-col>

            <v-col class="col-6 col-md-6 pb-0 pb-md-3"> 
                <miscellaneous-panel></miscellaneous-panel>
            </v-col>
        </v-row>
        <v-row v-else-if="isWidescreen">
            
            <v-col class="col-4 col-md-4 pb-0 pb-md-3"> 
                <webcam-panel></webcam-panel>
            </v-col>

            <v-col class="col-4 col-md-4 pb-0 pb-md-3">                
                <status-panel></status-panel>
            </v-col>
           
            <v-col class="col-4 col-md-4 pb-0 pb-md-3"> 
                <temperature-panel></temperature-panel>
            </v-col>

            <v-col class="col-4 col-md-4 pb-0 pb-md-3"> 
                <toolhead-control-panel></toolhead-control-panel>
            </v-col>

            <v-col class="col-4 col-md-4 pb-0 pb-md-3"> 
                <extruder-control-panel></extruder-control-panel>
            </v-col>

            <v-col class="col-4 col-md-4 pb-0 pb-md-3"> 
                <miscellaneous-panel></miscellaneous-panel>
            </v-col>
        </v-row>
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import ExtruderControlPanel from '@/components/panels/ExtruderControlPanel.vue'
import DashboardMixin from '@/components/mixins/dashboard'
import KlippyStatePanel from '@/components/panels/KlippyStatePanel.vue'
import MachineSettingsPanel from '@/components/panels/MachineSettingsPanel.vue'
import MacrogroupPanel from '@/components/panels/MacrogroupPanel.vue'
import MacrosPanel from '@/components/panels/MacrosPanel.vue'
import MiniconsolePanel from '@/components/panels/MiniconsolePanel.vue'
import MinSettingsPanel from '@/components/panels/MinSettingsPanel.vue'
import MiscellaneousPanel from '@/components/panels/MiscellaneousPanel.vue'
import SpoolmanPanel from '@/components/panels/SpoolmanPanel.vue'
import StatusPanel from '@/components/panels/StatusPanel.vue'
import ToolheadControlPanel from '@/components/panels/ToolheadControlPanel.vue'
import TemperaturePanel from '@/components/panels/TemperaturePanel.vue'
import WebcamPanel from '@/components/panels/WebcamPanel.vue'

@Component({
    components: {
        ExtruderControlPanel,
        KlippyStatePanel,
        MachineSettingsPanel,
        MacrogroupPanel,
        MinSettingsPanel,
        MiscellaneousPanel,
        SpoolmanPanel,
        StatusPanel,
        ToolheadControlPanel,
        TemperaturePanel,
        WebcamPanel,
    },
})
export default class PageDashboard extends Mixins(DashboardMixin) {
    get mobileLayout() {
        return this.$store.getters['gui/getPanels']('mobile', 0, true)
    }

    get tabletLayout1() {
        return this.$store.getters['gui/getPanels']('tablet', 1, true)
    }

    get tabletLayout2() {
        return this.$store.getters['gui/getPanels']('tablet', 2, true)
    }

    get desktopLayout1() {
        return this.$store.getters['gui/getPanels']('desktop', 1, true)
    }

    get desktopLayout2() {
        return this.$store.getters['gui/getPanels']('desktop', 2, true)
    }

    get widescreenLayout1() {
        return this.$store.getters['gui/getPanels']('widescreen', 1, true)
    }

    get widescreenLayout2() {
        return this.$store.getters['gui/getPanels']('widescreen', 2, true)
    }

    get widescreenLayout3() {
        return this.$store.getters['gui/getPanels']('widescreen', 3, true)
    }

    extractPanelName(name: string) {
        return name.split('_')[0] + '-panel'
    }

    extractPanelId(name: string) {
        return name.split('_')[1] ?? null
    }
}
</script>
