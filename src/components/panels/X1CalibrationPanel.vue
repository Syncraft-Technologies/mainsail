<template>
  <v-row no-gutters>
    <v-col cols="12" sm="12" md="10" lg="8" xl="7" class="mx-auto">
      <v-card elevation="2">
        <v-card-title class="text-h6">Compensação — X1 (Extrusor Único)</v-card-title>
        <v-card-text>
          <v-row>
            <v-col cols="12" sm="6" class="min-w-0">
              <v-text-field
                v-model.number="localX1.compX"
                type="number" step="0.01"
                label="Backlash X (mm)" suffix="mm" hide-details="auto"
                dense
              />
            </v-col>
            <v-col cols="12" sm="6" class="min-w-0">
              <v-text-field
                v-model.number="localX1.compY"
                type="number" step="0.01"
                label="Backlash Y (mm)" suffix="mm" hide-details="auto"
                dense
              />
            </v-col>
          </v-row>

          <v-switch
            :input-value="persistOnApply"
            @change="$emit('update:persistOnApply', $event)"
            inset label="Persistir no syncraft-machine.json ao aplicar" class="mb-2"
          />

          <div class="btn-wrap">
            <v-btn class="mr-2 mb-2" :disabled="!canApplyX1" @click="$emit('apply-x1')">
              Aplicar compensação X1
            </v-btn>
            <v-btn class="mr-2 mb-2" @click="$emit('calibrate', { model:'x1', axis:'X' })">
              Calibrar peça — Eixo X (X1)
            </v-btn>
            <v-btn class="mb-2" @click="$emit('calibrate', { model:'x1', axis:'Y' })">
              Calibrar peça — Eixo Y (X1)
            </v-btn>
          </div>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
export default {
  name: 'X1CalibrationPanel',
  props: {
    x1: { type: Object, required: true },
    persistOnApply: { type: Boolean, default: true },
    canApplyX1: { type: Boolean, default: false },
  },
  computed: {
    localX1: {
      get() { return { ...(this.x1 || { compX: 0, compY: 0 }) } },
      set(v) { this.$emit('update:x1', { compX: Number(v.compX)||0, compY: Number(v.compY)||0 }) },
    },
  },
}
</script>

<style scoped>
.min-w-0 { min-width: 0; }               /* evita overflow dos text-fields */
.btn-wrap { display:flex; flex-wrap:wrap; }
.btn-wrap .v-btn { margin-right: 8px; margin-bottom: 8px; }
</style>
