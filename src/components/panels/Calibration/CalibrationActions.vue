<template>
    <v-card outlined>
        <v-card-subtitle class="success--text font-weight-medium">
            <v-icon left color="success" small>mdi-printer</v-icon>
            Peças de Calibração
        </v-card-subtitle>
        
        <v-card-text>
            <v-row>
                <v-col cols="12" sm="6">
                    <v-btn
                        color="primary"
                        large
                        block
                        elevation="2"
                        :disabled="!canPrint"
                        @click="printX"
                        class="print-button"
                    >
                        <v-icon left size="24">mdi-axis-x-arrow</v-icon>
                        <div>
                            <div class="font-weight-bold">Imprimir Alinhamento X</div>
                            <div class="caption">Teste de precisão horizontal</div>
                        </div>
                    </v-btn>
                </v-col>
                
                <v-col cols="12" sm="6">
                    <v-btn
                        color="primary"
                        large
                        block
                        elevation="2"
                        :disabled="!canPrint"
                        @click="printY"
                        class="print-button"
                    >
                        <v-icon left size="24">mdi-axis-y-arrow</v-icon>
                        <div>
                            <div class="font-weight-bold">Imprimir Alinhamento Y</div>
                            <div class="caption">Teste de precisão vertical</div>
                        </div>
                    </v-btn>
                </v-col>
            </v-row>
            
            <v-row class="mt-4">
                <v-col cols="12">
                    <v-alert
                        dense
                        text
                        type="warning"
                        icon="mdi-alert"
                        v-if="printerType === 'IDEX'"
                    >
                        <strong>IDEX:</strong> Imprima as peças separadamente para cada extrusor 
                        para verificar o alinhamento individual.
                    </v-alert>
                    
                    <v-expansion-panels flat>
                        <v-expansion-panel>
                            <v-expansion-panel-header class="pa-0">
                                <template v-slot:default="{ open }">
                                    <v-row no-gutters>
                                        <v-col cols="4" class="d-flex justify-start">
                                            <v-icon color="info">mdi-help-circle</v-icon>
                                        </v-col>
                                        <v-col cols="8" class="text--secondary">
                                            <v-fade-transition leave-absolute>
                                                <span v-if="!open">
                                                    Como usar as peças de calibração
                                                </span>
                                            </v-fade-transition>
                                        </v-col>
                                    </v-row>
                                </template>
                            </v-expansion-panel-header>
                            
                            <v-expansion-panel-content>
                                <v-card flat color="grey lighten-5">
                                    <v-card-text>
                                        <ol class="body-2">
                                            <li class="mb-2">
                                                <strong>Imprima as peças:</strong> Comece com os valores padrão e imprima ambas as peças de teste.
                                            </li>
                                            <li class="mb-2">
                                                <strong>Analise os resultados:</strong> Verifique se há desalinhamentos ou inconsistências nas camadas.
                                            </li>
                                            <li class="mb-2">
                                                <strong>Ajuste os valores:</strong> Aumente o backlash se houver folga, diminua se houver tensão excessiva.
                                            </li>
                                            <li class="mb-2">
                                                <strong>Repita o processo:</strong> Continue ajustando até obter resultados satisfatórios.
                                            </li>
                                        </ol>
                                    </v-card-text>
                                </v-card>
                            </v-expansion-panel-content>
                        </v-expansion-panel>
                    </v-expansion-panels>
                </v-col>
            </v-row>
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    name: 'CalibrationActions',
    
    props: {
        printerType: {
            type: String,
            default: ''
        },
        canPrint: {
            type: Boolean,
            default: false
        }
    },
    
    methods: {
        printX() {
            this.$emit('print-x')
        },
        
        printY() {
            this.$emit('print-y')
        }
    }
}
</script>

<style scoped>
.print-button {
    height: 80px !important;
    text-transform: none !important;
}

.print-button .v-btn__content {
    flex-direction: column;
    align-items: flex-start;
    text-align: left;
}

.print-button .v-icon {
    margin-bottom: 4px !important;
    margin-left: 0 !important;
    margin-right: 8px !important;
}

.v-expansion-panel-content {
    margin-top: 8px;
}