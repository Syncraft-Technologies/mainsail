<template>
    <v-dialog
        :value="value"
        @input="$emit('input', $event)"
        max-width="450"
        persistent
    >
        <v-card>
            <v-card-title class="headline warning--text">
                <v-icon left color="warning" size="28">mdi-alert-circle</v-icon>
                {{ title }}
            </v-card-title>
            
            <v-card-text class="pb-0">
                <p class="body-1 mb-4">{{ message }}</p>
                
                <v-alert
                    dense
                    text
                    type="warning"
                    icon="mdi-alert"
                    class="mb-0"
                >
                    Esta ação não pode ser desfeita. Certifique-se antes de continuar.
                </v-alert>
            </v-card-text>
            
            <v-card-actions class="px-6 pb-6">
                <v-spacer></v-spacer>
                
                <v-btn
                    text
                    large
                    @click="cancel"
                    class="mr-2"
                >
                    <v-icon left>mdi-close</v-icon>
                    Cancelar
                </v-btn>
                
                <v-btn
                    color="warning"
                    large
                    elevation="2"
                    @click="confirm"
                >
                    <v-icon left>mdi-check</v-icon>
                    Confirmar
                </v-btn>
            </v-card-actions>
        </v-card>
    </v-dialog>
</template>

<script>
export default {
    name: 'ConfirmationDialog',
    
    props: {
        value: {
            type: Boolean,
            default: false
        },
        title: {
            type: String,
            default: 'Confirmação'
        },
        message: {
            type: String,
            default: 'Tem certeza que deseja continuar?'
        }
    },
    
    methods: {
        confirm() {
            this.$emit('confirm')
            this.cancel()
        },
        
        cancel() {
            this.$emit('input', false)
        }
    }
}
</script>

<style scoped>
.v-card {
    border-radius: 12px;
}

.headline {
    border-bottom: 1px solid rgba(0, 0, 0, 0.12);
    padding-bottom: 16px !important;
    margin-bottom: 0 !important;
}
</style>