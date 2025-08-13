<template>
    <v-snackbar
        :value="value"
        @input="$emit('input', $event)"
        :color="color"
        :timeout="timeout"
        bottom
        right
        shaped
        elevation="6"
    >
        <v-icon 
            left 
            :color="iconColor"
            size="20"
        >
            {{ icon }}
        </v-icon>
        
        <span class="font-weight-medium">{{ message }}</span>
        
        <template v-slot:action="{ attrs }">
            <v-btn
                text
                small
                :color="iconColor"
                v-bind="attrs"
                @click="close"
            >
                <v-icon small>mdi-close</v-icon>
            </v-btn>
        </template>
    </v-snackbar>
</template>

<script>
export default {
    name: 'NotificationSnackbar',
    
    props: {
        value: {
            type: Boolean,
            default: false
        },
        message: {
            type: String,
            default: ''
        },
        color: {
            type: String,
            default: 'success'
        },
        timeout: {
            type: Number,
            default: 4000
        }
    },
    
    computed: {
        icon() {
            switch (this.color) {
                case 'success':
                    return 'mdi-check-circle'
                case 'error':
                    return 'mdi-alert-circle'
                case 'warning':
                    return 'mdi-alert'
                case 'info':
                    return 'mdi-information'
                default:
                    return 'mdi-information'
            }
        },
        
        iconColor() {
            switch (this.color) {
                case 'success':
                    return 'white'
                case 'error':
                    return 'white'
                case 'warning':
                    return 'black'
                case 'info':
                    return 'white'
                default:
                    return 'white'
            }
        }
    },
    
    methods: {
        close() {
            this.$emit('input', false)
        }
    }
}
</script>

<style scoped>
::v-deep .v-snack__wrapper {
    border-radius: 12px;
    box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}
</style>