# Mainsail (Fork Syncraft) — Guia de Dev, Build e Instalação

Frontend do **Mainsail** com telas customizadas da Syncraft (ex.: **Configuração/Calibração** que lê/escreve o JSON e aciona **systemd** via **Moonraker**).

## Sumário

* [Pré-requisitos](#pré-requisitos)
* [Configuração do Moonraker (uma vez)](#configuração-do-moonraker-uma-vez)
* [Ambiente de desenvolvimento](#ambiente-de-desenvolvimento)
* [Build de produção](#build-de-produção)
* [Instalação na impressora (one-liner)](#instalação-na-impressora-one-liner)
* [Atualização / rollback](#atualização--rollback)
* [Dicas & solução de problemas](#dicas--solução-de-problemas)
* [Licença](#licença)

---

## Pré-requisitos

* **Node.js LTS ≥ 18**
* **npm** (ou **pnpm**/**corepack**, opcional)
* Impressora com **Moonraker** (e **Nginx** servindo `~/mainsail`)
* Na impressora: `curl`, `unzip` instalados

  ```bash
  sudo apt update && sudo apt install -y curl unzip
  ```

---

## Configuração do Moonraker (uma vez)

Necessária para que a página consiga **parar/reiniciar serviços** via API:

1. `moonraker.conf`

   ```ini
   [machine]
   provider: systemd_dbus    # (recomendado) ou systemd_cli
   ```
2. Regras do PolicyKit (DBus):

   ```bash
   cd ~/moonraker
   ./scripts/set-policykit-rules.sh
   ```
3. `~/printer_data/moonraker.asvc` (um serviço por linha, **sem** “.service”):

   ```
   klipper_mcu
   webcamd
   MoonCord
   KlipperScreen
   crowsnest
   syncraft-usb
   syncraft-backlash-watcher
   ```
4. Se for desenvolver via Vite (`:5173`), autorize CORS/trusted clients em `moonraker.conf`:

   ```ini
   [authorization]
   cors_domains:
       http://localhost:5173
       http://127.0.0.1:5173
       http://<seu-ip-lan>:5173
   trusted_clients:
       127.0.0.1
       192.168.0.0/16
       10.0.0.0/8
   ```
5. Reinicie o Moonraker:

   ```bash
   sudo systemctl restart moonraker
   ```

---

## Ambiente de desenvolvimento

1. **Instalar dependências**

   ```bash
   npm ci   # ou: npm install
   ```

   > Se preferir **pnpm**: `corepack enable && corepack prepare pnpm@latest --activate && pnpm i`

2. **Vars de ambiente (opcional)**
   Crie `.env` na raiz se quiser apontar explicitamente para o Moonraker:

   ```env
   VITE_MOONRAKER_URL=http://<IP-DA-IMPRESSORA>:7125
   VITE_SERVICE_NAME=syncraft-backlash-watcher   # sem ".service"
   ```

3. **Rodar em dev (HMR)**

   ```bash
   npm run dev
   ```

   Acesse: `http://<seu-ip>:5173`

   > Se desenvolver a partir de outra máquina, confira as entradas de `authorization` no Moonraker (CORS/Trusted).

---

## Build de produção

Gera a pasta `dist/` com os arquivos estáticos prontos para publicar em `~/mainsail`.

```bash
npm run build
```

> Opcional (gerar zip local, se tiver `zip` instalado):
>
> ```bash
> mkdir -p releases
> (cd dist && zip -qr9 ../releases/mainsail.zip . -x '**/.DS_Store')
> ```

Após o build, você pode **publicar** a pasta `dist/` manualmente (rsync/SSH) ou usar a **instalação one-liner** abaixo, que baixa um `.zip` do seu repositório e instala diretamente na impressora.

---

## Instalação na impressora (one-liner)

Este comando **baixa** o zip do GitHub, **limpa** `~/mainsail` e **descompacta** o build dentro dela.

> Pré-requisitos na impressora: `curl` e `unzip`.

```bash
ZIP_URL="https://github.com/Syncraft-Technologies/mainsail/blob/develop/dist/mainsail.zip"; TARGET="$HOME/mainsail"; RAW_URL=$(sed -E 's#https://github\.com/([^/]+)/([^/]+)/blob/([^/]+)/#https://raw.githubusercontent.com/\1/\2/\3/#' <<< "$ZIP_URL"); TMP=$(mktemp -d) && curl -fL --retry 3 -o "$TMP/mainsail.zip" "$RAW_URL" && unzip -tqq "$TMP/mainsail.zip" && mkdir -p "$TARGET" && find "$TARGET" -mindepth 1 -maxdepth 1 -exec rm -rf {} + && unzip -q "$TMP/mainsail.zip" -d "$TARGET" && rm -rf "$TMP"
```

* Altere `ZIP_URL` se publicar seu zip em outro branch/URL.
* O conteúdo do zip é extraído **dentro** de `~/mainsail`.

---

## Atualização / rollback

* **Atualizar**: gere um novo `mainsail.zip` e rode o **one-liner** novamente.
* **Rollback**: mantenha um zip anterior e reaplique o one-liner apontando para a URL do build anterior.

> Se usar Nginx com cache agressivo ou PWA, force refresh (Ctrl+F5) ou limpe cache do navegador ao trocar versões.

---

## Dicas & solução de problemas

* **`zip: not found` ao gerar zip local**: instale `zip` (`sudo apt install -y zip`) ou use uma solução Node-only (ex.: `bestzip`).
* **Erro CORS/401 no dev**: ajuste `[authorization]` do Moonraker (CORS/Trusted) para a origem `:5173`.
* **Serviço não reinicia via UI**:

  * Confira `provider` (`systemd_dbus` recomendado);
  * Execute `./scripts/set-policykit-rules.sh`;
  * Verifique se o serviço está em `moonraker.asvc` **sem** “.service”;
  * Veja logs:

    ```bash
    sudo journalctl -u moonraker -f
    sudo journalctl -u syncraft-backlash-watcher.service -f
    ```
* **Conteúdo não atualiza**: limpe cache do navegador (PWA) ou recarregue o Nginx: `sudo systemctl reload nginx`.

---

## Licença

Este fork segue a licença do repositório. Veja `LICENSE.md`.
