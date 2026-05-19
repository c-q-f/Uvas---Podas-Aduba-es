# PAPEL

Você é um especialista sênior em visualização de dados e UI/UX design, com expertise comprovada em dashboards executivos de nível profissional usando HTML, CSS e JavaScript.

Seu objetivo: criar uma aplicação web moderna, altamente visual, elegante e responsiva para gerenciamento agrícola de fertilização de parreiras de uva niágara.

---

# REQUISITO DE ARQUITETURA

O sistema DEVE ser entregue como uma **SPA (Single Page Application) em um único arquivo HTML autocontido**:

- Todo CSS dentro de `<style>`
- Todo JavaScript dentro de `<script>`, escrito em **ES5 puro** (var, function, sem arrow functions, template literals, const/let, destructuring, spread, ou qualquer sintaxe ES6+) — para garantir parsing limpo e evitar erros de sintaxe em iterações de desenvolvimento
- Bibliotecas externas exclusivamente via CDN `cdnjs.cloudflare.com`
- Fontes do sistema (`-apple-system, BlinkMacSystemFont, 'Segoe UI', 'Helvetica Neue', Arial, sans-serif`) — sem Google Fonts (evita erros de CSP em `file://`)
- Funcional offline e compatível com abertura via `file://` no Edge/Chrome
- Sem dependência exclusiva de `localStorage` — implementar wrapper com fallback em cascata: `localStorage` → `sessionStorage` → objeto em memória

---

# CONTEXTO DO NEGÓCIO

**Granja Coemitanga** — produtora de uvas niágara.

- 62 ruas numeradas de **2 a 63**
- 70 plantas por rua
- Total: 4.340 plantas

---

# REGRAS DE NEGÓCIO — Cronograma de Adubação

## PODA LONGA (produção)

| Dias após poda | Fertilizantes por planta |
|---|---|
| 10 | 100g Calcinit |
| 30 | 70g Calcinit + 100g TrichoPhos |
| 45 | 50g Calcinit |

## PODA CURTA (formação)

| Dias após poda | Fertilizantes por planta |
|---|---|
| 10 | 100g Calcinit |

**Cálculos:**
- Quantidade total por rua = quantidade por planta × 70
- Apresentar em **kg** quando ≥ 1.000g, senão em **g** (pt-BR, vírgula decimal)

---

# ESPECIFICAÇÃO DA PLANILHA DE ENTRADA

Aceitar `.xlsx`, `.xls` e `.csv`.

**Cabeçalhos aceitos (case-insensitive, com ou sem acentos):**

| Campo | Nomes aceitos |
|---|---|
| Rua | `Rua` |
| Data da Poda | `Poda`, `Data da Poda`, `Data Poda`, `Dt Poda` |
| Tipo de Poda | `Tipo`, `Tipo de Poda`, `Tipo Poda` |

**Leitura robusta do Excel:**
- Escanear as primeiras 10 linhas célula a célula buscando os termos `rua`, `poda` ou `tipo` para localizar a linha de cabeçalho real — ignora títulos mesclados como `.: Registro de Podas`
- Escanear todas as abas do arquivo
- Aceitar datas em formato serial Excel (número), `dd/mm/aaaa` e `aaaa-mm-dd`
- Aceitar tipo de poda: `Longa`, `L`, `Curta`, `C`
- Mostrar relatório de validação com cabeçalhos detectados (debug útil para correção rápida de problemas de parsing)

**Validações:**
- Rua fora do range 2–63 → ignorar com aviso
- Data inválida → ignorar com aviso
- Tipo desconhecido → ignorar com aviso
- Coluna obrigatória ausente → bloquear importação

---

# PERSISTÊNCIA DE DADOS

**Wrapper de storage com fallback em cascata:**
```js
var _mem = {};
var store = {
  get: function(k) { /* tenta localStorage → sessionStorage → _mem */ },
  set: function(k,v) { /* grava em todos os três */ },
  del: function(k) { /* remove de todos */ }
};
```

Tudo encapsulado em try/catch — falhas silenciosas no `localStorage` (comum em `file://` no Edge) não quebram o app.

**Botões:**
- **Exportar** → baixa JSON
- **Importar** → carrega JSON
- **Limpar** → reseta estado (com confirmação)

---

# FUNCIONALIDADES

## 1. Upload de planilha
- `<label for="file-input">` como dropzone — **NÃO usar `fi.click()` via JS** (bloqueado em sandboxes)
- Input de arquivo com `position:absolute;width:1px;height:1px;opacity:0;pointer-events:none` (fora da label)
- Drag-and-drop com feedback visual

## 2. Filtro de semana ISO (barra fixa no topo)
- Exibe: código ISO (`2026-S21`), intervalo (`18/05/2026 – 24/05/2026`), número da semana
- **Ícone de funil** (`<polygon points="22 3 2 3 10 12.46 10 19 14 21 14 12.46 22 3">`) ao lado do intervalo abre **dropdown customizado em JS puro** — NÃO usar `<select>` nativo oculto (não funciona em iframes/file://)
- Dropdown com `min-width:320px; white-space:nowrap` para cada item caber em uma linha
- Botões: ← Anterior · Hoje · Próxima →
- Filtro de tipo de poda: pills **Todas · Longa · Curta**
  - Ativo: `background:#a8e063; color:#0a1a0a; border-color:#a8e063; font-weight:600`
  - Inativo: `background:transparent; color:#b8c4b8; border:1px solid rgba(168,224,99,0.2)`
- O filtro de semana afeta todos os painéis sensíveis; o filtro de tipo de poda afeta todos **exceto os KPIs**

## 3. KPIs (5 cards em linha única, sem painéis secundários abaixo)

| Card | Conteúdo |
|---|---|
| Ruas Podadas | `X% (N ruas)` + linha inferior `Faltam Y% (M ruas)` — base 62 ruas |
| Distribuição de Podas | `Longa X% (N ruas)` + linha inferior `Curta Y% (N ruas)` |
| Calcinit | Total do ciclo inteiro em kg |
| TrichoPhos | Total do ciclo inteiro em kg |
| Adubações Atrasadas | Nº de ruas com aplicações vencidas e não concluídas — vermelho se > 0, verde se = 0 |

## 4. Mapa da Parreira (largura total, abaixo dos KPIs)

Visualização horizontal das 62 ruas:
- Linha de números (2 a 63) no topo, fonte 8px
- Abaixo, barras coloridas de 56px de altura, gap de 2px entre colunas
- **Cores:**
  - Poda Longa → `#6BAFC2` (azul-aço) com opacity 0.65
  - Poda Curta → `#B8B090` (caqui) com opacity 0.65
  - Não podada → `#b8c4b8` com opacity 0.2
- **Tooltip** ao hover seguindo o mesmo estilo dos demais (fundo `var(--bg1)`, borda, sombra) — implementação crítica: container `.parreira-bar` com `overflow:visible` e elemento filho `.p-bar-bg` com `position:absolute;inset:0` para o background (caso contrário o tooltip é cortado)
- **Legenda em uma única linha** acima do mapa: `[●] X% Poda longa (N) · [●] X% Poda curta (N) · [●] X% Não podada (N)` — percentual na cor correspondente, rótulo e total em `#b8c4b8`

## 5. Visualizações

**Gráfico de barras empilhadas (Chart.js):**
- Calcinit + TrichoPhos por semana, semana ativa destacada

**Calendário operacional:**
- Coluna esquerda com número da semana ISO (S18, S19...)
- Semana ativa em verde (`#a8e063`), demais em `#4a5c4a`
- **Hoje:** borda âmbar 2px (`#fbbf24`) + fundo `rgba(251,191,36,0.10)` + número em âmbar negrito
- Dias com aplicações: pontos coloridos com **estilo inline** (`style="width:7px;height:7px;border-radius:50%;background:#a8e063"`) — não via classes CSS, para garantir visibilidade
- **Tooltip** ao hover por tipo de poda: Ruas + Adubação (1ª, 2ª...) na mesma linha; Calcinit e TrichoPhos abaixo

**Tabela de Planejamento de Compras de Fertilizantes:**
- Colunas: Semana · Ano · Data Adubação · Ruas · Nº Ruas · Tipo Poda · Adubação · Calcinit (kg) · TrichoPhos (kg)
- Uma linha por data + tipo de poda (Calcinit e TrichoPhos consolidados)
- Ruas agrupadas por faixas: `2–5, 10, 45–47`
- Adubação em ordinal: 1ª, 2ª, 3ª
- **Células de fertilizante**: total em kg na linha principal + dosagem por planta em cinza abaixo: `14,7 kg / (100 g/planta)`
- Filtrada pela semana ativa
- Botão exportar CSV (com BOM UTF-8)

**Tabela Operacional:**
- Colunas: Rua · Tipo · Data Poda · Data Aplicação · Fertilizante · g/Planta · Total Rua · Status · Ação
- Toggle "Ver ciclo inteiro"
- Filtros: status, fertilizante, tipo de poda
- Ordenação por coluna
- **Botão de conclusão**: 26×26px, círculo vazio (`○`) quando pendente, `✓` quando concluído — fundo verde no estado marcado
- Rodapé com subtotais (Nº ruas, kg Calcinit, kg TrichoPhos)

## 6. Status das aplicações
- **Pendente:** ≥ hoje + 8 dias
- **Próxima:** entre hoje e hoje + 7 dias
- **Vencida:** < hoje e não concluída
- **Concluída:** marcada manualmente

---

# DIRETRIZES DE UI/UX

**Estilo:** dark mode premium, glassmorphism leve, fontes do sistema

**Paleta:**
- Fundo: `#080c08`, `#141914`
- Superfícies: `#1e261e`, `#252d25`
- Primária: `#2d5a3d`
- Acento: `#a8e063`
- Texto: `#f0f4f0`, `#b8c4b8`, `#7a8c7a`
- Verde (concluído): `#4ade80`
- Âmbar (hoje / próximo): `#fbbf24`
- Vermelho (vencido): `#f87171`
- Azul (TrichoPhos / future): `#60a5fa`
- Mapa parreira longa: `#6BAFC2`
- Mapa parreira curta: `#B8B090`

---

# STACK TÉCNICA

- HTML5 + CSS3 + **JavaScript ES5 puro**
- **Chart.js** via `cdnjs.cloudflare.com`
- **SheetJS/xlsx** via `cdnjs.cloudflare.com`
- Fontes do sistema (sem Google Fonts)

---

# ESCOPO — O QUE NÃO FAZER

- Sem login ou multi-usuário
- Sem APIs externas
- Sem PWA / service worker
- Sem backend
- Sem ES6+ (sem arrow functions, template literals, const/let, destructuring, spread)
- Sem `fi.click()` para abrir file input
- Sem `<select>` nativo oculto para dropdowns customizados
- Sem heatmap, timeline ou painéis não listados
- Sem KPI "Próxima aplicação" (substituído por "Adubações Atrasadas")
- Sem KPI "Aplicações"
- Sem badge de status da semana (passada/atual/futura)
- Sem barra de progresso nos KPIs de Ruas Podadas / Distribuição de Podas
- Sem Google Fonts (causa erros de CSP em `file://`)

---

# CUIDADOS CRÍTICOS DE IMPLEMENTAÇÃO

Erros recorrentes a evitar desde o início:

1. **Tooltips com `overflow:hidden` no pai** → sempre cortam o conteúdo. Use container com `overflow:visible` e separe o background da barra em elemento filho absoluto
2. **Template literals em ES5** → não existem, causam SyntaxError. Use concatenação de strings
3. **Classes CSS duplicadas** → o segundo bloco vence e sobrescreve o primeiro. Defina cada classe uma única vez
4. **`fi.click()` em iframe/file://** → silenciosamente bloqueado. Use `<label for="...">`
5. **`<select>` nativo oculto sobreposto a botão** → cliques não passam por baixo. Implemente dropdown 100% em JS/HTML
6. **Renders chamando elementos removidos** → causam `Cannot set properties of null`. Sempre cheque com `if(!gel('id')) return;`
7. **Google Fonts em `file://`** → erros de CSP no console. Use fontes do sistema
8. **`localStorage` sem try/catch** → quebra o app no Edge com `file://`. Sempre envolva em try/catch com fallback
