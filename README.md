# 📈 Analisador Fundamentalista B3

> Ferramenta web standalone para análise e ranqueamento de ações da Bolsa Brasileira (B3) com base em indicadores fundamentalistas. Sem API key, sem instalação, sem banco de dados — abre direto no navegador.

---

## 📋 Índice

- [Visão Geral](#visão-geral)
- [Origem do Projeto](#origem-do-projeto)
- [Tecnologias Utilizadas](#tecnologias-utilizadas)
- [Como Usar](#como-usar)
- [Arquitetura do Código](#arquitetura-do-código)
- [Fonte de Dados — Brapi.dev](#fonte-de-dados--brapidev)
- [Metodologia de Score](#metodologia-de-score)
- [Indicadores Fundamentalistas](#indicadores-fundamentalistas)
- [Filtros e Ordenação](#filtros-e-ordenação)
- [Estrutura de Arquivos](#estrutura-de-arquivos)
- [Limitações Conhecidas](#limitações-conhecidas)
- [Histórico de Versões](#histórico-de-versões)
- [Aviso Legal](#aviso-legal)

---

## Visão Geral

O **Analisador Fundamentalista B3** é uma aplicação web de arquivo único (`AnalisadorB3.html`) que busca dados financeiros em tempo real de ações da B3 e calcula automaticamente um **score de qualidade fundamentalista** para cada ativo, ranqueando as melhores oportunidades de investimento de longo prazo.

**Características principais:**

- Arquivo HTML único — abre com dois cliques, sem instalar nada
- Dados em tempo real via [Brapi.dev](https://brapi.dev) (API gratuita, brasileira, sem chave)
- Score automático baseado nos métodos Bazin, Graham e Barsi
- 30 ações pré-configuradas da B3, com possibilidade de adicionar qualquer ticker
- Filtros interativos: DY mínimo, ordenação por múltiplos critérios, perfil de investimento
- Interface responsiva com análise detalhada por ação

---

## Origem do Projeto

Este projeto é a **reescrita e evolução** do projeto original `Furlaneti.Finance.ChangeBase` (C# / .NET), que:

- Lia tickers de um arquivo Excel (`acoes.xlsx`)
- Consultava a API HG Brasil (paga) para dados de preço e dividendos
- Fazia scraping do site Status Invest para indicadores como P/L, ROE, EV/EBITDA
- Salvava os dados em um banco SQL Server via Stored Procedures

**Problemas do projeto original resolvidos nesta versão:**

| Problema Original | Solução Atual |
|---|---|
| Dependência de API HG Brasil (paga) | Brapi.dev — 100% gratuita |
| Banco de dados SQL Server necessário | Sem banco — tudo em memória no browser |
| Scraping do Status Invest (frágil) | API estruturada da Brapi |
| Só rodava em Windows com .NET | Roda em qualquer navegador moderno |
| Sem interface visual | Dashboard completo com score e filtros |

---

## Tecnologias Utilizadas

| Tecnologia | Uso |
|---|---|
| **HTML5** | Estrutura da página |
| **CSS3** | Estilização, animações, layout responsivo com Grid e Flexbox |
| **JavaScript (ES2020+)** | Lógica de negócio, fetch assíncrono, manipulação do DOM |
| **Brapi.dev API** | Fonte de dados de ações B3 (REST, JSON, gratuita) |
| **Google Fonts** | Tipografia: IBM Plex Mono + DM Sans |
| **SVG** | Gráficos de score em formato de anel (ring chart) |

**Sem dependências externas de biblioteca** — o arquivo é 100% vanilla HTML/CSS/JS.

---

## Como Usar

### Pré-requisitos

- Qualquer navegador moderno: Chrome, Edge, Firefox ou Safari
- Conexão com a internet (para buscar os dados da API)

### Passo a Passo

**1. Abrir o arquivo**

Dê dois cliques no arquivo `AnalisadorB3.html`. Ele abrirá diretamente no seu navegador padrão.

**2. Configurar a lista de ações**

O sistema já vem com 30 ações pré-selecionadas. Você pode:

- **Remover** uma ação clicando no `×` ao lado do ticker
- **Adicionar** uma ação digitando o código (ex: `ITSA4`) no campo de texto e pressionando `Enter` ou clicando em `+ Add`

> **Atenção:** Use os códigos exatos da B3 (ex: `PETR4`, `VALE3`, `BBAS3`). FIIs como `MXRF11` podem não retornar todos os dados fundamentalistas.

**3. Iniciar a análise**

Clique no botão **🚀 Iniciar Análise**. A barra de progresso mostrará o carregamento em tempo real.

A análise busca as ações em lotes de 10, com um intervalo de 600ms entre cada lote para respeitar os limites da API gratuita. Para 30 ações, o tempo total é aproximadamente **20 a 40 segundos**.

**4. Interpretar os resultados**

As ações aparecem ordenadas pelo score (padrão). Cada card mostra:

- **Preço atual** e variação do dia
- **Score** de 0 a 100 com classificação visual
- **6 indicadores** coloridos (verde/amarelo/vermelho)
- Clique no card para expandir e ver a **análise detalhada** com os critérios aplicados

**5. Usar os filtros**

Após o carregamento, os filtros ficam disponíveis:

- Ordenar por: Score, DY, ROE, P/L, P/VP
- Filtrar por perfil: Todas / Score ≥65 / DY ≥5% / P/L ≤15
- Slider de DY mínimo

---

## Arquitetura do Código

O arquivo `AnalisadorB3.html` está organizado em três seções principais dentro de uma única página:

```
AnalisadorB3.html
├── <head>           → Metadados, fontes Google, variáveis CSS (:root)
├── <style>          → Todo o CSS da aplicação (~250 linhas)
├── <body>           → Estrutura HTML estática (header, painéis, containers)
└── <script>         → Toda a lógica JavaScript (~350 linhas)
    ├── Estado global
    ├── calcScore()       → Motor de pontuação fundamentalista
    ├── gradeOf()         → Classificação por faixa de score
    ├── Formatadores      → fmt(), fmtP(), fmtC()
    ├── pillColor()       → Cor semântica dos indicadores
    ├── HTML builders     → pillHTML(), ringHTML(), cardHTML()
    ├── fetchBatch()      → Chamada à API Brapi.dev
    ├── parseStock()      → Normalização dos dados recebidos
    ├── runAnalysis()     → Orquestrador principal (async)
    ├── renderStats()     → Atualiza painel de estatísticas
    ├── renderCards()     → Renderiza grid de cards com filtros
    └── UI helpers        → addTicker(), removeTicker(), renderTags()
```

### Fluxo de Execução

```
Usuário clica "Iniciar Análise"
        │
        ▼
runAnalysis()
        │
        ├─→ Para cada lote de 10 tickers:
        │       │
        │       ├─→ fetchBatch(tickers)
        │       │       └─→ GET brapi.dev/api/quote/TICKER1,TICKER2,...
        │       │
        │       ├─→ parseStock(resultado) → objeto normalizado
        │       │
        │       ├─→ stocksData.push(parsed)
        │       │
        │       └─→ renderCards() → atualiza DOM em tempo real
        │
        └─→ Fim: renderStats() + renderCards() com dados completos
```

### Estado Global

```javascript
let stocksData   = [];   // Array de objetos de ações carregadas
let expandedCard = null; // Symbol da ação com card expandido
let errors       = [];   // Tickers que falharam na busca
let tickerList   = [...]; // Lista atual de tickers configurados
```

---

## Fonte de Dados — Brapi.dev

### Por que Brapi.dev?

| Critério | Brapi.dev | Yahoo Finance | HG Brasil (original) |
|---|---|---|---|
| Custo | Gratuito | Gratuito (instável) | Pago |
| API Key | Não obrigatória | Não | Sim |
| CORS (browser) | ✅ Liberado | ❌ Bloqueado | ❌ Bloqueado |
| Dados B3 nativos | ✅ Sim | Parcial (.SA) | ✅ Sim |
| Fundamentos | ✅ Sim | Parcial | Básico |

### Endpoint Utilizado

```
GET https://brapi.dev/api/quote/{TICKERS}?fundamental=true&dividends=true
```

**Parâmetros:**

| Parâmetro | Valor | Descrição |
|---|---|---|
| `{TICKERS}` | `PETR4,VALE3,...` | Até 10 tickers separados por vírgula |
| `fundamental` | `true` | Inclui dados fundamentalistas (P/L, P/VP, ROE, etc.) |
| `dividends` | `true` | Inclui histórico e yield de dividendos |

### Campos Recebidos e Mapeamento

```javascript
// Resposta da API → Objeto interno parseStock()

r.symbol                              → stock.symbol       // Código do ativo (ex: PETR4)
r.shortName / r.longName             → stock.name         // Nome da empresa
r.regularMarketPrice                 → stock.price        // Preço atual
r.regularMarketChangePercent         → stock.change       // Variação % do dia
r.marketCap                          → stock.marketCap    // Capitalização de mercado
r.dividendYield                      → stock.dy           // DY 12 meses (em %)
r.priceEarnings                      → stock.pl           // Preço/Lucro
r.priceToBook                        → stock.pvp          // Preço/Valor Patrimonial
r.summaryProfile.returnOnEquity * 100 → stock.roe         // Retorno sobre Patrimônio (%)
r.earningsPerShare                   → stock.lpa          // Lucro por Ação
r.bookValuePerShare                  → stock.vpa          // Valor Patrimonial por Ação
r.profitMargin * 100                 → stock.margemLiquida // Margem Líquida (%)
r.summaryProfile.sector              → stock.sector       // Setor da empresa
r.logourl                            → stock.logo         // URL do logo
```

---

## Metodologia de Score

O score é calculado pela função `calcScore(stock)` e varia de **0 a 100 pontos**, combinando três metodologias clássicas de análise fundamentalista:

### Distribuição dos Pontos

| Indicador | Peso | Metodologia de Referência |
|---|---|---|
| Dividend Yield | 30 pts | Décio Barros Bazin |
| P/L (Preço/Lucro) | 20 pts | Benjamin Graham |
| ROE | 20 pts | Warren Buffett |
| P/VP | 15 pts | Benjamin Graham |
| Margem Líquida | 10 pts | Análise de eficiência |
| LPA positivo | +5 pts (bônus) | Empresa lucrativa |
| **Total máximo** | **100 pts** | |

### Critérios Detalhados por Indicador

#### Dividend Yield — 30 pts (Método Bazin)

Luiz Barsi Filho e Décio Bazin defendem que uma ação de qualidade deve pagar no mínimo 6% ao ano em dividendos. Quanto maior e mais consistente o DY, melhor.

```
DY ≥ 8%   → 30 pts  (excelente)
DY ≥ 6%   → 24 pts  (muito bom)
DY ≥ 4%   → 15 pts  (moderado)
DY ≥ 2%   →  7 pts  (baixo)
DY < 2%   →  0 pts  (muito baixo)
```

#### P/L — 20 pts (Benjamin Graham)

O P/L indica quantos anos de lucro atual o investidor paga pelo ativo. Graham considerava P/L abaixo de 15 como justo, e abaixo de 10 como barato.

```
P/L ≤ 8    → 20 pts  (muito barato)
P/L ≤ 12   → 16 pts  (barato)
P/L ≤ 15   → 10 pts  (justo)
P/L ≤ 25   →  4 pts  (caro)
P/L > 25   →  0 pts  (muito caro)
P/L < 0    →  0 pts  (empresa em prejuízo)
```

#### ROE — 20 pts (Warren Buffett)

O ROE mede a eficiência com que a empresa usa o capital dos acionistas para gerar lucro. Buffett busca empresas com ROE consistentemente acima de 15%.

```
ROE ≥ 25%  → 20 pts  (excepcional)
ROE ≥ 15%  → 15 pts  (forte)
ROE ≥ 10%  →  8 pts  (aceitável)
ROE > 0%   →  3 pts  (fraco)
ROE ≤ 0%   →  0 pts  (retorno negativo)
```

#### P/VP — 15 pts (Margem de Segurança Graham)

O P/VP compara o preço de mercado com o valor patrimonial contábil. Graham recomendava P/VP abaixo de 1,5 como indicativo de margem de segurança.

```
P/VP ≤ 1.0  → 15 pts  (muito atrativo)
P/VP ≤ 1.5  → 12 pts  (atrativo)
P/VP ≤ 2.5  →  6 pts  (razoável)
P/VP > 2.5  →  0 pts  (caro)
```

#### Margem Líquida — 10 pts

A margem líquida mede o percentual do faturamento que se converte em lucro. Empresas com margens altas têm poder de precificação e eficiência operacional.

```
Margem ≥ 20%  → 10 pts  (excelente)
Margem ≥ 10%  →  7 pts  (boa)
Margem ≥  5%  →  4 pts  (ok)
Margem >  0%  →  1 pt   (baixa)
Margem ≤  0%  →  0 pts  (prejuízo)
```

#### Bônus LPA Positivo — +5 pts

Confirma que a empresa está gerando lucro por ação, sinalizando saúde financeira básica.

### Classificação Final

| Faixa de Score | Classificação | Cor |
|---|---|---|
| 80 – 100 | EXCELENTE | 🟢 Verde vivo |
| 65 – 79  | BOM | 🟢 Verde claro |
| 45 – 64  | REGULAR | 🟡 Amarelo |
| 25 – 44  | FRACO | 🟠 Laranja |
| 0 – 24   | EVITAR | 🔴 Vermelho |

---

## Indicadores Fundamentalistas

Descrição de cada indicador exibido nos cards:

| Indicador | Sigla | O que mede | Bom quando |
|---|---|---|---|
| Dividend Yield | DY | % do preço pago em dividendos nos últimos 12 meses | ≥ 6% |
| Preço/Lucro | P/L | Quanto o mercado paga por R$1 de lucro | Entre 5 e 15 |
| Preço/Valor Patrimonial | P/VP | Preço vs. patrimônio líquido contábil | < 1.5 |
| Retorno sobre Patrimônio | ROE | Eficiência no uso do capital dos acionistas | ≥ 15% |
| Lucro por Ação | LPA | Lucro líquido dividido pelo número de ações | > 0 |
| Margem Líquida | Margem | % da receita que vira lucro | ≥ 10% |
| Valor Patrimonial por Ação | VPA | Patrimônio líquido ÷ número de ações | Referência |

---

## Filtros e Ordenação

### Ordenação disponível

| Opção | Critério |
|---|---|
| 🏆 Score | Score fundamentalista (maior primeiro) |
| 💰 DY maior | Dividend Yield decrescente |
| 📊 ROE maior | Return on Equity decrescente |
| 💲 P/L menor | Preço/Lucro crescente (mais barato primeiro) |
| 📚 P/VP menor | Preço/VP crescente |

### Filtros de perfil

| Filtro | Critério aplicado |
|---|---|
| Todas | Sem filtro |
| ✅ Score ≥ 65 | Apenas ações com score "BOM" ou "EXCELENTE" |
| 💰 DY ≥ 5% | Ações com yield acima de 5% |
| 💲 P/L ≤ 15 | Ações baratas pelo critério de Graham |

### Slider DY mínimo

Permite definir um patamar mínimo de Dividend Yield de 0% a 12%, filtrando ações que não atingem o yield desejado.

---

## Estrutura de Arquivos

```
projeto/
├── AnalisadorB3.html        ← Aplicação completa (único arquivo necessário)
├── AnalisadorB3_Brapi.jsx   ← Versão React (para uso no Claude.ai / Vite)
├── README.md                ← Esta documentação
└── Archive/
    └── acoes.xlsx           ← Lista original de tickers (projeto C# legado)
```

### Versão React vs. HTML Standalone

| Característica | AnalisadorB3.html | AnalisadorB3_Brapi.jsx |
|---|---|---|
| Instalação | Nenhuma | Node.js + npm |
| Como abrir | Dois cliques | `npm run dev` |
| Portabilidade | ✅ Máxima | Requer build |
| Customização | Editar HTML direto | Componentes React |
| Uso recomendado | Uso pessoal / compartilhar | Integrar em projeto web |

---

## Limitações Conhecidas

**Dados não disponíveis via Brapi gratuito:**

- Dívida Líquida / EBITDA — requer plano pago ou scraping
- CAGR de lucro 5 anos — não exposto na API free
- EV/EBITDA — não disponível
- Histórico de dividendos por ano — dados agregados apenas

**Limitações de rate limit:**

- A API gratuita da Brapi permite consultas sem autenticação, mas tem limites implícitos
- O código usa batches de 10 ações com 600ms de intervalo para evitar bloqueios
- Para listas muito longas (> 50 ações), considere criar uma conta gratuita na Brapi e adicionar o token

**FIIs (Fundos de Investimento Imobiliário):**

- Tickers como `MXRF11`, `HGLG11` podem ser buscados mas os indicadores fundamentalistas (P/L, ROE, Margem) não se aplicam da mesma forma
- Para FIIs, os indicadores mais relevantes são DY e P/VP

**Atualização dos dados:**

- Preços: atualizados em tempo real (durante horário de pregão)
- Dados fundamentais (P/L, ROE, etc.): atualizados trimestralmente conforme balanços publicados

---

## Histórico de Versões

### v3.0 — AnalisadorB3.html (atual)
- Reescrita completa em HTML/CSS/JS vanilla
- Funciona sem instalação, abre direto no navegador
- Fonte de dados: Brapi.dev (gratuita, sem CORS)

### v2.0 — AnalisadorB3_Brapi.jsx
- Versão React com hooks
- Substituiu Yahoo Finance (bloqueado por CORS) por Brapi.dev

### v1.0 — AnalisadorFundamentalistaB3.jsx
- Primeira versão React
- Usava Yahoo Finance via proxy CORS (instável no Claude.ai)

### v0.1 — Furlaneti.Finance.ChangeBase (C# original)
- Projeto legado em .NET 6
- API HG Brasil (paga) + scraping Status Invest + SQL Server

---

## Aviso Legal

> **Este projeto é de uso educacional e informativo.**
>
> Os dados exibidos são obtidos de fontes públicas e podem conter imprecisões ou atrasos. O score fundamentalista calculado automaticamente **não constitui recomendação de investimento**.
>
> Antes de tomar qualquer decisão de investimento, consulte um assessor de investimentos certificado (AAI) habilitado pela CVM e leia o prospecto completo de cada ativo.
>
> O autor não se responsabiliza por perdas financeiras decorrentes do uso desta ferramenta.

---

*Desenvolvido com base no projeto original Furlaneti.Finance.ChangeBase — reescrito e modernizado.*
