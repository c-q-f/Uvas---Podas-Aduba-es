# CLAUDE.md — Granja Coemitanga / Uvas-Podas-Adubações

Painel de controle de podas e adubações de parreiras de uva niágara da Granja Coemitanga.

## Arquitetura

- **SPA em um único arquivo HTML autocontido** ([index.html](index.html)).
- Todo CSS dentro de `<style>`, todo JS dentro de `<script>`.
- Sem build, sem `node_modules`, sem backend. Abre direto no navegador (inclusive via `file://`).
- Bibliotecas externas via **`cdnjs.cloudflare.com`** apenas (Chart.js, SheetJS/xlsx).
- Firebase Realtime Database via CDN `gstatic.com` (v8.10.1).

## Regras técnicas inegociáveis

- **JavaScript ES5 puro.** Sem arrow functions, template literals, `const`/`let`, destructuring, spread ou qualquer sintaxe ES6+. Use `var`, `function`, concatenação de strings.
- **Fontes do sistema** (`-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Helvetica Neue', Arial, sans-serif`). Nunca Google Fonts (quebra CSP em `file://`).
- **`localStorage` sempre em try/catch** com fallback em cascata: `localStorage` → `sessionStorage` → objeto em memória.
- **Não usar `fi.click()`** para abrir file input (bloqueado em iframes/file://). Use `<label for="file-input">` como dropzone.
- **Não usar `<select>` nativo oculto** sobreposto a botões customizados. Implemente dropdowns 100% em JS/HTML.
- Sempre cheque elementos com `if (!gel('id')) return;` antes de manipular (evita `Cannot set properties of null`).
- Tooltips: container com `overflow:visible`, background em elemento filho `position:absolute;inset:0`.

## Regras de negócio

- **62 ruas** numeradas de 2 a 63, **70 plantas por rua** (total 4.340 plantas).
- **Poda Longa (produção):** 10d → 100g Calcinit · 30d → 70g Calcinit + 100g TrichoPhos · 45d → 50g Calcinit.
- **Poda Curta (formação):** 10d → 100g Calcinit.
- Quantidade total por rua = dose por planta × 70.
- Apresentar em **kg** quando ≥ 1.000g, senão em **g** (pt-BR, vírgula decimal).
- **Status das aplicações:** Pendente (≥ hoje+8d) · Próxima (hoje a hoje+7d) · Vencida (< hoje, não concluída) · Concluída (manual).

## Paleta (dark mode)

Fundo `#080c08`/`#141914` · Superfícies `#1e261e`/`#252d25` · Primária `#2d5a3d` · Acento `#a8e063` · Texto `#f0f4f0`/`#b8c4b8`/`#7a8c7a` · Verde `#4ade80` · Âmbar `#fbbf24` · Vermelho `#f87171` · Azul `#60a5fa` · Mapa parreira longa `#6BAFC2` · Mapa parreira curta `#B8B090`.

## Persistência

- Wrapper `store` (get/set/del) com fallback `localStorage` → `sessionStorage` → memória, tudo em try/catch.
- **Firebase Realtime Database** para sincronização em tempo real entre máquinas. Conexão automática na abertura, retry em 4s/10s, dashboard abre imediatamente com dados locais.
- Botões de Exportar JSON · Importar JSON · Limpar (com confirmação) · Nova planilha.

## Importação de planilhas

- Aceita `.xlsx`, `.xls`, `.csv`.
- Escanear primeiras 10 linhas célula a célula buscando `rua`, `poda`, `tipo` para achar o cabeçalho real (ignora títulos mesclados).
- Escanear todas as abas.
- Cabeçalhos aceitos: `Rua` · `Poda`/`Data da Poda`/`Data Poda`/`Dt Poda` · `Tipo`/`Tipo de Poda`/`Tipo Poda` (case-insensitive, com ou sem acentos).
- Datas: serial Excel, `dd/mm/aaaa`, `aaaa-mm-dd`.
- Tipo: `Longa`/`L`/`Curta`/`C`.
- Rua fora do range 2–63 → ignorar com aviso. Coluna obrigatória ausente → bloquear importação.

## O que NÃO fazer

- Sem login, multi-usuário, APIs externas, PWA, service worker, backend.
- Sem ES6+ em nenhuma circunstância.
- Sem Google Fonts.
- Sem KPI "Próxima aplicação" ou "Aplicações" (substituídos por "Adubações Atrasadas").
- Sem badge de status da semana, sem barras de progresso nos KPIs de Ruas Podadas / Distribuição de Podas.
- Não criar arquivos `.md` ou novos arquivos sem necessidade — o projeto é intencionalmente um único `index.html`.

## Referência completa da especificação

Sempre que houver dúvida sobre KPIs, layout, tabelas ou tooltips, consultar [prompt_coemitanga_uva_adubo.md](prompt_coemitanga_uva_adubo.md) — é a fonte canônica da especificação.

## Roadmap Firebase

Histórico e plano de integração em [firebase_roadmap.md](firebase_roadmap.md).
