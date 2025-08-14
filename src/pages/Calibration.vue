<template>
  <div class="pa-4">
    <v-row class="mb-4" align="stretch">
      <v-col cols="12">
        <v-alert dense outlined type="info" class="mb-2">
          <div class="d-flex align-center justify-space-between flex-wrap">
            <div class="mb-2 mb-sm-0">
              <strong>Modelo detectado:</strong>
              <span v-if="printerModel">{{ printerModel }}</span>
              <span v-else>—</span>
            </div>
            <div class="d-flex flex-wrap">
              <v-btn x-small class="mr-2 mb-2" @click="loadMachineJson">Recarregar config</v-btn>
              <v-btn x-small class="mb-2" @click="saveMachineJson" :disabled="!canSaveJson">Salvar config</v-btn>
            </div>
          </div>
        </v-alert>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <div class="panel-container">
          <X1CalibrationPanel
            v-if="modelKind === 'X1'"
            :persist-on-apply.sync="persistOnApply"
            :x1.sync="x1"
            :can-apply-x1="canApplyX1"
            @apply-x1="applyX1"
            @calibrate="calibrate"
          />

          <IdexCalibrationPanel
            v-else-if="modelKind === 'IDEX'"
            :persist-on-apply.sync="persistOnApply"
            :idex.sync="idex"
            :can-apply-idex="canApplyIdex"
            @apply-idex="applyIdex"
            @calibrate="calibrate"
          />

          <template v-else>
            <v-alert type="warning" outlined>
              Modelo de impressora não reconhecido. Valor atual: <strong>{{ printerModel || 'indefinido' }}</strong>.
            </v-alert>
          </template>
        </div>
      </v-col>
    </v-row>

    <v-row>
      <v-col cols="12">
        <v-card elevation="2">
          <v-card-title class="text-h6">Ações</v-card-title>
          <v-card-text>
            <v-alert dense text type="info" class="mb-4">
              Após alterar compensações, é recomendável reiniciar o serviço para garantir que os novos valores sejam aplicados globalmente.
            </v-alert>
            <div class="d-flex flex-wrap">
              <v-btn class="mr-2 mb-2" @click="resetParameters">Zerar parâmetros</v-btn>
              <v-btn color="error" class="mb-2" @click="restartService">Reiniciar serviço</v-btn>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import Component from 'vue-class-component'
import { Vue } from 'vue-property-decorator'
import X1CalibrationPanel from '@/components/panels/X1CalibrationPanel.vue'
import IdexCalibrationPanel from '@/components/panels/IdexCalibrationPanel.vue'

interface AxisPair { compX: number | null; compY: number | null }
interface SyncraftMachine {
  printerModel?: 'X1' | 'IDEX' | 'Syncraft' | string
  bc_x0?: number
  bc_x1?: number
  bc_y0?: number
  bc_y1?: number
  bootVideo?: string
  welcomeScreen?: boolean
}

@Component({
  name: 'BacklashCalibrationPage',
  components: { X1CalibrationPanel, IdexCalibrationPanel },
})
export default class BacklashCalibrationPage extends Vue {
  // UI/estado
  printerModel: string | null = null
  persistOnApply = true

  // Arquivo JSON
  jsonFileName = 'syncraft-machine.json'
  jsonPathRoot: '' | 'config' | 'data' | 'gcodes' | '~' | 'home' = 'config'
  machineJson: SyncraftMachine | null = null

  // X1 (single extrusor)
  x1: AxisPair = { compX: null, compY: null }

  // IDEX (E0/E1)
  idex = {
    e0: { compX: null as number | null, compY: null as number | null },
    e1: { compX: null as number | null, compY: null as number | null },
  }

  // —— Lifecycle ——
  async mounted() {
    await this.loadMachineJson()
  }

  // —— Computed ——
  get canApplyX1() {
    return this.isNumber(this.x1.compX) || this.isNumber(this.x1.compY)
  }
  get canApplyIdex() {
    return (
      this.isNumber(this.idex.e0.compX) ||
      this.isNumber(this.idex.e0.compY) ||
      this.isNumber(this.idex.e1.compX) ||
      this.isNumber(this.idex.e1.compY)
    )
  }
  get canSaveJson() { return !!this.machineJson }

  get modelKind(): 'X1' | 'IDEX' {
    const p = (this.printerModel || '').toUpperCase()
    if (p === 'X1') return 'X1'
    if (p.includes('IDEX') || p.includes('SYNC')) return 'IDEX'
    return 'X1'
  }

  // —— Helpers ——
  private isNumber(v: unknown): v is number { return typeof v === 'number' && !isNaN(v as number) }

  private mapJsonToUI(json: SyncraftMachine) {
    this.printerModel = json.printerModel ?? this.printerModel ?? 'X1'
    // X1 usa E0 como base
    if ((json.printerModel || 'X1').toUpperCase() === 'X1') {
      this.x1.compX = this.isNumber(json.bc_x0) ? (json.bc_x0 as number) : 0
      this.x1.compY = this.isNumber(json.bc_y0) ? (json.bc_y0 as number) : 0
    }
    // Preencher IDEX também
    this.idex.e0.compX = this.isNumber(json.bc_x0) ? (json.bc_x0 as number) : 0
    this.idex.e0.compY = this.isNumber(json.bc_y0) ? (json.bc_y0 as number) : 0
    this.idex.e1.compX = this.isNumber(json.bc_x1) ? (json.bc_x1 as number) : 0
    this.idex.e1.compY = this.isNumber(json.bc_y1) ? (json.bc_y1 as number) : 0
  }

  private mapUIToJson(): SyncraftMachine {
    const base: SyncraftMachine = {
      ...(this.machineJson ?? {}),
      printerModel: this.printerModel ?? (this.machineJson?.printerModel || 'X1'),
      bc_x0: this.isNumber(this.idex.e0.compX) ? (this.idex.e0.compX as number) : 0,
      bc_y0: this.isNumber(this.idex.e0.compY) ? (this.idex.e0.compY as number) : 0,
      bc_x1: this.isNumber(this.idex.e1.compX) ? (this.idex.e1.compX as number) : 0,
      bc_y1: this.isNumber(this.idex.e1.compY) ? (this.idex.e1.compY as number) : 0,
    }
    return base
  }

  private extractContent(rpcRes: any): string {
    if (rpcRes == null) return ''
    if (typeof rpcRes === 'string') return rpcRes
    const candidates = [
      rpcRes?.result?.content,
      rpcRes?.result?.contents,
      rpcRes?.content,
      rpcRes?.contents,
      rpcRes?.data,
      typeof rpcRes?.result === 'string' ? rpcRes.result : undefined,
    ]
    for (const c of candidates) {
      if (typeof c === 'string' && c.length) return c
    }
    return ''
  }

  private tryParseJson(text: string): SyncraftMachine | null {
    try { return JSON.parse(text) } catch (e) {
      // alguns Moonrakers retornam base64 em "content"
      try { return JSON.parse(atob(text)) } catch { return null }
    }
  }

  // —— Carregar JSON (via RPC, caminho fixo em config/) ——
  async loadMachineJson() {
    try {
      const filename = `config/${this.jsonFileName}`
      // @ts-ignore
      const rpcRes = await this.$store.dispatch('server/request', {
        method: 'server.files.read',
        params: { filename }
      })
      const text = this.extractContent(rpcRes)
      const json = this.tryParseJson(text || '')
      if (!json) throw new Error('Conteúdo vazio ou JSON inválido')

      this.machineJson = json
      this.jsonPathRoot = 'config'
      this.mapJsonToUI(json)
      this.$emit('notify', { type: 'success', message: `Config carregada (${filename}).` })
    } catch (e) {
      console.error('[syncraft] loadMachineJson error', e)
      this.$emit('notify', { type: 'error', message: 'Falha ao ler config em config/syncraft-machine.json (RPC).' })
    }
  }

  // —— Salvar JSON (via RPC) ——
  async saveMachineJson() {
    try {
      const json = this.mapUIToJson()
      const content = JSON.stringify(json, null, 2)
      const root = this.jsonPathRoot || 'config'
      const filename = `${root}/${this.jsonFileName}`
      // @ts-ignore
      await this.$store.dispatch('server/request', {
        method: 'server.files.write',
        params: { filename, content }
      })
      this.machineJson = json
      this.$emit('notify', { type: 'success', message: `Config salva (${filename}).` })
    } catch (e) {
      console.error('[syncraft] saveMachineJson error', e)
      this.$emit('notify', { type: 'error', message: 'Falha ao salvar config (RPC).' })
    }
  }

  // —— Aplicações de compensação ——
  async applyX1() {
    const cmds: string[] = []
    if (this.isNumber(this.x1.compX)) cmds.push(`SET_BACKLASH X={${(this.x1.compX as number).toFixed(3)}}`)
    if (this.isNumber(this.x1.compY)) cmds.push(`SET_BACKLASH Y={${(this.x1.compY as number).toFixed(3)}}`)
    if (!cmds.length) return
    await this.batchSend(cmds)
    if (this.persistOnApply) await this.saveMachineJson()
  }

  async applyIdex() {
    const cmds: string[] = []
    if (this.isNumber(this.idex.e0.compX)) cmds.push(`SET_BACKLASH T=E0 X={${(this.idex.e0.compX as number).toFixed(3)}}`)
    if (this.isNumber(this.idex.e0.compY)) cmds.push(`SET_BACKLASH T=E0 Y={${(this.idex.e0.compY as number).toFixed(3)}}`)
    if (this.isNumber(this.idex.e1.compX)) cmds.push(`SET_BACKLASH T=E1 X={${(this.idex.e1.compX as number).toFixed(3)}}`)
    if (this.isNumber(this.idex.e1.compY)) cmds.push(`SET_BACKLASH T=E1 Y={${(this.idex.e1.compY as number).toFixed(3)}}`)
    if (!cmds.length) return
    await this.batchSend(cmds)
    if (this.persistOnApply) await this.saveMachineJson()
  }

  // —— Calibração ——
  calibrate(model: 'x1'|'idex', axis: 'X'|'Y', tool?: 'E0'|'E1') {
    let script = ''
    if (model === 'x1') script = `CALIBRATE_X1_${axis}`
    else script = `CALIBRATE_IDEX_${axis}_${tool ?? 'E0'}`
    this.sendScript(script)
  }

  // —— Ações ——
  async resetParameters() {
    this.x1.compX = this.x1.compY = 0
    this.idex.e0.compX = this.idex.e0.compY = 0
    this.idex.e1.compX = this.idex.e1.compY = 0
    const cmds = [
      'SET_BACKLASH X={0}',
      'SET_BACKLASH Y={0}',
      'SET_BACKLASH T=E0 X={0}',
      'SET_BACKLASH T=E0 Y={0}',
      'SET_BACKLASH T=E1 X={0}',
      'SET_BACKLASH T=E1 Y={0}',
    ]
    await this.batchSend(cmds)
    if (this.persistOnApply) await this.saveMachineJson()
  }

  restartService() { this.sendScript('RESTART') }

  // —— Envio ——
  private async batchSend(cmds: string[]) { for (const c of cmds) await this.sendScript(c) }

  private async sendScript(script: string) {
    try {
      // @ts-ignore
      if (this.$store && this.$store.dispatch) {
        await this.$store.dispatch('server/request', {
          method: 'printer.gcode.script',
          params: { script },
        })
        this.$emit('notify', { type: 'success', message: 'Comando enviado.' })
        return
      }
    } catch (e) { console.error(e) }

    // Fallback
    // @ts-ignore
    if (this.$store && this.$store.dispatch) {
      // @ts-ignore
      await this.$store.dispatch('printer/send', script)
      this.$emit('notify', { type: 'success', message: 'Comando enviado (fallback).' })
    } else {
      console.warn('Envio de G-code não configurado. Adapte sendScript().')
      this.$emit('notify', { type: 'warning', message: 'Precisa configurar o envio de G-code.' })
    }
  }
}
</script>

<style scoped>
/* container centralizado com largura fluida */
.panel-container{max-width:1040px;margin:0 auto;padding:0 8px}

/* garantir que os campos não estourem em telas estreitas */
:deep(.v-text-field){min-width:0}
:deep(.v-input){min-width:0}
:deep(.v-col){min-width:0}

/* botões quebram para a próxima linha em telas pequenas */
:deep(.v-card .v-card-text){overflow:hidden}
:deep(.v-card .v-btn){margin-right:8px;margin-bottom:8px}

/* toolbar superior responsiva */
:deep(.v-alert .d-flex){flex-wrap:wrap}
</style>
