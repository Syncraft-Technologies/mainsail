<template>
    <div class="calibration-page">
        <!-- Apenas o painel de calibração de backlash -->
        <backlash-calibration-panel></backlash-calibration-panel>
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
export default class PageCalibration extends Mixins(DashboardMixin) {
    // Removi todos os getters de layout que podem estar carregando outros painéis
}
</script>

<style scoped>
.calibration-page {
    padding: 16px;
}

/* Esconde qualquer painel de controle de movimento que possa aparecer */
::v-deep .toolhead-control-panel,
::v-deep .move-to-input,
::v-deep .movement-controls,
::v-deep .toolhead-control {
    display: none !important;
}

/* Esconde painéis específicos que podem estar sendo carregados automaticamente */
::v-deep [class*="toolhead"],
::v-deep [class*="movement"],
::v-deep [class*="control-panel"]:not(.backlash-calibration-panel) {
    display: none !important;
}
</style>