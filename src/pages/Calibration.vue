<template>
    <div>
        <v-row v-if="isMobile">
           <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <backlash-calibration-panel></backlash-calibration-panel>
            </v-col>
        </v-row>
        
        <v-row v-else-if="isTablet">
            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <backlash-calibration-panel></backlash-calibration-panel>
            </v-col>
        </v-row>

        <v-row v-else-if="isDesktop">
            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <backlash-calibration-panel></backlash-calibration-panel>
            </v-col>
        </v-row>
        <v-row v-else-if="isWidescreen">
            <v-col class="col-12 col-md-12 pb-0 pb-md-3"> 
                <backlash-calibration-panel></backlash-calibration-panel>
            </v-col>
        </v-row>
    </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Mixins } from 'vue-property-decorator'
import DashboardMixin from '@/components/mixins/dashboard'
import BacklashCalibrationPanel from '@/components/panels/BacklashCalibrationPanel.vue'

@Component({
    components: {
        BacklashCalibrationPanel,
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
