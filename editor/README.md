# Editor Visual de Certificado (v05)

Editor de composição em **3 camadas gráficas** para certificados — imagem-base,
elemento intermediário e elemento superior — com ajustes em tempo real e
exportação PNG/PDF. Processamento 100% local no navegador; nada é enviado a
servidor algum.

**Em produção:** <https://carlosrosset.com.br/samples/editor/>

## O que faz

- **3 camadas independentes** sobre uma imagem-base (certificado), renderizadas
  proporcionalmente em tempo real por Canvas API
- **Elemento superior** (ex.: selo dourado): opacidade 0–100%, brilho
  independente, posição X/Y, escala proporcional
- **Elemento intermediário** (ex.: logo do hospital): opacidade 0–100%,
  posição X/Y, escala proporcional
- **Restaurar** configurações iniciais com um toque
- **Exportar** a composição em **PNG** ou **PDF** (A4 horizontal), na resolução
  original da imagem-base — independente da tela do dispositivo
- Posição e escala também aceitam digitação direta (além dos sliders)

## Camadas padrão (embarcadas)

| Camada | Arquivo | Origem |
|---|---|---|
| Base | `certificado_base_sem_selo_e_marca_dagua.png` (2.8 MB) | zip do projeto |
| Intermediária | `logo_sao_lucas.svg` | zip do projeto |
| Superior | `selo_dourado_republica_federativa_brasil.png` (1.7 MB) | zip do projeto |

As imagens do projeto ficam em `assets/` (copiadas byte-a-byte do zip
`imagens_projeto_organizadas.zip` do Drive — SHA-256 conferido no deploy).

## Stack

| Camada | O quê |
|---|---|
| Frontend | HTML/CSS/JS único (`index.html`), zero dependências, zero CDN |
| Renderização | Canvas API (`drawImage`, `globalAlpha`, `filter: brightness` com fallback manual) |
| Exportação PNG | `canvas.toBlob` na resolução natural da base |
| Exportação PDF | escritor `%PDF-1.4` em JS puro (sem jsPDF, sem rede) |
| Hospedagem | GitHub Pages (`CarlosRosset/samples`, branch `gh-pages`) sob `carlosrosset.com.br` |

## 🧹 O sistema de limpeza de cache (por que e como funciona)

**O problema:** este site vive sob `carlosrosset.com.br`, cujo **service worker da raiz**
(Workbox, estratégia `CacheFirst` para CSS e imagens) e o **cache HTTP do navegador**
(`max-age=600`) podem servir versões antigas do editor e das imagens — o visitante vê
o fundo/texto velho mesmo com o site novo no ar.

**As 3 camadas de cache e suas curas (cada uma com ferramenta própria):**

| Camada | O que é | Como invalida |
|---|---|---|
| **1. HTTP cache** | o navegador guarda o PNG/CSS por 10 min | URLs **versionadas**: os assets são pedidos como `assets/x.png?r=REV` — a constante `ASSET_REV` no `index.html` é **bumpada a cada troca de asset**; URL nova = nenhum cache a segura |
| **2. Service worker** (Workbox da raiz) | intercepta pedidos e responde do cache ANTES da rede | o botão **🐛** no header e a página `limpar.html` fazem `unregister()` de todos os SWs + `caches.delete()` de todos os caches nomeados |
| **3. Storage do origin** | localStorage, sessionStorage, IndexedDB | o 🐛/`limpar.html` executam `clear()`/`deleteDatabase()` em tudo |

**As 2 vias de limpeza:**

- **O botão 🐛** (header, canto direito) — "limpar super" dentro do próprio editor:
  confirm → desregistra SWs → apaga caches → limpa storage → apaga IndexedDB →
  recarrega com `?v=timestamp`. Com rastro `[bugReset]` no console (F12 para auditar).
  Herdeiro direto do botão `ios-bug` / `forceResetApp()` da home antiga do site (2018).
- **`limpar.html`** (página standalone) — a mesma faxina, sem confirm, redirecionando
  sozinha para o editor. Útil quando a própria página está corrompida em cache.

**A hierarquia de uso:** o `ASSET_REV` resolve 95% dos casos sozinho (o visitante comum
nunca precisa limpar nada). O 🐛 é para o usuário que chegou num estado velho de outra
época. O `limpar.html` é o plano C — quando nem o botão é confiável.

**Histórico da descoberta (27–28/11/2026):** após trocar o fundo do certificado (V1→V2→V3),
o dono continuou vendo a V2 mesmo com a V3 no ar e com `?v=NN` na URL — o service worker
da raiz respondia o PNG do próprio cache. Diagnóstico por eliminação (anônima via ✓ →
normal ✗ → o SW é o vilão) levou às 3 curas acima: `ASSET_REV` (a19cc7f), `limpar.html`
(29eddd6), botão 🐛 (c914d46 + rastro de console 8208db4). Homologado pelo dono em 28/11
("deu certo e vi certo").

## Histórico de versões

A homologação adotou o padrão **versão única em vigor**: a versão aprovada
vive na raiz `editor/` e as anteriores deixam o ar (o git preserva tudo).

- **v15 (atual, 28/11/2026)** — fundo V3 do certificado (via Drive, 3 iterações do dono);
  sistema de limpeza de cache completo (🐛 + limpar.html + ASSET_REV); badge v15
- v05 (homologada 27/11/2026) — título "Editor visual de certificado",
  badge "Versão • v05"; imagens reais do projeto como padrão; sem uploads, sem
  avisos de demonstração
- v01–v04 — evolução interna do dia 27/11 (uploads manuais → assets reais →
  limpeza de textos → fix de listeners). Recuperáveis pelo histórico do git
  (`99d203a`…`3b997d8`)

## Homologação

Funciona em Chrome para Android, Safari para iOS e navegadores desktop.
Preservação da resolução original da base, transparência real, formatos
PNG/SVG/WebP, atualização imediata dos controles.

## Notas de manutenção

- **Não há segredos nem env** — tudo estático e público por natureza
- Deploy = commit na `gh-pages` (manobra segura: branch nova → push →
  `merge --ff-only` → push; nunca force-push na gh-pages)
- Validação antes de subir: servidor local + inspeção de **pixels do canvas**
  via DevTools (`getImageData`) — screenshot sozinho não prova renderização
- O JS referencia os inputs de arquivo por concatenação (`k + 'File'`) com
  guard (`if (!el) continue;`) — os inputs foram removidos da UI na v05, o
  motor permanece pronto caso voltem
- Fonte dos elementos gráficos: `imagens_projeto_organizadas.zip` (Drive do
  Carlos, pasta do projeto certificados Hospital São Lucas)
