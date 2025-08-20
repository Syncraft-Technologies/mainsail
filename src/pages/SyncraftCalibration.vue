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
            </div>
            <div class="mt-2 mt-sm-0">
              <v-btn x-small class="mr-2" :loading="busy" @click="loadMachineJson">Recarregar</v-btn>
            </div>
          </div>
        </v-alert>
      </v-col>
    </v-row>

    <!-- Painel X1 -->
    <v-row v-if="modelKind === 'X1'">
      <v-col cols="12">
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
      <v-col cols="12">
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
      <v-col cols="12">
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
            <div class="btn-wrap">
              <v-btn class="mr-2 mb-2" :loading="busy" :disabled="busy" @click="stopAndZero">
                Parar serviço e zerar parâmetros
              </v-btn>
              <v-btn color="blue-darken-3" class="mb-2" :loading="busy" :disabled="!canSave || busy" @click="saveAndRestart">
                Salvar parâmetros e reiniciar serviço
              </v-btn>
            </div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'vue-property-decorator'

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
  dirty = false
  busy = false

  // edição
  x1: AxisPair = { compX: 0, compY: 0 }
  idex = {
    e0: { compX: 0, compY: 0 },
    e1: { compX: 0, compY: 0 },
  }

  // cache do json
  machineJson: SyncraftMachine | null = null

  async mounted() { await this.loadMachineJson() }

  // computed
  get modelKind(): ModelKind {
    const p = (this.printerModel || '').toUpperCase().trim()
    if (p === 'X1') return 'X1'
    if (p.includes('IDEX')) return 'IDEX'
    return 'UNKNOWN'
  }
  get canSave() { return !!this.machineJson }

  // ===== Helpers de ambiente =====
  private readEnv(name: string): string | undefined {
    let meta: any; try { meta = (import.meta as any)?.env } catch {}
    const penv: any = (typeof process !== 'undefined' && (process as any)?.env) ? (process as any).env : undefined
    return (meta && meta[name] !== undefined) ? meta[name] : (penv ? penv[name] : undefined)
  }

  // ===== Base URL do Moonraker =====
  private moonrakerBases(): string[] {
    const full = this.readEnv('VITE_MOONRAKER_URL') || this.readEnv('VUE_APP_MOONRAKER_URL')
    const host = this.readEnv('VUE_APP_HOSTNAME') || this.readEnv('VITE_MOONRAKER_HOST')
    const port = this.readEnv('VUE_APP_PORT')      || this.readEnv('VITE_MOONRAKER_PORT') || '7125'
    const path = (this.readEnv('VUE_APP_PATH')     || this.readEnv('VITE_MOONRAKER_PATH') || '/').replace(/\/+$/,'')
    const bases: string[] = []
    if (full) bases.push(full.replace(/\/+$/, ''))
    else if (host) bases.push(`http://${host}${port ? ':' + port : ''}${path}`)
    // relativa por último (proxy de /server habilitado)
    bases.push('')
    return bases
  }

  // ===== fetch com fallback e cache-busting =====
  private async fetchMoonraker(path: string, init: RequestInit = {}) {
    const method = (init.method || 'GET').toString().toUpperCase()
    const p0 = path.startsWith('/') ? path : `/${path}`
    const p = method === 'GET'
      ? `${p0}${p0.includes('?') ? '&' : '?'}_ts=${Date.now()}`
      : p0

    const isHtml = (ct: string | null, body: string) =>
      (ct || '').toLowerCase().includes('text/html') ||
      body.trim().toLowerCase().startsWith('<!doctype') ||
      body.trim().startsWith('<html')

    let lastErr: any
    for (const base of this.moonrakerBases()) {
      const url = base ? `${base}${p}` : p
      try {
        const r = await fetch(url, {
          credentials: 'include',
          cache: 'no-store',
          headers: { 'Cache-Control': 'no-cache', ...(init.headers || {}) },
          ...init,
        })
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

  // ===== Nome do serviço (sem .service) =====
  private serviceId(): string {
    const raw = this.readEnv('VITE_SERVICE_NAME') || 'syncraft-backlash-watcher'
    return raw.replace(/\.service$/i, '')
  }

  // ===== Chamada de ação de serviço no Moonraker =====
  private async postServiceAction(action: 'restart' | 'stop') {
    const body = JSON.stringify({ service: this.serviceId() })
    const res = await this.fetchMoonraker(`machine/services/${action}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body
    })
    if (!res.ok) {
      let info = ''
      try { info = await res.text() } catch {}
      throw new Error(`${action} failed ${info ? `: ${info}` : ''}`)
    }
  }

  // ===== Carga/Salvamento do JSON =====
  async loadMachineJson() {
    try {
      this.busy = true
      const resp = await this.fetchMoonraker('server/files/config/syncraft-machine.json?download=1')
      const text = await resp.text()
      let json: SyncraftMachine
      try { json = JSON.parse(text) } catch { json = JSON.parse(atob(text)) }

      this.machineJson  = json
      this.mapJsonToUI(json)
      this.dirty = false
      this.$emit('notify', { type: 'success', message: 'Config carregada.' })
    } catch (e) {
      console.error('[syncraft] loadMachineJson', e)
      this.$emit('notify', { type: 'error', message: 'Falha ao ler config (HTTP). Veja o console para detalhes.' })
    } finally {
      this.busy = false
    }
  }

  private mapJsonToUI(json: SyncraftMachine) {
    this.printerModel = json.printerModel ?? 'X1'
    const e0x = Number(json.bc_x0 ?? 0) || 0
    const e0y = Number(json.bc_y0 ?? 0) || 0
    const e1x = Number(json.bc_x1 ?? e0x) || 0
    const e1y = Number(json.bc_y1 ?? e0y) || 0

    this.x1.compX = e0x
    this.x1.compY = e0y

    this.idex.e0.compX = e0x
    this.idex.e0.compY = e0y
    this.idex.e1.compX = e1x
    this.idex.e1.compY = e1y
  }

  private mapUIToJson(): SyncraftMachine {
    const base: SyncraftMachine = { ...(this.machineJson || {}) }
    base.printerModel = this.printerModel || 'X1'

    if (this.modelKind === 'X1') {
      const x = Number(this.x1.compX) || 0
      const y = Number(this.x1.compY) || 0
      base.bc_x0 = x; base.bc_y0 = y
      base.bc_x1 = x; base.bc_y1 = y
    } else {
      base.bc_x0 = Number(this.idex.e0.compX) || 0
      base.bc_y0 = Number(this.idex.e0.compY) || 0
      base.bc_x1 = Number(this.idex.e1.compX) || 0
      base.bc_y1 = Number(this.idex.e1.compY) || 0
    }
    return base
  }

  async saveJson() {
    try {
      this.busy = true
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
      this.dirty = false
      this.$emit('notify', { type: 'success', message: 'Config salva.' })
    } catch (e) {
      console.error('[syncraft] saveJson', e)
      this.$emit('notify', { type: 'error', message: 'Falha ao salvar config (HTTP). Veja o console para detalhes.' })
    } finally {
      this.busy = false
    }
  }

  // ===== Ações =====
  async stopService() {
    await this.postServiceAction('stop')
  }

  async stopAndZero() {
    this.busy = true
    try {
      // 1) Parar serviço
      await this.stopService()
      // 2) Zerar UI
      this.x1.compX = this.x1.compY = 0
      this.idex.e0.compX = this.idex.e0.compY = 0
      this.idex.e1.compX = this.idex.e1.compY = 0
      this.dirty = true
      // 3) Salvar JSON zerado
      await this.saveJson()
      this.$emit('notify', { type: 'success', message: 'Serviço parado e parâmetros zerados.' })
    } catch (e: any) {
      console.error('[stopAndZero]', e)
      this.$emit('notify', { type: 'error', message: `Falha ao parar/zerar: ${e?.message || e}` })
    } finally {
      this.busy = false
    }
  }

  async saveAndRestart() {
    this.busy = true
    try {
      await this.saveJson()
      await this.postServiceAction('restart')
      this.$emit('notify', { type: 'success', message: 'Config salva e serviço reiniciado.' })
    } catch (e: any) {
      console.error('[saveAndRestart]', e)
      this.$emit('notify', { type: 'error', message: `Falha ao reiniciar serviço: ${e?.message || e}` })
    } finally {
      this.busy = false
    }
  }

  // —— watchers (classe) ——————————————————————
  @Watch('x1', { deep: true })
  onX1Changed() { this.dirty = true }

  @Watch('idex', { deep: true })
  onIdexChanged() { this.dirty = true }
}
</script>

<style scoped>
.min-w-0 { min-width: 0; }
.btn-wrap { display: flex; flex-wrap: wrap; }
.btn-wrap .v-btn { margin-right: 8px; margin-bottom: 8px; }
</style>
