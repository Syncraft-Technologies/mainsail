<template>
  <div class="pa-4">
    <!-- Header -->
    <v-row class="mb-4">
      <v-col cols="12">
        <v-alert dense outlined type="info">
          <div class="d-flex align-center justify-space-between flex-wrap">
            <div>
              <strong>Modelo detectado:</strong>
              <span v-if="printerModel">{{ printerModel }}</span>
              <span v-else>—</span>
              <span v-if="jsonPathUsed" class="ml-3">
                <strong>Arquivo:</strong> {{ jsonPathUsed }}
              </span>
            </div>
            <div class="mt-2 mt-sm-0">
              <v-btn x-small class="mr-2" @click="loadMachineJson">Recarregar</v-btn>
              <v-btn x-small :disabled="!dirty" @click="revertChanges">Descartar alterações</v-btn>
            </div>
          </div>
        </v-alert>
      </v-col>
    </v-row>

    <!-- Painel X1 -->
    <v-row v-if="modelKind === 'X1'">
      <v-col cols="12" sm="12" md="8" lg="7" class="mx-auto">
        <v-card elevation="2">
          <v-card-title class="text-h6">Compensação — X1 (Extrusor Único)</v-card-title>
          <v-card-text>
            <v-row>
              <v-col cols="12" sm="6" class="min-w-0">
                <v-text-field
                  v-model.number="x1.compX"
                  type="number" step="0.01"
                  label="Backlash X (mm)" suffix="mm" dense hide-details="auto"
                />
              </v-col>
              <v-col cols="12" sm="6" class="min-w-0">
                <v-text-field
                  v-model.number="x1.compY"
                  type="number" step="0.01"
                  label="Backlash Y (mm)" suffix="mm" dense hide-details="auto"
                />
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Painel IDEX -->
    <v-row v-else-if="modelKind === 'IDEX'">
      <v-col cols="12" sm="12" md="10" lg="9" class="mx-auto">
        <v-card elevation="2">
          <v-card-title class="text-h6">Compensação — IDEX (E0 / E1)</v-card-title>
          <v-card-text>
            <v-row>
              <v-col cols="12" sm="6" class="min-w-0">
                <v-text-field
                  v-model.number="idex.e0.compX"
                  type="number" step="0.01"
                  label="E0 — Backlash X (mm)" suffix="mm" dense hide-details="auto"
                />
              </v-col>
              <v-col cols="12" sm="6" class="min-w-0">
                <v-text-field
                  v-model.number="idex.e0.compY"
                  type="number" step="0.01"
                  label="E0 — Backlash Y (mm)" suffix="mm" dense hide-details="auto"
                />
              </v-col>
            </v-row>

            <v-row>
              <v-col cols="12" sm="6" class="min-w-0">
                <v-text-field
                  v-model.number="idex.e1.compX"
                  type="number" step="0.01"
                  label="E1 — Backlash X (mm)" suffix="mm" dense hide-details="auto"
                />
              </v-col>
              <v-col cols="12" sm="6" class="min-w-0">
                <v-text-field
                  v-model.number="idex.e1.compY"
                  type="number" step="0.01"
                  label="E1 — Backlash Y (mm)" suffix="mm" dense hide-details="auto"
                />
              </v-col>
            </v-row>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>

    <!-- Se modelo não reconhecido -->
    <v-row v-else>
      <v-col cols="12" md="8" class="mx-auto">
        <v-alert type="warning" outlined>
          Modelo de impressora não reconhecido no JSON. Valor atual: <strong>{{ printerModel || 'indefinido' }}</strong>.
        </v-alert>
      </v-col>
    </v-row>

    <!-- Ações -->
    <v-row class="mt-4">
      <v-col cols="12">
        <v-card elevation="2">
          <v-card-title class="text-h6">Ações</v-card-title>
          <v-card-text>
            <v-alert dense text type="info" class="mb-4">
              Salvar sempre grava no JSON e reinicia o serviço na impressora.
            </v-alert>
            <div class="btn-wrap">
              <v-btn class="mr-2 mb-2" @click="resetAndRestart">Zerar parâmetros e reiniciar</v-btn>
              <v-btn color="primary" class="mb-2" :disabled="!canSave" @click="saveAndRestart">
                Salvar parâmetros e reiniciar
              </v-btn>
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

type ModelKind = 'X1' | 'IDEX' | 'UNKNOWN'
interface AxisPair { compX: number; compY: number }
interface SyncraftMachine {
  printerModel?: string
  bc_x0?: number
  bc_x1?: number
  bc_y0?: number
  bc_y1?: number
  [k: string]: any
}

@Component({ name: 'SyncraftCalibration' })
export default class SyncraftCalibration extends Vue {
  // estado da UI
  printerModel: string | null = null
  jsonPathUsed: string | null = null
  dirty = false

  // edição
  x1: AxisPair = { compX: 0, compY: 0 }
  idex = {
    e0: { compX: 0, compY: 0 },
    e1: { compX: 0, compY: 0 },
  }

  // cache do json
  machineJson: SyncraftMachine | null = null

  // candidatos de caminho (tenta na ordem)
  private candidates = [
    '~/syncraft-machine.json',
    'home/syncraft-machine.json',
    'config/syncraft-machine.json',
    'printer_data/config/syncraft-machine.json',
  ]

  async mounted() { await this.loadMachineJson() }

  // computed
  get modelKind(): ModelKind {
    const p = (this.printerModel || '').toUpperCase()
    if (p === 'X1') return 'X1'
    if (p.includes('IDEX') || p.includes('SYNC')) return 'IDEX'
    return 'UNKNOWN'
  }
  get canSave() { return !!this.machineJson }

  // —— carga/salvamento ————————————————————————
  private extractContent(rpcRes: any): string {
    if (rpcRes == null) return ''
    if (typeof rpcRes === 'string') return rpcRes
    const cands = [
      rpcRes?.result?.content,
      rpcRes?.result?.contents,
      rpcRes?.content,
      rpcRes?.contents,
      rpcRes?.data,
      typeof rpcRes?.result === 'string' ? rpcRes.result : undefined,
    ]
    for (const c of cands) if (typeof c === 'string' && c.length) return c
    return ''
  }

  private tryParseJson(s: string): SyncraftMachine | null {
    try { return JSON.parse(s) } catch {
      try { return JSON.parse(atob(s)) } catch { return null }
    }
  }

  // Lê variáveis em Vite (VITE_*) e Vue CLI (VUE_APP_*)
  private readEnv(name: string): string | undefined {
    let meta: any; try { meta = (import.meta as any)?.env } catch {}
    const penv: any = (typeof process !== 'undefined' && (process as any)?.env) ? (process as any).env : undefined
    return (meta && meta[name] !== undefined) ? meta[name] : (penv ? penv[name] : undefined)
  }

  // Bases: ENV explícita → relativo (se proxy existir)
  private moonrakerBases(): string[] {
    // Dê preferência a uma URL completa (mais simples)
    const full = this.readEnv('VITE_MOONRAKER_URL') || this.readEnv('VUE_APP_MOONRAKER_URL')
    const host = this.readEnv('VUE_APP_HOSTNAME') || this.readEnv('VITE_MOONRAKER_HOST')
    const port = this.readEnv('VUE_APP_PORT')      || this.readEnv('VITE_MOONRAKER_PORT') || '7125'
    const path = (this.readEnv('VUE_APP_PATH')     || this.readEnv('VITE_MOONRAKER_PATH') || '/').replace(/\/+$/,'')

    const bases: string[] = []
    if (full) bases.push(full.replace(/\/+$/, ''))
    else if (host) bases.push(`http://${host}${port ? ':' + port : ''}${path}`)
    // relativa por último (só funciona se o Nginx proxyar /server)
    bases.push('')
    return bases
  }

  // fetch com fallback (ENV → relativo), recusando HTML (index/404)
  private async fetchMoonraker(path: string, init?: RequestInit) {
    const p = path.startsWith('/') ? path : `/${path}`
    const isHtml = (ct: string | null, body: string) =>
      (ct || '').toLowerCase().includes('text/html') ||
      body.trim().toLowerCase().startsWith('<!doctype') ||
      body.trim().startsWith('<html')

    let lastErr: any
    const bases = this.moonrakerBases()
    console.info('[syncraft] bases para Moonraker:', bases)
    for (const base of bases) {
      const url = base ? `${base}${p}` : p
      try {
        const r = await fetch(url, init)
        const t = await r.text()
        if (r.ok && !isHtml(r.headers.get('content-type'), t)) {
          return new Response(t, { status: r.status, headers: r.headers })
        }
        console.warn(`[syncraft] ${url} retornou HTML/erro (${r.status}).`)
      } catch (e) {
        lastErr = e
        console.warn(`[syncraft] falha no fetch ${url}`, e)
      }
    }
    throw lastErr || new Error('Falha no fetchMoonraker')
  }


  // LER via HTTP (download)
  // Troque seu loadMachineJson por este:
  async loadMachineJson() {
    try {
      const resp = await this.fetchMoonraker('server/files/config/syncraft-machine.json?download=1')
      const text = await resp.text()

      let json: any
      try { json = JSON.parse(text) }
      catch { json = JSON.parse(atob(text)) } // se algum servidor mandar base64

      this.machineJson  = json
      this.jsonPathUsed = 'config/syncraft-machine.json'
      this.mapJsonToUI(json)
      this.dirty = false
      this.$emit('notify', { type: 'success', message: 'Config carregada (config/syncraft-machine.json).' })
    } catch (e) {
      console.error('[syncraft] loadMachineJson', e)
      this.$emit('notify', { type: 'error', message: 'Falha ao ler config (HTTP). Veja o console para detalhes.' })
    }
  }



  private mapJsonToUI(json: SyncraftMachine) {
    this.printerModel = json.printerModel ?? 'X1'
    // preencher X1 a partir de E0
    this.x1.compX = Number(json.bc_x0 ?? 0) || 0
    this.x1.compY = Number(json.bc_y0 ?? 0) || 0
    // preencher IDEX
    this.idex.e0.compX = Number(json.bc_x0 ?? 0) || 0
    this.idex.e0.compY = Number(json.bc_y0 ?? 0) || 0
    this.idex.e1.compX = Number(json.bc_x1 ?? 0) || 0
    this.idex.e1.compY = Number(json.bc_y1 ?? 0) || 0
  }

  private mapUIToJson(): SyncraftMachine {
    const base = { ...(this.machineJson || {}) }
    base.printerModel = this.printerModel || 'X1'
    // sempre persistimos os 4 campos (E0/E1)
    base.bc_x0 = Number(this.idex.e0.compX) || 0
    base.bc_y0 = Number(this.idex.e0.compY) || 0
    base.bc_x1 = Number(this.idex.e1.compX) || 0
    base.bc_y1 = Number(this.idex.e1.compY) || 0
    return base
  }

  

  // SALVAR via HTTP (upload) — sobrescreve arquivo em config/
  async saveJson() {
    try {
      const json = this.mapUIToJson()
      const blob = new Blob([JSON.stringify(json, null, 2)], { type: 'application/json' })
      const form = new FormData()
      form.append('file', blob, 'syncraft-machine.json')
      form.append('root', 'config')

      try {
        await this.fetchMoonraker('server/files/upload', { method: 'POST', body: form })
      } catch {
        await this.fetchMoonraker('server/files/config/syncraft-machine.json', { method: 'DELETE' })
        await this.fetchMoonraker('server/files/upload', { method: 'POST', body: form })
      }

      this.machineJson  = json
      this.jsonPathUsed = 'config/syncraft-machine.json'
      this.dirty = false
      this.$emit('notify', { type: 'success', message: 'Config salva (config/syncraft-machine.json).' })
    } catch (e) {
      console.error('[syncraft] saveJson', e)
      this.$emit('notify', { type: 'error', message: 'Falha ao salvar config (HTTP). Veja o console para detalhes.' })
    }
  }



  // —— ações ————————————————————————————————
  async resetAndRestart() {
    // zera tudo na UI
    this.x1.compX = this.x1.compY = 0
    this.idex.e0.compX = this.idex.e0.compY = 0
    this.idex.e1.compX = this.idex.e1.compY = 0
    this.dirty = true

    // grava zeros no JSON e reinicia serviço
    await this.saveJson()
    await this.restartService()
  }

  async saveAndRestart() {
    await this.saveJson()
    await this.restartService()
  }

  revertChanges() {
    if (this.machineJson) this.mapJsonToUI(this.machineJson)
    this.dirty = false
  }

  async restartService() {
    try {
      // envia G-code para reiniciar Klippy
      // @ts-ignore
      await this.$store.dispatch('server/request', {
        method: 'printer.gcode.script',
        params: { script: 'RESTART' }
      })
      this.$emit('notify', { type: 'success', message: 'Serviço reiniciado.' })
    } catch {
      this.$emit('notify', { type: 'error', message: 'Falha ao reiniciar serviço.' })
    }
  }

  // watchers simples para marcar alteração
  watch = {
    x1: { handler: () => (this.dirty = true), deep: true },
    idex: { handler: () => (this.dirty = true), deep: true },
  } as any
}
</script>

<style scoped>
.min-w-0 { min-width: 0; }
.btn-wrap { display: flex; flex-wrap: wrap; }
.btn-wrap .v-btn { margin-right: 8px; margin-bottom: 8px; }
</style>
