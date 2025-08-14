<template>
  <v-row no-gutters>
    <v-col cols="12" sm="12" md="11" lg="9" xl="8" class="mx-auto">
      <v-card elevation="2">
        <v-card-title class="text-h6">Compensação — Syncraft IDEX (E0 / E1)</v-card-title>
        <v-card-text>
          <v-row>
            <v-col cols="12" sm="6" class="min-w-0">
              <v-text-field v-model.number="localIdex.e0.compX" type="number" step="0.01"
                            label="E0 — Backlash X (mm)" suffix="mm" hide-details="auto" dense/>
            </v-col>
            <v-col cols="12" sm="6" class="min-w-0">
              <v-text-field v-model.number="localIdex.e0.compY" type="number" step="0.01"
                            label="E0 — Backlash Y (mm)" suffix="mm" hide-details="auto" dense/>
            </v-col>
          </v-row>

          <v-row>
            <v-col cols="12" sm="6" class="min-w-0">
              <v-text-field v-model.number="localIdex.e1.compX" type="number" step="0.01"
                            label="E1 — Backlash X (mm)" suffix="mm" hide-details="auto" dense/>
            </v-col>
            <v-col cols="12" sm="6" class="min-w-0">
              <v-text-field v-model.number="localIdex.e1.compY" type="number" step="0.01"
                            label="E1 — Backlash Y (mm)" suffix="mm" hide-details="auto" dense/>
            </v-col>
          </v-row>

          <v-switch :input-value="persistOnApply"
                    @change="$emit('update:persistOnApply', $event)"
                    inset label="Persistir no syncraft-machine.json ao aplicar" class="mb-2"/>

          <div class="btn-wrap">
            <v-btn class="mr-2 mb-2" :disabled="!canApplyIdex" @click="$emit('apply-idex')">
              Aplicar compensação IDEX
            </v-btn>
            <v-divider class="my-4" inset/>
            <v-btn class="mr-2 mb-2" @click="$emit('calibrate', { model:'idex', axis:'X', tool:'E0' })">Calibrar — X (E0)</v-btn>
            <v-btn class="mr-2 mb-2" @click="$emit('calibrate', { model:'idex', axis:'Y', tool:'E0' })">Calibrar — Y (E0)</v-btn>
            <v-btn class="mr-2 mb-2" @click="$emit('calibrate', { model:'idex', axis:'X', tool:'E1' })">Calibrar — X (E1)</v-btn>
            <v-btn class="mb-2"       @click="$emit('calibrate', { model:'idex', axis:'Y', tool:'E1' })">Calibrar — Y (E1)</v-btn>
          </div>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'IdexCalibrationPanel',
  props: {
    idex: { type: Object, required: true },
    persistOnApply: { type: Boolean, default: true },
    canApplyIdex: { type: Boolean, default: false },
  },
  computed: {
    localIdex: {
      get() {
        const b = this.idex || {}
        return {
          e0: { compX: Number(b?.e0?.compX)||0, compY: Number(b?.e0?.compY)||0 },
          e1: { compX: Number(b?.e1?.compX)||0, compY: Number(b?.e1?.compY)||0 },
        }
      },
      set(v) {
        this.$emit('update:idex', {
          e0: { compX: Number(v.e0.compX)||0, compY: Number(v.e0.compY)||0 },
          e1: { compX: Number(v.e1.compX)||0, compY: Number(v.e1.compY)||0 },
        })
      },
    },
  },
}
</script>

<style scoped>
.min-w-0 { min-width: 0; }
.btn-wrap { display:flex; flex-wrap:wrap; }
.btn-wrap .v-btn { margin-right: 8px; margin-bottom: 8px; }
</style>
