<template>
    <v-card outlined class="mb-4">
        <v-card-subtitle class="primary--text font-weight-medium">
            <v-icon left color="primary" small>mdi-tune</v-icon>
            Configurações de Backlash - X1 Standard
        </v-card-subtitle>
        
        <v-card-text>
            <v-row>
                <v-col cols="12" sm="6">
                    <v-text-field
                        :value="backlashX"
                        @input="updateBacklashX"
                        label="Backlash Eixo X (mm)"
                        type="number"
                        step="0.01"
                        min="0"
                        max="2"
                        outlined
                        dense
                        suffix="mm"
                        prepend-icon="mdi-axis-x-arrow"
                        hint="Compensação de folga no eixo X"
                        persistent-hint
                    ></v-text-field>
                </v-col>
                
                <v-col cols="12" sm="6">
                    <v-text-field
                        :value="backlashY"
                        @input="updateBacklashY"
                        label="Backlash Eixo Y (mm)"
                        type="number"
                        step="0.01"
                        min="0"
                        max="2"
                        outlined
                        dense
                        suffix="mm"
                        prepend-icon="mdi-axis-y-arrow"
                        hint="Compensação de folga no eixo Y"
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
                        <strong>Dica:</strong> Valores típicos ficam entre 0.05mm e 0.20mm. 
                        Ajuste conforme os resultados das peças de calibração.
                    </v-alert>
                </v-col>
            </v-row>
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    name: 'BacklashSettingsX1',
    
    props: {
        backlashX: {
            type: Number,
            default: 0.0
        },
        backlashY: {
            type: Number,
            default: 0.0
        }
    },
    
    methods: {
        updateBacklashX(value) {
            const numValue = parseFloat(value) || 0
            this.$emit('update:backlash-x', numValue)
            this.$emit('settings-changed')
        },
        
        updateBacklashY(value) {
            const numValue = parseFloat(value) || 0
            this.$emit('update:backlash-y', numValue)
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
</style>