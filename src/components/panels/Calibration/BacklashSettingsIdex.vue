<template>
    <v-card outlined class="mb-4">
        <v-card-subtitle class="primary--text font-weight-medium">
            <v-icon left color="primary" small>mdi-tune</v-icon>
            Configurações de Backlash - IDEX Dual Extruder
        </v-card-subtitle>
        
        <v-card-text>
            <!-- Extrusor 0 -->
            <v-subheader class="px-0 font-weight-bold">
                <v-icon left color="orange">mdi-printer-3d-nozzle</v-icon>
                Extrusor 0 (Principal)
            </v-subheader>
            
            <v-row>
                <v-col cols="12" sm="6">
                    <v-text-field
                        :value="extruder0.x"
                        @input="updateExtruder0X"
                        label="Backlash E0 - X (mm)"
                        type="number"
                        step="0.01"
                        min="0"
                        max="2"
                        outlined
                        dense
                        suffix="mm"
                        prepend-icon="mdi-axis-x-arrow"
                        hint="Compensação X do extrusor principal"
                        persistent-hint
                    ></v-text-field>
                </v-col>
                
                <v-col cols="12" sm="6">
                    <v-text-field
                        :value="extruder0.y"
                        @input="updateExtruder0Y"
                        label="Backlash E0 - Y (mm)"
                        type="number"
                        step="0.01"
                        min="0"
                        max="2"
                        outlined
                        dense
                        suffix="mm"
                        prepend-icon="mdi-axis-y-arrow"
                        hint="Compensação Y do extrusor principal"
                        persistent-hint
                    ></v-text-field>
                </v-col>
            </v-row>
            
            <v-divider class="my-4"></v-divider>
            
            <!-- Extrusor 1 -->
            <v-subheader class="px-0 font-weight-bold">
                <v-icon left color="blue">mdi-printer-3d-nozzle</v-icon>
                Extrusor 1 (Secundário)
            </v-subheader>
            
            <v-row>
                <v-col cols="12" sm="6">
                    <v-text-field
                        :value="extruder1.x"
                        @input="updateExtruder1X"
                        label="Backlash E1 - X (mm)"
                        type="number"
                        step="0.01"
                        min="0"
                        max="2"
                        outlined
                        dense
                        suffix="mm"
                        prepend-icon="mdi-axis-x-arrow"
                        hint="Compensação X do extrusor secundário"
                        persistent-hint
                    ></v-text-field>
                </v-col>
                
                <v-col cols="12" sm="6">
                    <v-text-field
                        :value="extruder1.y"
                        @input="updateExtruder1Y"
                        label="Backlash E1 - Y (mm)"
                        type="number"
                        step="0.01"
                        min="0"
                        max="2"
                        outlined
                        dense
                        suffix="mm"
                        prepend-icon="mdi-axis-y-arrow"
                        hint="Compensação Y do extrusor secundário"
                        persistent-hint
                    ></v-text-field>
                </v-col>
            </v-row>
            
            <v-row>
                <v-col cols="12">
                    <v-alert
                        dense
                        text
                        type="info"
                        icon="mdi-information"
                        class="mt-2"
                    >
                        <strong>Importante:</strong> Em sistemas IDEX, cada extrusor pode ter 
                        diferentes valores de backlash devido às tolerâncias mecânicas. 
                        Calibre cada extrusor separadamente.
                    </v-alert>
                </v-col>
            </v-row>
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    name: 'BacklashSettingsIdex',
    
    props: {
        extruder0: {
            type: Object,
            default: () => ({ x: 0.0, y: 0.0 })
        },
        extruder1: {
            type: Object,
            default: () => ({ x: 0.0, y: 0.0 })
        }
    },
    
    methods: {
        updateExtruder0X(value) {
            const numValue = parseFloat(value) || 0
            this.$emit('update:extruder0', { ...this.extruder0, x: numValue })
            this.$emit('settings-changed')
        },
        
        updateExtruder0Y(value) {
            const numValue = parseFloat(value) || 0
            this.$emit('update:extruder0', { ...this.extruder0, y: numValue })
            this.$emit('settings-changed')
        },
        
        updateExtruder1X(value) {
            const numValue = parseFloat(value) || 0
            this.$emit('update:extruder1', { ...this.extruder1, x: numValue })
            this.$emit('settings-changed')
        },
        
        updateExtruder1Y(value) {
            const numValue = parseFloat(value) || 0
            this.$emit('update:extruder1', { ...this.extruder1, y: numValue })
            this.$emit('settings-changed')
        }
    }
}
</script>

<style scoped>
.v-card {
    transition: all 0.3s ease;
}

.v-card:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.v-divider {
    border-color: rgba(0, 0, 0, 0.1);
}
</style>