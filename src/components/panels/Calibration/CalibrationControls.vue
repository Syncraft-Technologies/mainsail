<template>
    <v-card outlined>
        <v-card-subtitle class="error--text font-weight-medium">
            <v-icon left color="error" small>mdi-cog-outline</v-icon>
            Controles do Sistema
        </v-card-subtitle>
        
        <v-card-text>
            <v-btn
                color="warning"
                large
                block
                elevation="2"
                class="mb-3 control-button"
                @click="resetParams"
            >
                <v-icon left size="20">mdi-backup-restore</v-icon>
                <div>
                    <div class="font-weight-bold">Resetar Parâmetros</div>
                    <div class="caption">Voltar aos valores padrão</div>
                </div>
            </v-btn>
            
            <v-btn
                color="error"
                large
                block
                elevation="2"
                class="control-button"
                @click="restartService"
            >
                <v-icon left size="20">mdi-restart</v-icon>
                <div>
                    <div class="font-weight-bold">Reiniciar Serviço</div>
                    <div class="caption">Aplicar mudanças críticas</div>
                </div>
            </v-btn>
            
            <v-alert
                dense
                text
                type="info"
                icon="mdi-information"
                class="mt-4 mb-0"
            >
                <div class="caption">
                    <strong>Reset:</strong> Remove todas as configurações personalizadas.<br>
                    <strong>Reiniciar:</strong> Necessário após mudanças importantes.
                </div>
            </v-alert>
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    name: 'CalibrationControls',
    
    methods: {
        resetParams() {
            this.$emit('reset-params')
        },
        
        restartService() {
            this.$emit('restart-service')
        }
    }
}
</script>

<style scoped>
.control-button {
    height: 64px !important;
    text-transform: none !important;
}

.control-button .v-btn__content {
    flex-direction: column;
    align-items: flex-start;
    text-align: left;
}

.control-button .v-icon {
    margin-bottom: 4px !important;
    margin-left: 0 !important;
    margin-right: 8px !important;
}
</style>