# Pico Cívico — Dossier Técnico de Handoff

**Data do levantamento:** 2026-09-03
**Âmbito:** dois repositórios, `rodrigoquadros-commits/pico-civico` (branch `claude/pico-civico-platform-s1emb0`, commit `e674919`) e `rodrigoquadros-commits/lajes-acervo` (branch `claude/assembleia-voto-nominal`, commit `342b161`).
**Objetivo deste documento:** permitir que outra equipa desenhe um módulo de Social Intelligence / Trends / Influence Mapping / Predictive Analytics sem destruir nem duplicar o que existe.

## Aviso metodológico

Este dossier descreve **apenas o que está implementado e verificado no código e nos dados**. Onde não foi possível determinar, está escrito `DESCONHECIDO`. Onde algo é intenção documentada mas não código, está marcado `PLANEADO`.

**Leitura obrigatória antes de desenhar:** o sistema atual **não é** uma plataforma de social listening. É um *pipeline de forense documental* — descobre PDF em sites municipais, faz OCR, extrai estrutura por expressões regulares, e exporta JSON para uma página HTML estática. Não existe backend, API, autenticação, alojamento aplicacional, fila de processamento, índice de texto, base vetorial, nem qualquer utilização de IA ou LLM. Praticamente todo o substrato de que um módulo de Social Intelligence precisa **ainda não existe**. O que existe é limpo, normalizado e com proveniência — e não deve ser reconstruído.

### Legenda de estado

| Marca | Significado |
|---|---|
| `IMPLEMENTADO` | Existe em código, corre, e produziu dados verificados |
| `PARCIAL` | Existe em código mas cobre só parte do âmbito, ou depende de passo manual |
| `PLANEADO` | Documentado em docstring/README como intenção; sem código funcional |
| `INEXISTENTE` | Não existe de nenhuma forma |
| `DESCONHECIDO` | Não foi possível determinar a partir do projeto |

---

# 1. Objetivo e âmbito atual

## 1.1 Objetivo

Do `README.md` do `pico-civico`, verbatim: *"Plataforma de inteligência cívica e fiscalização da ilha do Pico (Açores). Um mapa interativo, concelho a concelho e freguesia a freguesia, que reúne indicadores sociais, económicos e políticos e liga quem governa · o que investe · como estão as contas · o que muda."*

O eixo operacional é a **fiscalização municipal com lastro documental**: cada valor apresentado tem ano, fonte e marca de fiabilidade (`V` verificado / `~` imprensa / `?` por apurar).

## 1.2 Casos de utilização implementados

1. **Perfil de concelho** — indicadores, governação, contratação, finanças, num único painel.
2. **Drill-down geográfico** — mapa SVG da ilha → concelho → freguesia.
3. **Registo de voto nominal da Assembleia Municipal** — presenças e sentido de voto por membro, extraídos de atas em PDF por OCR. É a funcionalidade diferenciadora: este grão não existe estruturado a nível municipal em Portugal.
4. **Forense de contratação pública** — série anual, concentração (HHI), fornecedores em ascensão, clustering junto a limiares legais, rede cross-concelho, tabela pesquisável.
5. **Peso público documentado** — índice por titular de cargo (assiduidade, voto nominal, dissidência de bancada, pivotalidade, longevidade) e por empresa (quota, persistência, amplitude).
6. **Apoio eleitoral por freguesia** — votos por lista, afluência, brancos e nulos.

## 1.3 Tipos de utilizador

**INEXISTENTE como conceito no sistema.** Não há autenticação, não há tabela de utilizadores, não há papéis, não há sessões. A página é pública e anónima para todos os visitantes. O `README` identifica a audiência pretendida ("a comunidade do Pico") mas isso não está modelado em código.

## 1.4 Estado das funcionalidades

| Funcionalidade | Estado | Nota |
|---|---|---|
| Mapa SVG por freguesia com drill-down | `IMPLEMENTADO` | 17 freguesias, projeção equirretangular calculada em JS |
| Perfil de concelho por separadores | `IMPLEMENTADO` | 7 separadores; 3 dos 3 concelhos têm Contratos e Finanças |
| Indicadores por área (12 categorias) | `PARCIAL` | Só Lajes do Pico (38 indicadores). Madalena e São Roque caem num fallback de 5 cartões escritos à mão em `CONC[].ind` |
| Assembleia — sessões e deliberações | `IMPLEMENTADO` | 8 mandatos, 1997–2026, só Lajes do Pico |
| Assembleia — voto nominal por membro | `PARCIAL` | **Desde 2015-08-06.** As atas anteriores registam só o resultado (unanimidade/maioria) sem nomear quem votou — limitação da fonte, não do sistema. A prática instala-se a meio do mandato 2013-2017 |
| Contratação pública 2021–2026 | `IMPLEMENTADO` | 3 concelhos, 735 contratos |
| Finanças municipais | `PARCIAL` | Só 2017–2019 (limite dos dados abertos da DGAL) |
| Influência — peso na decisão | `IMPLEMENTADO` | 6 mandatos com dados, só Lajes do Pico |
| Influência — peso económico | `IMPLEMENTADO` | 25 fornecedores, 3 concelhos |
| Apoio eleitoral por freguesia | `PARCIAL` | 2017 e 2021 apenas. 2025 bloqueado (ver §6) |
| Governação (presidente, câmara, assembleia) | `PARCIAL` | Escrito à mão em `CONC` no JS, não vem de base de dados |
| Decisões → Resultados | `PLANEADO` | Os dados em `CONC[].dr` são **demonstrativos e assinalados como tal na página**. Não são reais |
| Pesquisa de texto integral | `PLANEADO` | `lajes_acervo/index.py` é só uma docstring |
| Notícias / imprensa / redes sociais | `INEXISTENTE` | Nenhuma recolha, nenhuma tabela, nenhum campo |
| Taxonomia temática | `INEXISTENTE` | Ver §9 |
| Atas da Câmara Municipal | `PLANEADO` | `config.ATAS_CAMARA` tem os 29 IDs de página mapeados; nunca foi corrido |
| Documentos financeiros (orçamentos, relatórios) | `PLANEADO` | `discover.discover_financas()` está implementado; nunca foi corrido |

## 1.5 Unidade geográfica

**Unidade inicial:** o concelho, identificado pelo **código INE de 4 dígitos**. Três concelhos da ilha do Pico:

| Código INE | Concelho | NIF do município | geocod INE (API pindica) | Chave DGAI |
|---|---|---|---|---|
| `4601` | Lajes do Pico | 512074143 | 2004601 | `LOCAL-460100` |
| `4602` | Madalena | 512070946 | 2004602 | `LOCAL-460200` |
| `4603` | São Roque do Pico | 512074771 | 2004603 | `LOCAL-460300` |

Abaixo do concelho: 17 freguesias (6 + 6 + 5). No `pico-civico` estão em `GEO` (GeoJSON) e `FMETA` (metadados escritos à mão). Na DGAI têm chaves `LOCAL-DDMMFF`.

O acervo documental está implementado **apenas para 4601** (Lajes do Pico), porque a descoberta depende de IDs de página específicos do site `cm-lajesdopico.pt`, mapeados em `lajes_acervo/config.py`.

## 1.6 Expansão prevista

O `catalog.py` está desenhado para escalar: *"Curar aqui UMA vez (códigos INE varcd, geografia, período) serve os 308 municípios."* Na prática:

- **Indicadores e Censos** — escalam trocando o código do município (`MUNICIPIOS` no catálogo). Conectores genéricos.
- **Contratação** — escala trocando o NIF do adjudicante.
- **Eleições** — escala trocando a chave `LOCAL-DDMMFF`. O conector já descobre freguesias por concelho.
- **Atas** — **NÃO escala**. Cada câmara tem um site diferente. `discover.py` está escrito contra a paginação e os dois formatos de listagem do `cm-lajesdopico.pt`. Novo município = novo descobridor.
- **Extração da Assembleia** — o `assembleia.py` depende da fraseologia ritual das atas das Lajes ("Presentes estavam…", "eleitos pelo Grupo Municipal do…"). Outro município exigirá recalibrar as expressões regulares, embora a arquitetura (roster → presenças casadas → votos) seja transferível.

---

# 2. Arquitetura técnica

## 2.1 O que NÃO existe

Enunciado primeiro, porque é a informação mais importante desta secção:

| Componente convencional | Estado |
|---|---|
| Backend / servidor de aplicação | `INEXISTENTE` |
| API HTTP (própria) | `INEXISTENTE` |
| Autenticação / autorização | `INEXISTENTE` |
| Alojamento cloud / PaaS | `INEXISTENTE` (ver 2.4) |
| Fila de mensagens / processamento assíncrono | `INEXISTENTE` |
| Tarefas agendadas (cron, scheduler) | `INEXISTENTE` — toda a ingestão é invocada à mão |
| Índice de pesquisa (FTS5, Elastic, Meili) | `PLANEADO` |
| Base de dados vetorial | `INEXISTENTE` |
| Embeddings | `INEXISTENTE` (existe a coluna `bloco.embedding BLOB`, tabela vazia) |
| Utilização de IA / LLM | `INEXISTENTE` — zero chamadas, zero SDK, zero prompts |
| Logging estruturado / APM / monitorização | `INEXISTENTE` (existe a tabela `execucao`, ver 2.7) |
| Frontend framework (React, Vue, Svelte) | `INEXISTENTE` |
| Build step / bundler / transpilação | `INEXISTENTE` — deliberado |
| Biblioteca de gráficos | `INEXISTENTE` — SVG e HTML escritos à mão |
| Docker / containers | `INEXISTENTE` |
| CI / CD | `INEXISTENTE` — não há `.github/workflows` |
| ORM (Prisma, SQLAlchemy, Django) | `INEXISTENTE` — SQL escrito à mão em `db.py` |

## 2.2 Componente → Tecnologia → Função → Estado

```text
FRONTEND
  Página única        → HTML5 + CSS3 + JavaScript ES5 (sem build)
                      → Toda a interface; dados embutidos inline
                      → IMPLEMENTADO  (pico-civico/index.html, 677 247 bytes, 1239 linhas)

  Tipografia          → Google Fonts (Space Grotesk, IBM Plex Mono)
                      → Única dependência externa em runtime
                      → IMPLEMENTADO

  Cartografia         → SVG gerado em JS a partir de GeoJSON inline
                      → Mapa da ilha, drill-down, centróides, rótulos
                      → IMPLEMENTADO  (funções eachCoord/proj/ringPath/geomPath/centroid)

  Gráficos            → SVG e <div> com width em % escritos à mão
                      → sparklines, barras de bancada, barras empilhadas, cobertura
                      → IMPLEMENTADO  (spark, seatBar, seatBarP, partesBar, barras)

  Pesquisa            → Array.filter + String.indexOf no cliente
                      → Filtro da tabela de contratos (input #tsearch)
                      → IMPLEMENTADO  (renderTabela)

INGESTÃO / PROCESSAMENTO  (pacote lajes_acervo, Python 3.9+)
  HTTP                → urllib.request (stdlib) + retry/backoff próprio
                      → GET educado com User-Agent identificável
                      → IMPLEMENTADO  (http.py, 41 linhas)

  Descoberta          → html.parser.HTMLParser (stdlib)
                      → Encontra PDF em páginas de menu paginadas
                      → IMPLEMENTADO  (discover.py, 154 linhas)

  Metadados           → Expressões regulares
                      → Data/tipo/número da reunião a partir do rótulo da listagem
                      → IMPLEMENTADO  (metadata.py, 128 linhas)

  Descarga            → urllib + hashlib.sha256 + ThreadPoolExecutor(4)
                      → GET do PDF, sha256, sonda de páginas e camada de texto
                      → IMPLEMENTADO  (download.py, 95 linhas)

  OCR                 → tesseract 5.3.4 (lang=por, --psm 3, 300 dpi)
                        + poppler-utils (pdftoppm, pdftotext, pdfinfo)
                      → Texto por página; pdftotext quando há camada de texto
                      → IMPLEMENTADO  (ocr.py, 121 linhas)

  Extração estrutural → Expressões regulares + difflib.SequenceMatcher
                      → Presenças, deliberações, voto nominal, composição do órgão
                      → IMPLEMENTADO  (assembleia.py, 869 linhas — o módulo central)

  Métricas            → Python puro (collections)
                      → Índices de peso público por titular e por empresa
                      → IMPLEMENTADO  (influencia.py, 315 linhas)

  Conectores          → urllib + json
                      → INE pindica, geoapi.pt, DGAI eleições
                      → IMPLEMENTADO  (connectors/: ine.py, geoapi.py, eleicoes.py, catalog.py)

BASE DE DADOS
  Armazenamento       → SQLite 3 (ficheiro único)
                      → 12 tabelas; SQL escrito à mão, sem ORM
                      → IMPLEMENTADO  (db.py, 487 linhas; data/acervo.sqlite)
                      → NOTA: gitignored. NÃO está publicado nem servido.

  PDF originais       → Sistema de ficheiros local
                      → Fonte da verdade; data/pdf/{orgao}/{ano}/{id}-{nome}.pdf
                      → IMPLEMENTADO  (110 ficheiros)

  Datasets derivados  → JSON em build/datasets/ (versionados em git)
                      → Ponte entre o pipeline e a plataforma
                      → IMPLEMENTADO  (8 ficheiros, ~1,04 MB)

ENTREGA
  Injeção             → build/inject.py (splice por índice, ASCII-safe)
                      → Substitui blocos <script>var NOME=…</script> no index.html
                      → IMPLEMENTADO  (108 linhas)

  Publicação          → git push + (nesta sessão) um Artifact privado em claude.ai
                      → GitHub Pages está PLANEADO, não configurado
                      → PARCIAL
```

## 2.3 Linguagens e versões

| Item | Valor |
|---|---|
| Python | 3.9+ declarado (`from __future__ import annotations` em todos os módulos); corrido em 3.11.x |
| JavaScript | ES5 deliberado — `var`, `function`, sem arrow functions, sem template literals, sem `let/const` |
| SQL | SQLite dialect, escrito à mão |
| Dependências Python externas | **ZERO.** Apenas stdlib: `argparse collections concurrent dataclasses datetime difflib hashlib html json os re shutil sqlite3 subprocess sys tempfile time unicodedata urllib` |
| Dependências de sistema | `tesseract` (+ `tesseract-lang` por), `poppler-utils` |
| Testes | `pytest` (dev only) — 49 testes, 4 ficheiros |
| `requirements.txt` | Contém apenas comentários. Lista `ocrmypdf`, `pymupdf`, `sentence-transformers` como opcionais futuros — **nenhum é importado** |

## 2.4 Alojamento

- `pico-civico/index.html` é **autónomo**: abre com `file://`, sem servidor.
- Nenhum domínio, nenhum CDN, nenhum bucket, nenhuma função serverless.
- Único artefacto publicado nesta sessão: um Artifact privado em `claude.ai/code/artifact/5acdf098-994f-4b0e-a02e-c3b338ad1ea1`. Não é infraestrutura de produção.
- `DESCONHECIDO`: se existe intenção de domínio próprio ou conta cloud.

## 2.5 APIs

Não existe API **fornecida**. As APIs **consumidas** estão em §6.

## 2.6 Fluxo real

```text
FONTES EXTERNAS
  cm-lajesdopico.pt (HTML paginado + PDF)
  www.ine.pt/ine/json_indicador/pindica.jsp (JSON)
  json.geoapi.pt (JSON)
  www.eleicoes.mai.gov.pt (ficheiros estáticos JSON da SPA)
  dados.gov.pt → AIQGP da DGAL (descarga MANUAL)
  base.gov.pt (POST; recolha já feita, endpoint não reconfirmado)
        │
        ▼
INGESTÃO  ── invocada À MÃO, sem agendador
  python3 -m lajes_acervo atas        → discover → download → ocr
  python3 -m lajes_acervo eleicoes    → connectors/eleicoes
  python3 -m lajes_acervo indicadores → connectors/ine + geoapi
  build/fetch_contratos.py            → contratos_raw.json
        │
        ▼
PROCESSAMENTO
  ocr.py           → texto por página (tesseract/pdftotext)
  assembleia.py    → roster → presenças casadas → deliberações → votos
  influencia.py    → métricas por titular e por empresa
  build/forensics_all.py → forense de contratação
        │
        ▼
BASE DE DADOS  (SQLite local, NÃO servida, NÃO versionada)
  data/acervo.sqlite — 12 tabelas
        │
        ▼
EXPORTAÇÃO  → JSON em build/datasets/  (versionado em git)
  --export nos comandos assembleia / influencia / eleicoes / indicadores
        │
        ▼
INJEÇÃO  → build/inject.py faz splice ASCII-safe no HTML
        │
        ▼
FRONTEND  → pico-civico/index.html  (dados INLINE, sem fetch)
```

**Ponto crítico de arquitetura para quem for integrar:** a base de dados **não é o backend da plataforma**. É um artefacto de build local. A plataforma consome cópias JSON congeladas, embutidas no HTML. Não há caminho de leitura em runtime entre o frontend e o SQLite.

## 2.7 Logging e monitorização

Existe uma única tabela, `execucao`, com uma linha por etapa corrida:

```sql
CREATE TABLE execucao (
  id INTEGER PRIMARY KEY, etapa TEXT NOT NULL,
  iniciado_em TEXT, terminado_em TEXT, estado TEXT,
  itens_vistos INTEGER DEFAULT 0, itens_novos INTEGER DEFAULT 0,
  itens_erro INTEGER DEFAULT 0, nota TEXT
);
```

Conteúdo atual (6 linhas): `descarregar` (110/110, 0 erros), `ocr` (110 docs / 1526 páginas), `assembleia` (×3, a última com 110), `eleicoes` (120, nota "ano 2021"). `iniciado_em` e `terminado_em` são gravados com o mesmo timestamp — **não medem duração**. Não há níveis de log, nem stack traces persistidos, nem alertas. Os erros são impressos em `stderr` e perdidos.

---

# 3. Estrutura da base de dados

Motor: **SQLite 3**, ficheiro `lajes-acervo/data/acervo.sqlite`. Definida como uma constante `SCHEMA` (string SQL) em `lajes_acervo/db.py`, aplicada com `executescript()`. **Não há ORM, não há migrações, não há Prisma.** `db.init()` corre `CREATE TABLE IF NOT EXISTS`, pelo que uma alteração de esquema a uma tabela existente **não é aplicada** — foi por isso criada `db.refazer_assembleia()`, que faz `DROP` das cinco tabelas do órgão e recria (são 100% derivadas de `pagina`).

`PRAGMA foreign_keys` **não é ativado** em `db.connect()`. As FK estão declaradas mas **não são impostas** em runtime.

## 3.1 Sumário: 12 tabelas

| Tabela | Registos | Finalidade |
|---|---|---|
| `documento` | 110 | Catálogo de PDF descobertos e o seu ciclo de vida |
| `pagina` | 1 526 | Texto por página (OCR ou camada de texto) |
| `bloco` | 0 | `PLANEADO` — blocos para indexação/embeddings |
| `indicador` | 0 | Indicadores estatísticos por município (**vazia**; os dados vivem no JSON) |
| `membro` | 162 | Membro de um órgão **num mandato** |
| `sessao` | 109 | Sessão de um órgão |
| `presenca` | 970 | Presença/ausência de um membro numa sessão |
| `deliberacao` | 723 | Deliberação tomada numa sessão |
| `voto` | 5 781 | Voto por membro ou por bancada numa deliberação |
| `eleicao_territorio` | 120 | Totais eleitorais de um território numa eleição |
| `eleicao_lista` | 312 | Resultado de cada lista nesse território |
| `execucao` | 6 | Registo de corridas do pipeline |

## 3.2 `documento` — catálogo documental

**PK** `id`. **UNIQUE** `url_origem` (idempotência da descoberta). **Índices:** `ix_doc_orgao_ano (orgao, ano)`, `ix_doc_estado (estado)`.

| Campo | Tipo | Notas |
|---|---|---|
| `orgao` | TEXT NOT NULL | `assembleia` \| `camara` \| `financas` |
| `seccao` | TEXT | ex.: `atas-2025` |
| `ano` | INTEGER | |
| `titulo_original` | TEXT NOT NULL | **Rótulo da listagem — a única fonte semântica fiável.** Os nomes de ficheiro (`154-0.pdf`) não têm significado |
| `tipo_documento` | TEXT | `ata` \| `relatorio_contas` \| `plano_orcamento` \| … |
| `numero_reuniao` | INTEGER | |
| `tipo_reuniao` | TEXT | `ordinaria` \| `extraordinaria` \| `publica` |
| `data_reuniao` | TEXT | ISO `AAAA-MM-DD`, ou NULL quando não extraível com confiança |
| `url_origem` | TEXT NOT NULL | **UNIQUE** — URL do PDF |
| `nome_ficheiro`, `caminho_armazenamento` | TEXT | |
| `sha256` | TEXT | Do conteúdo descarregado |
| `bytes`, `n_paginas` | INTEGER | |
| `tem_camada_texto` | INTEGER | 1/0/NULL — decide OCR vs `pdftotext`. Atual: 3 com texto, 107 sem |
| `needs_review` | INTEGER DEFAULT 0 | Marcado quando os metadados não foram extraídos com confiança |
| `estado` | TEXT NOT NULL DEFAULT 'descoberto' | `descoberto` → `descarregado` → `ocr_feito`; ou `link_invalido` |
| `descoberto_em`, `atualizado_em` | TEXT | Timestamps ISO |

Distribuição atual: 110 documentos, todos `orgao='assembleia'`, `estado='ocr_feito'`, anos 1998–2026.

## 3.3 `pagina` — texto extraído

**PK** `id`. **FK** `documento_id → documento.id ON DELETE CASCADE`. **UNIQUE** `(documento_id, numero_pagina)`.

| Campo | Tipo | Notas |
|---|---|---|
| `numero_pagina` | INTEGER NOT NULL | 1-indexado |
| `texto` | TEXT | Texto integral da página |
| `confianca_ocr` | REAL | **Sempre NULL** — o tesseract é invocado sem `--tsv`, a confiança não é capturada |
| `motor_ocr` | TEXT | `tesseract-por-300dpi` ou `pdftotext` |

**1 526 páginas, 2 958 453 caracteres.** É o único corpus de texto do sistema. **Não está indexado.**

## 3.4 `bloco` — `PLANEADO`, vazia

**PK** `id`. **FK** `documento_id → documento.id ON DELETE CASCADE`.
Campos: `pagina_inicio`, `pagina_fim`, `texto NOT NULL`, `n_tokens`, `embedding BLOB`.
A coluna `embedding` existe mas **nunca foi escrita**. `lajes_acervo/index.py` (7 linhas) é apenas uma docstring que descreve a intenção: blocos de 700–1000 tokens com 15% de sobreposição, FTS5 português primeiro, embeddings depois, recuperação híbrida com proveniência.

## 3.5 `indicador` — vazia

**PK** `id`. **UNIQUE** `(municipio, lab, ano)`. **Índice** `ix_ind_muni (municipio, categoria)`.
Campos: `municipio NOT NULL`, `categoria NOT NULL`, `lab NOT NULL`, `valor REAL`, `unidade`, `ano`, `fonte`, `url`, `rel DEFAULT 'V'`, `recolhido_em`.

**Inconsistência importante:** a tabela tem 0 registos, mas o `pico-civico` mostra 38 indicadores em 12 categorias. Esses dados vivem em `build/datasets/lajes_indicadores.json`, produzido numa corrida anterior, e a base foi recriada depois. **O JSON é atualmente a única cópia.** Reproduzi-lo exige correr `python3 -m lajes_acervo indicadores --municipio 4601`.

## 3.6 Bloco da Assembleia — cinco tabelas

Estas cinco são **100% derivadas** de `pagina`. `db.refazer_assembleia()` faz `DROP` + recria antes de cada processamento.

### `membro` — 162 registos

**Um membro é por MANDATO, não por pessoa.** Decisão deliberada: a bancada e a sua fiabilidade valem para o mandato em que foram apuradas; uma pessoa que serve dois mandatos tem duas linhas, porque pode ter mudado de lista e a ata de instalação de um mandato não confirma o outro.

**PK** `id`. **UNIQUE** `(municipio, orgao, mandato, nome_norm)`. **Índice** `ix_membro_pessoa (municipio, orgao, nome_norm)` — para cruzar a mesma pessoa entre mandatos.

| Campo | Tipo | Notas |
|---|---|---|
| `municipio` | TEXT NOT NULL | Código INE (`4601`) |
| `orgao` | TEXT NOT NULL DEFAULT 'assembleia' | Só `assembleia` em uso |
| `mandato` | TEXT NOT NULL | ex.: `2021-2025` |
| `nome` | TEXT NOT NULL | Forma curta usada nas atas (`Óscar Pimentel`) |
| `nome_norm` | TEXT NOT NULL | Sem acentos, minúsculas — casa grafias do OCR |
| `nome_oficial` | TEXT | Nome completo da ata de instalação, quando casado |
| `partido` | TEXT | Sigla normalizada; NULL quando não apurada |
| `partido_rel` | TEXT DEFAULT '~' | `V` = ata de instalação · `~` = cláusulas de voto · `?` = por apurar |

**Não há tabela de partidos.** `partido` é uma string livre normalizada por `assembleia.normaliza_partido()`. Valores observados: `PS`, `PSD`, `PSD/CDS`, `PSD/CDS/PPM`, `CDS-PP`, `CDU`, e nomes de movimentos de cidadãos (`Podemos Mais`).

### `sessao` — 109 registos

**PK** `id`. **FK** `documento_id → documento.id ON DELETE SET NULL`. **UNIQUE** `(municipio, orgao, data, tipo)` — `tipo` entra na chave porque a ata de **instalação** e a sessão ordinária do mesmo dia são dois documentos distintos. **Índice** `ix_sessao_data (municipio, data)`.

| Campo | Tipo | Notas |
|---|---|---|
| `data` | TEXT | ISO `AAAA-MM-DD` |
| `mandato` | TEXT | Derivado da data por `catalog.mandato_de()` |
| `tipo` | TEXT | `ordinaria` \| `extraordinaria` \| `instalacao` |
| `hora_abertura` | TEXT | Texto literal da ata ("eram catorze horas e cinquenta minutos") — **não normalizado** |
| `n_presentes`, `n_deliberacoes` | INTEGER | |
| `fonte_url` | TEXT | URL do PDF de origem |
| `motor_ocr` | TEXT | Proveniência da extração |
| `confianca` | TEXT DEFAULT '~' | `V` para atas de instalação, `~` para as restantes |

### `presenca` — 970 registos

**PK** `id`. **FK** `sessao_id → sessao.id CASCADE`, `membro_id → membro.id CASCADE`. **UNIQUE** `(sessao_id, membro_id)`.
Campos: `estado TEXT NOT NULL` (`presente` \| `ausente` \| `substituido` — só os dois primeiros em uso), `nome_como_lido TEXT` (grafia exata do OCR, auditável).

### `deliberacao` — 723 registos

**PK** `id`. **FK** `sessao_id → sessao.id CASCADE`. **UNIQUE** `(sessao_id, ordem)`.

| Campo | Tipo | Notas |
|---|---|---|
| `ordem` | INTEGER | Posição na ata |
| `assunto` | TEXT | Extraído de "Posta à votação, \<assunto\> foi aprovado…". **NULL em 215 de 723 (29,7%)** |
| `resultado` | TEXT | `aprovado` \| `rejeitado` \| `retirado` |
| `modo` | TEXT | `unanimidade` \| `maioria` |
| `n_favor`, `n_contra`, `n_abstencao` | INTEGER | Contagens de votos **nominais** |
| `excerto` | TEXT | Texto-fonte (até 400 chars) — para nunca citar sem lastro |

### `voto` — 5 781 registos

**PK** `id`. **FK** `deliberacao_id → deliberacao.id CASCADE`, `membro_id → membro.id CASCADE`. **UNIQUE** `(deliberacao_id, membro_id, partido, granularidade)`. **Índice** `ix_voto_delib (deliberacao_id)`.

| Campo | Tipo | Notas |
|---|---|---|
| `membro_id` | INTEGER | **NULL quando é voto de bancada** |
| `partido` | TEXT | Preenchido sempre que conhecido |
| `sentido` | TEXT NOT NULL | `favor` \| `contra` \| `abstencao` |
| `granularidade` | TEXT NOT NULL | `membro` \| `bancada` |

Dos 5 781: **5 155 com `granularidade='membro'`** (voto nominal, 2015-08-06 → 2026-02-23) e 626 com `bancada`.

## 3.7 Bloco eleitoral — duas tabelas

### `eleicao_territorio` — 120 registos

**PK** `id`. **UNIQUE** `(ano, territorio, eleicao)`. **Índice** `ix_elt_muni (municipio, eleicao, ano)`.

| Campo | Tipo | Notas |
|---|---|---|
| `ano` | INTEGER NOT NULL | 2017 ou 2021 |
| `municipio` | TEXT NOT NULL | Código INE |
| `territorio` | TEXT NOT NULL | Chave DGAI `LOCAL-DDMMFF` |
| `nome`, `nivel` | TEXT NOT NULL | `nivel` = `concelho` \| `freguesia` |
| `eleicao` | TEXT NOT NULL | `CM` \| `AM` \| `AF` |
| `inscritos`, `votantes` | INTEGER | **O denominador válido** |
| `afluencia` | REAL | % |
| `brancos`, `brancos_pct`, `nulos`, `nulos_pct` | | |
| `mandatos` | INTEGER | Total de mandatos em disputa |
| `fonte` | TEXT | URL exato do ficheiro de origem |
| `derivado_de` | INTEGER | Ano do ficheiro que trouxe este resultado como `previousResults` |

### `eleicao_lista` — 312 registos

**PK** `id`. **FK** `territorio_id → eleicao_territorio.id CASCADE`. **UNIQUE** `(territorio_id, sigla)`.
Campos: `sigla NOT NULL` (como vem na fonte), `sigla_norm` (**NULL quando não reconhecível** — a DGAI rotula a mesma coligação como `PPD/PSD.CDS-PP.PPM` em 2021 e `pm` em 2017), `votos`, `pct`, `pct_validos`, `mandatos`.

Siglas atualmente sem normalização: `pm`, `GRUPO CIDADÃOS`, `B.E.`, `MMSR`.

## 3.8 Entidades que o utilizador perguntou e NÃO existem

| Entidade pedida | Estado |
|---|---|
| `Person` genérica | `INEXISTENTE` — ver §4 |
| `Organization` genérica | `INEXISTENTE` — ver §5 |
| Partidos (tabela) | `INEXISTENTE` — string em `membro.partido` e `eleicao_lista.sigla` |
| Empresas (tabela) | `INEXISTENTE` — string em `contratos_raw.json.rows[].contracted` |
| Candidatos | `INEXISTENTE` — a DGAI expõe `PartiesCandidates`/`Candidate/candidates`, não recolhido |
| Deputados (nacionais/regionais) | `INEXISTENTE` — âmbito é municipal |
| Municípios / freguesias (tabelas) | `INEXISTENTE` — códigos como strings; geografia em `catalog.MUNICIPIOS` (Python) e `GEO`/`FMETA` (JS) |
| Intervenções / discursos | `INEXISTENTE` — o texto das intervenções está no OCR de `pagina`, **não extraído** |
| Propostas / requerimentos | `PARCIAL` — só como `deliberacao.assunto`, texto livre e muitas vezes NULL |
| Contratos (tabela) | `INEXISTENTE` — **vivem apenas em JSON**, nunca entraram no SQLite |
| Contas públicas | `INEXISTENTE` em tabela — só `financas.json` (3 concelhos × 2017–2019) |
| Notícias | `INEXISTENTE` |
| Temas / categorias | `INEXISTENTE` como entidade — ver §9 |
| Fontes (tabela) | `INEXISTENTE` — a proveniência é um campo (`url_origem`, `fonte`, `fonte_url`) |

---

# 4. Entidade "Pessoa"

## 4.1 Não existe uma entidade `Person`

A única representação de uma pessoa é a tabela **`membro`**, e ela **não é uma pessoa**: é *"um membro de um órgão num mandato"*. Consequências:

- Uma pessoa que serviu 5 mandatos tem **5 linhas** em `membro`, com `id` diferentes.
- Não há identificador estável de pessoa. Não há UUID, não há slug, não há chave externa.
- O único mecanismo de reconciliação entre mandatos é a função **`influencia.chave_pessoa(nome)`**, que devolve `primeiro_termo + " " + último_termo` desacentuado e em minúsculas (`"Antonino Lourenço Azevedo"` → `"antonino azevedo"`). **É calculada em runtime, não persistida.**
- `influencia.pessoas_ambiguas()` deteta quando essa chave reúne nomes incompatíveis e marca a longevidade como por confirmar. Casos atuais: `carlos freitas` (duas pessoas reais — *Carlos Eduardo da Cunha Freitas* e *Carlos Manuel Llano de Freitas*) e `nilton goulart`.

## 4.2 O que existe por pessoa

| Atributo pedido | Estado | Onde |
|---|---|---|
| Identificador estável | `INEXISTENTE` | — |
| Nome | `IMPLEMENTADO` | `membro.nome` (curto), `membro.nome_oficial` (completo, quando há ata de instalação) |
| Aliases / variantes de grafia | `PARCIAL` | `membro.nome_norm` (uma forma canónica); `presenca.nome_como_lido` guarda a grafia exata lida. `assembleia.canonicaliza()` funde variantes de OCR em runtime, com corte 0.90 |
| Cargo | `PARCIAL` | Implícito em `membro.orgao` + `mandato`. **Não há campo de cargo** — presidente da mesa, secretário, presidente de junta não estão distinguidos |
| Organização / bancada | `PARCIAL` | `membro.partido` (string) + `membro.partido_rel` (fiabilidade) |
| Localização | `INEXISTENTE` | A ata de instalação contém a freguesia de residência; é **deliberadamente descartada** (§17) |
| Histórico de funções | `PARCIAL` | Reconstituível por `nome_norm`/`chave_pessoa` através de várias linhas de `membro`; não há tabela de mandatos por pessoa |
| Relações pessoa↔pessoa | `INEXISTENTE` como aresta. Inferível de co-votação (`voto` partilhando `deliberacao_id`) |
| Ligação a documentos | `IMPLEMENTADO` (indireta) | `membro → presenca → sessao → documento → url_origem` |
| Ligação a intervenções | `INEXISTENTE` | O nome do interveniente está no texto OCR ("passou a palavra ao membro Ana Neves") mas **não é extraído** |
| Ligação a notícias | `INEXISTENTE` | Não há notícias no sistema |
| Redes sociais | `INEXISTENTE` | Nenhum campo, nenhuma recolha |
| Fotografia | `INEXISTENTE` | |
| Metadados livres | `INEXISTENTE` | Não há coluna JSON nem tabela de atributos |

## 4.3 Métricas calculadas por pessoa (em runtime, não persistidas)

`influencia.membros(conn, municipio)` devolve, por mandato: `nome`, `nome_oficial`, `nome_norm`, `pessoa` (chave), `partido`, `partido_rel`, `sessoes_mandato`, `presencas`, `ausencias`, `assiduidade` (%), `votos`, `favor`, `contra`, `abstencao`, `dissidencias`, `votos_com_linha`, `dissidencia_pct`, `maiorias`, `pivotal`, `pivotal_pct`, `mandatos`, `mandatos_rel`.

## 4.4 O modelo suportaria pessoas não-políticas?

**Não, sem alteração de esquema.** Razões estruturais:

1. `membro` tem `municipio NOT NULL`, `orgao NOT NULL`, `mandato NOT NULL`. Um empresário, jornalista ou investigador não tem mandato nem órgão. Estes três `NOT NULL` tornam a tabela inutilizável para qualquer pessoa fora de um órgão eleito.
2. A identidade é derivada do nome. Sem identificador estável, não há como ligar a mesma pessoa a papéis heterogéneos (sócio de empresa + dirigente associativo + autor de opinião).
3. Não há tabela de papéis/afiliações. `partido` é uma string única por linha — não suporta múltiplas afiliações simultâneas.

**Recomendação factual, não desenho:** representar empresários, investigadores, jornalistas, dirigentes associativos, artistas, desportistas, líderes comunitários, criadores de conteúdo ou cidadãos ativos no debate público exige uma entidade `pessoa` nova, com identificador próprio, e transformar `membro` numa tabela de **afiliação/mandato** que referencia essa pessoa. Isso é uma migração, não uma extensão.

---

# 5. Entidade "Organização"

**`INEXISTENTE`.** Não há tabela, coleção ou entidade de organização em nenhum dos repositórios. As organizações aparecem apenas como **strings**, em quatro sítios distintos e sem ligação entre si:

| Tipo de organização | Como está representada | Onde |
|---|---|---|
| Empresas (adjudicatários) | String no nome do contrato | `contratos_raw.json` → `rows[].contracted`. Agrupadas por `influencia._chave_forn()`: desacentua, remove formas societárias (`S A`, `LDA`, `UNIPESSOAL`, `SGPS`, `SOCIEDADE`, …), trunca a 34 chars. **Sem NIF** |
| Partidos | Sigla normalizada | `membro.partido`, `eleicao_lista.sigla` / `sigla_norm` |
| Movimentos de cidadãos | Nome da lista como "sigla" | `eleicao_lista.sigla` (`GRUPO CIDADÃOS`, `MMSR`) |
| Municípios (como entidade adjudicante) | Nome + NIF | `contratos_raw.json` → `rows[].contracting`, `rows[]._nif` |
| Associações, clubes, universidades, meios de comunicação | `INEXISTENTE` | Nenhuma recolha |

**Consequência crítica:** não existe identidade de organização. `AIRC - Associação de Informática da Região Centro` e `Tecnovia Açores, Sociedade de Empreitadas, S.A.` são chaves derivadas de texto. O próprio módulo declara esta limitação no que exporta:

> `nota_identidade`: *"Fornecedores agrupados por grafia do nome, não por NIF: a recolha do Portal BASE não devolve o NIF do adjudicatário. Empresas do mesmo grupo com nomes diferentes contam separadas."*

O campo `grafias` (número de variantes ortográficas agrupadas) é exportado por fornecedor, para dar visibilidade ao risco.

**Rede de participações/administrações partilhadas:** `INEXISTENTE` e **deliberadamente não automatizada**. O `publicacoes.mj.pt` (Registo Comercial) permite pesquisa por NIF/NIPC mas o formulário tem um controlo anti-bot (`NoBot1_NoBotExtender_ClientState`, AJAX Control Toolkit). Está documentado em `build/README.md` que construir um raspador que o contorne não foi considerado caminho legítimo.

---

# 6. Fontes de dados

## 6.1 IMPLEMENTADO

### F1 — Atas da Assembleia Municipal das Lajes do Pico
- **URL:** `https://cm-lajesdopico.pt/menu/{id}/{ano}` (29 IDs de página mapeados em `config.ATAS_ASSEMBLEIA`)
- **Conteúdo:** PDF de atas, 1998–2026. 107 dos 110 são **digitalizações sem camada de texto**
- **Método:** `discover.py` (HTMLParser: paginação por `data-total-pages`, dois formatos de listagem, âncoras `/upload_files/*.pdf`) → `download.py` (GET + sha256) → `ocr.py` (tesseract `por`, 300 dpi, `--psm 3`)
- **Automatização:** comando único `python3 -m lajes_acervo atas`; **invocado à mão**
- **Frequência de atualização:** `DESCONHECIDO` na origem. As atas são publicadas com atraso: a ata de 12/12/2025 foi publicada em 13/03/2026 (≈3 meses)
- **Formato:** HTML → PDF → texto
- **Qualidade:** OCR com erros reais e observados (`José Fermandes` vs `José Fernandes`; `Amilcar` vs `Amílcar`; `Ana VIA QUA Neves`). `pagina.confianca_ocr` é sempre NULL
- **Limitações:** voto nominal só a partir do mandato 2013-2017. Não existe página de 2021 da Assembleia (lacuna na origem, documentada em `config.LACUNAS`). Dois links partidos conhecidos em `config.LINKS_INVALIDOS_CONHECIDOS`
- **Termos/licença:** `DESCONHECIDO` — não foi verificado o `robots.txt` nem os termos do site. O `config.py` define um User-Agent identificável, pausa mínima de 0,5 s e máximo de 4 ligações, por educação, não por obrigação verificada
- **Histórico:** 1997-12-31 a 2026-02-23

### F2 — INE, API `pindica.jsp`
- **URL:** `https://www.ine.pt/ine/json_indicador/pindica.jsp?op=2&varcd={varcd}&Dim1={periodo}&Dim2={geocod}`
- **Conteúdo:** 9 indicadores municipais testados (`connectors/catalog.INE_CATALOGO`): médicos, enfermeiros e farmácias por 1000 hab., taxa bruta de mortalidade, crimes registados, taxa de criminalidade, taxa de retenção no básico, despesa das câmaras em cultura, resíduos urbanos
- **Método:** `connectors/ine.py` (`meta()`, `serie()`, `valor_municipio()`). `op=2` dados, `op=1` metadados. geocod municipal dos Açores = `200` + código de concelho. Períodos anuais = `S7A` + ano. `Dim3=T` = Total
- **Automatização:** `python3 -m lajes_acervo indicadores --municipio 4601`; à mão
- **Frequência:** anual na origem
- **Qualidade:** boa, oficial. **Verificado a responder (HTTP 200) em 2026-09-03**
- **Limitações:** transportes (acidentes), água/saneamento (séries 2001-05) e alunos não têm dado municipal fiável na API — assinalado no catálogo como "só regional"

### F3 — geoapi.pt (Censos INE)
- **URL:** `https://json.geoapi.pt/municipio/{nome}?json=1`
- **Conteúdo:** blocos `censos2011` / `censos2021` — edifícios, alojamentos, famílias, indivíduos por idade; área
- **Método:** `connectors/geoapi.py`
- **Gotcha documentado no código:** o campo `areaha` vem **em km², não em hectares** (validado contra Lajes = 155,31 km²)
- **Qualidade:** boa. **Verificado a responder (HTTP 200) em 2026-09-03**

### F4 — DGAI, resultados eleitorais autárquicos
- **URL:** `https://www.eleicoes.mai.gov.pt/autarquicas{ano}/assets/static/territory-results/territory-results-{chave}-{eleicao}.json`
- **Conteúdo:** inscritos, votantes, afluência, brancos, nulos, mandatos, votos por lista. Por concelho e por freguesia. Três eleições: `CM`, `AM`, `AF`
- **Método:** `connectors/eleicoes.py`. **O serviço vivo `/frontend/data/…` está desativado (404 em tudo).** O modo `useStaticFiles` da SPA Angular continua servido, e a chave do ficheiro é a concatenação dos *valores* dos parâmetros por `-` (comportamento do `objectToFileName` do bundle)
- **Automatização:** `python3 -m lajes_acervo eleicoes --ano 2021`; à mão
- **Qualidade:** oficial. Cada ficheiro traz `previousResults`, pelo que 2021 dá também 2017
- **Limitações:** `ANOS_SUPORTADOS = (2021,)`. **2025 está bloqueado:** a aplicação nova usa ids de território numéricos e opacos (`electionData.mainTerritories[].id`) que só se resolvem executando a SPA, e os caminhos estáticos mudaram (`assets/static/result/territory-*.json`) sem responder a chaves `LOCAL-*`. Via alternativa documentada: Mapa Oficial (CNE / Diário da República) — **não implementada**
- **Histórico obtido:** 2017 e 2021. O site lista autárquicas de 2001, 2005, 2007, 2009, 2013, 2017, 2021, 2025 — só 2021 tem layout legível por este conector

### F5 — Portal BASE (contratação pública)
- **URL documentado:** POST `https://www.base.gov.pt/Base4/pt/resultados/` com `type=search_contratos&version=139.0.0&query=<urlencoded>` e headers `X-Requested-With` + `Referer`
- **Conteúdo:** 735 contratos, 2021–2026, 3 concelhos. Campos: `id`, `contracted`, `contracting`, `contractingProcedureType`, `initialContractualPrice`, `objectBriefDescription`, `publicationDate`, `signingDate`, `ccp`, mais `_nif`, `_muni`, `_year` acrescentados na recolha
- **Método:** `build/fetch_contratos.py`
- **Estado da fonte:** ⚠️ **NÃO RECONFIRMADO.** Em 2026-09-03 o host devolveu HTTP 404 mesmo na raiz a partir do ambiente de teste, o que pode ser restrição de rede desse ambiente e não avaria da fonte. **A concluir antes da próxima recolha**
- **Limitações:** a resposta de pesquisa **não inclui o NIF do adjudicatário** — daí a identidade das empresas ser por grafia. Nenhum endpoint de detalhe por contrato foi encontrado
- **Qualidade:** boa nos valores; frágil na identidade das partes

### F6 — DGAL / AIQGP (finanças municipais)
- **URL:** ficheiros AIQGP anuais em `dados.gov.pt`
- **Conteúdo:** `divida_total`, `divida_pc`, `receita`, `despesa`, `saldo_orcamental`, `exec_receita`, `exec_despesa`, `prazo_pagamento`, `independencia`, `endividamento`, `pagamentos_atraso`
- **Método:** **MANUAL.** Não há conector. `build/datasets/financas.json` foi montado a partir dos ficheiros descarregados
- **Limitações:** só **2017–2019** em dados abertos. 2020+ exige SIIAL (acesso não público)

## 6.2 EM DESENVOLVIMENTO

Nada. Não há código a meio caminho.

## 6.3 PLANEADO

| Fonte | Evidência no código | Estado |
|---|---|---|
| Atas da Câmara Municipal (Lajes) | `config.ATAS_CAMARA` — 29 IDs de página mapeados, 1998–2026 | Nunca corrido. `cmd_atas --orgao camara` existe |
| Documentos financeiros do município | `config.FINANCAS` (2 secções) + `discover.discover_financas()` implementado com crawl em profundidade | Nunca corrido |
| Índice de texto FTS5 | `index.py` (docstring) + `cli` stub `index` | Sem código |
| Embeddings semânticos | `bloco.embedding BLOB` + comentário em `requirements.txt` | Sem código |
| Registo Comercial (interlocks) | `build/README.md` documenta a fonte e a razão de não estar automatizada | Bloqueado por decisão |
| Mapa Oficial CNE/DRE para 2025 | `connectors/eleicoes.py` docstring | Sem código |

## 6.4 Fontes que a pergunta menciona e NÃO existem

`INEXISTENTE` em qualquer forma — sem tabela, sem campo, sem código, sem dados:

- Notícias, imprensa, RSS
- Redes sociais (Facebook, Instagram, X, TikTok, YouTube, LinkedIn)
- Google Trends
- Parlamento nacional / Assembleia Legislativa Regional dos Açores
- Portais de transparência além do que já está em `IND` como URL de fonte
- SREA (Serviço Regional de Estatística dos Açores) — **aparece como texto de `fonte` em `lajes_indicadores.json` mas não há conector**. Nota: `srea.azores.gov.pt` devolveu HTTP 403 no teste de 2026-09-03
- Avaliações, reviews, dados de mercado
- Qualquer fonte comercial de social listening

---

# 7. Pipeline de ingestão

## 7.1 O que acontece com uma ata nova

```text
1. DESCOBERTA            discover.discover_menu(conn, orgao, seccao, menu_path, ano)
   HTML da página de menu → HTMLParser
   • paginação por atributo data-total-pages + ?page=N (não por links visíveis)
   • apanha TODAS as âncoras cujo href contenha /upload_files/ e termine em .pdf
   • rótulo = texto interno da âncora; se vazio, o title sem o prefixo "Transferir - "
   → metadata.parse_label(label, orgao)  → tipo_documento, tipo_reuniao,
     numero_reuniao, data_reuniao (ISO), ano, needs_review
   → db.upsert_descoberto()  |  UNIQUE(url_origem)  → estado='descoberto'

2. DESCARGA              download.descarregar(conn, orgao, desde_ano)
   ThreadPoolExecutor(max_workers=4)
   • se o ficheiro já existe em disco, reutiliza (não repete o GET)
   • valida que o corpo começa por %PDF; senão → estado='link_invalido'
   • HTTP 404 → estado='link_invalido' (definitivo, não aborta a corrida)
   • grava em data/pdf/{orgao}/{ano}/{id}-{nome_ficheiro}.pdf
   • sonda_pdf(): pdfinfo → n_paginas ; pdftotext -l 3 → tem_camada_texto
     (heurística: >400 caracteres não-espaço nas 3 primeiras páginas)
   → sha256, bytes, n_paginas, tem_camada_texto  → estado='descarregado'

3. OCR                   ocr.correr(conn, orgao, desde_ano, workers=4)
   • tem_camada_texto=1 → pdftotext -layout, split por \f  (motor='pdftotext')
   • senão → por página: pdftoppm -r 300 -gray -png  →  tesseract -l por --psm 3
     com OMP_THREAD_LIMIT=1 (sem isto o OpenMP do tesseract satura a máquina)
   → INSERT … ON CONFLICT(documento_id,numero_pagina) DO UPDATE  → estado='ocr_feito'
   Débito medido: 110 atas / 1 526 páginas em ≈13 min com 4 workers

4. EXTRAÇÃO ESTRUTURAL   assembleia.processar(conn, municipio)
   db.refazer_assembleia()  ← DROP + recria as 5 tabelas (são derivadas)
   Por mandato (catalog.mandato_de(data) usa as 8 datas de eleições autárquicas):
   4a. atas de instalação → parse_instalacao() → composição OFICIAL (nome + lista)
   4b. restantes atas → parse_ata() → tipo, hora, presenças, deliberações, votos
   4c. roster(atas, min_votos=3) → quem é membro e de que bancada, a partir das
       CLÁUSULAS DE VOTO (não da lista de chamada, que mistura o executivo)
   4d. reconcilia_oficiais() → identidade = nome oficial quando conhecido
   4e. presencas_casadas() → casa nomes do roster contra o bloco de chamada
   4f. gravação: upsert_sessao (apaga presenças/deliberações/votos da sessão antes
       de regravar) → gravar_presenca → gravar_deliberacao → gravar_voto

5. EXPORTAÇÃO            cmd_assembleia --export / cmd_influencia --export
   db.assembleia_export() / influencia.membros() + empresas() + db.eleicoes_export()
   → build/datasets/*.json  (versionados em git)

6. INJEÇÃO               build/inject.py NOME ficheiro.json index.html --chave lajes
   json.dumps(ensure_ascii=True, separators=(",",":")).replace("</","<\\/")
   splice por ÍNDICE entre `<script>var NOME=` e `</script>`
   verifica(): U+2028/U+2029 ausentes, JSON válido, sem `</` não escapado
   → aborta a escrita se o HTML resultante for inválido

7. PUBLICAÇÃO            git commit + push
```

## 7.2 Tratamento de duplicados

| Nível | Mecanismo |
|---|---|
| Documento | `UNIQUE(url_origem)` — descobrir duas vezes não duplica |
| Ficheiro em disco | `sha256` calculado; ficheiro existente é reutilizado sem novo GET |
| Página | `UNIQUE(documento_id, numero_pagina)` com `ON CONFLICT DO UPDATE` |
| Sessão | `UNIQUE(municipio, orgao, data, tipo)`. Duas atas com o mesmo título e data colapsam numa sessão (caso real: 2000-02-28 duplicada na origem) |
| Deliberação | `assembleia._dedup()` — as atas repetem a mesma fórmula de votação; colapsa por `(assunto, resultado, modo, votantes)` |
| Voto | `UNIQUE(deliberacao_id, membro_id, partido, granularidade)` com `INSERT OR IGNORE` |
| Membro | `UNIQUE(municipio, orgao, mandato, nome_norm)`; variantes de OCR fundidas por `canonicaliza()` (difflib, corte 0.90, exige mesmo número de palavras e primeiro OU último termo igual) |
| Território eleitoral | `UNIQUE(ano, territorio, eleicao)` |

## 7.3 Identificação de entidades

**Não há NER.** Toda a identificação é por expressão regular sobre a fraseologia ritual das atas:

- **Nomes:** `_NOME` exige inicial maiúscula (com acento) e ≥2 palavras; `_valido()` rejeita ≤4 palavras, tokens todos-maiúsculas no meio (lixo de OCR) e termos de cargo
- **Bancadas:** ancoradas em `eleit[oa]s? pel[oa] <PARTIDO>`; `_corta_partido()` separa a designação do primeiro nome quando falta a vírgula, mas **só quando o prefixo é uma bancada reconhecida** — senão um movimento de cidadãos seria cortado ao meio
- **Sentido de voto:** propaga-se pelas cláusulas seguintes até novo marcador (`a favor` / `contra` / `de abstenção`), que é como as atas estão escritas. Em unanimidades sem marcador, o sentido é deduzido do resultado
- **Corroboração:** um nome só conta como membro com ≥3 votos (`min_votos`); os descartados são devolvidos para auditoria, não silenciados

## 7.4 Classificação temática

`INEXISTENTE`. Nenhum documento, deliberação ou contrato é classificado por tema.

## 7.5 Tratamento de erros

- `http.fetch()`: 4 tentativas, espera `2.0 ** tentativa`; HTTP 404 levanta imediatamente (é definitivo)
- Um ano de descoberta que falhe **não aborta a corrida** — grava `execucao(estado='erro')` e continua
- `download`: erros contados em `erros`, documento fica no estado anterior
- `parse_instalacao()` levanta `AssertionError` se dados pessoais escaparem ao extractor — **falha em vez de publicar**
- `inject.py` aborta a escrita se o HTML resultante não validar
- **Não há retry queue, dead-letter, nem alerta.** Os erros vão para `stderr` e desaparecem

## 7.6 Versionamento

- **PDF:** `sha256` por documento. Uma nova versão do mesmo URL substitui o ficheiro e o hash — **a versão anterior não é preservada**
- **Texto:** `ON CONFLICT DO UPDATE` sobrescreve
- **Tabelas derivadas:** `DROP` + recriar a cada processamento
- **Datasets:** versionados em git — o histórico do repositório **é** o versionamento dos dados exportados
- **Não há tabela de histórico, nem `valid_from`/`valid_to`, nem soft delete**

---

# 8. Inteligência artificial atualmente utilizada

## NENHUMA.

Isto não é uma simplificação. Verificado por busca exaustiva em todo o código dos dois repositórios (`grep -rniE "openai|anthropic|claude|gpt|llm|embedding|sentence.transformer|faiss|chroma|pgvector|qdrant|weaviate|transformers"`): **zero ocorrências funcionais**. As únicas correspondências são a coluna `bloco.embedding BLOB` (tabela vazia), a docstring de `index.py`, e um comentário em `requirements.txt`.

| Função | Modelo | Input | Output | Onde é armazenado |
|---|---|---|---|---|
| Classificação de documentos | — | — | — | `INEXISTENTE` |
| Resumo | — | — | — | `INEXISTENTE` |
| Extração de entidades (NER) | — | — | — | `INEXISTENTE` |
| Reconhecimento de pessoas | **Expressões regulares + `difflib.SequenceMatcher`** | Texto OCR da ata | Lista de nomes canónicos | `membro`, `presenca.nome_como_lido` |
| Identificação de organizações | **Expressões regulares** (bancadas) / **normalização de strings** (empresas) | Cláusulas de voto / nome do adjudicatário | Sigla / chave truncada a 34 chars | `membro.partido`, JSON de empresas |
| Classificação temática | — | — | — | `INEXISTENTE` |
| Análise de sentimento | — | — | — | `INEXISTENTE` |
| Embeddings | — | — | — | `INEXISTENTE` |
| Pesquisa semântica | — | — | — | `INEXISTENTE` |
| RAG | — | — | — | `INEXISTENTE` |
| Geração de texto | — | — | — | `INEXISTENTE` |
| Análise de discursos | — | — | — | `INEXISTENTE` |
| Reconhecimento de relações | — | — | — | `INEXISTENTE` |
| Processamento de PDF | **tesseract 5.3.4 (LSTM, lang=por)** + poppler | PDF digitalizado, 300 dpi cinza | Texto por página | `pagina.texto` |

O tesseract é o único componente de aprendizagem automática no sistema. É OCR clássico, não um LLM, e a confiança que produz **não é capturada** (`pagina.confianca_ocr` é sempre NULL, porque é invocado sem `--tsv`).

---

# 9. Taxonomia temática atual

**`INEXISTENTE` como entidade.** Não há tabela de temas, não há hierarquia, não há classificação de conteúdos.

O que existe são **duas listas de strings de categoria**, ambas fixas no código, sem relação entre si e sem aplicação a documentos:

### 9.1 `categoria` dos indicadores

`indicador.categoria` (TEXT, tabela vazia) e `lajes_indicadores.json` usam 12 valores:

`Demografia` · `Habitação` · `Economia` · `Emprego/Rendimento` · `Educação` · `Saúde` · `Ação Social` · `Segurança/Justiça` · `Ambiente/Energia/Resíduos` · `Cultura/Desporto/Lazer` · `Território/Ordenamento` · `Governação/Finanças`

### 9.2 `categoria` do catálogo INE

`connectors/catalog.INE_CATALOGO` usa 5 valores, **com nomes diferentes** dos anteriores: `Saúde` · `Segurança/Justiça` · `Educação` · `Cultura/Desporto/Lazer` · `Ambiente/Energia/Resíduos`

### 9.3 Respostas diretas

| Pergunta | Resposta |
|---|---|
| Como são os temas definidos? | Escritos à mão em duas listas Python. Não há vocabulário controlado |
| Podem ser hierárquicos? | Não. São strings planas. Algumas têm barra (`Emprego/Rendimento`) mas isso é notação, não hierarquia |
| Um conteúdo pode ter vários temas? | Não aplicável — nenhum conteúdo é classificado. Um *indicador* tem exatamente uma categoria |
| Existem subtemas? | Não |
| Manual ou automática? | Manual, e apenas para indicadores |
| Como ficam guardados? | Como string na coluna `categoria`. Sem tabela, sem id, sem FK |

**Nenhuma deliberação, ata, contrato ou resultado eleitoral tem tema atribuído.** Existe `deliberacao.assunto` (texto livre extraído da ata, frequentemente NULL) — é o material bruto de onde uma classificação temática poderia partir, mas não é uma classificação.

---

# 10. Séries temporais

## 10.1 O que tem dimensão temporal

| Dados | Granularidade | Período | Onde |
|---|---|---|---|
| Sessões da Assembleia | Data exata | 1997-12-31 → 2026-02-23 | `sessao.data` |
| Deliberações | Herdada da sessão | idem | via `deliberacao.sessao_id` |
| Votos nominais | Herdada da deliberação | **2015-08-06 → 2026-02-23** | via `voto → deliberacao → sessao` |
| Presenças | Herdada da sessão | 1997 → 2026 (162 membros-mandato) | via `presenca → sessao` |
| Mandatos | Quadriénio | 8 mandatos, 1997–2029 | `sessao.mandato`, `membro.mandato` |
| Contratos | Data de publicação e de assinatura | 2021 → 2026 | `contratos_raw.json`: `publicationDate`, `signingDate` (formato `DD-MM-AAAA`) |
| Finanças | Anual | 2017, 2018, 2019 | `financas.json` |
| Eleições | Por ato eleitoral | 2017, 2021 | `eleicao_territorio.ano` |
| Indicadores | Anual | vários, 2011–2023 | `indicador.ano` / campo `ano` no JSON |
| Corridas do pipeline | Timestamp | 2026-09-03 | `execucao.iniciado_em` |

## 10.2 O exemplo pedido: "menções por semana"

```text
Pessoa X — número de menções por semana
2025-W01 → 12
```

**IMPOSSÍVEL. Não existe o conceito de menção.** Não há tabela de menções, não há corpus de notícias ou redes sociais, e o texto que existe (`pagina.texto`) não está indexado nem ligado a pessoas.

O que **é** possível hoje, com SQL directo:

```sql
-- Votos nominais de um membro por ano
SELECT strftime('%Y', s.data) AS ano, COUNT(*) AS votos
FROM voto v
JOIN deliberacao d ON d.id = v.deliberacao_id
JOIN sessao s      ON s.id = d.sessao_id
JOIN membro m      ON m.id = v.membro_id
WHERE v.granularidade = 'membro' AND m.nome_norm = 'antonio santos'
GROUP BY ano ORDER BY ano;

-- Presenças por mandato
SELECT s.mandato, p.estado, COUNT(*) FROM presenca p
JOIN sessao s ON s.id = p.sessao_id
JOIN membro m ON m.id = p.membro_id
WHERE m.nome_norm = 'carlos simas' GROUP BY s.mandato, p.estado;

-- Deliberações por semestre
SELECT strftime('%Y', s.data) AS ano,
       CASE WHEN CAST(strftime('%m', s.data) AS INT) <= 6 THEN 'S1' ELSE 'S2' END AS sem,
       COUNT(d.id)
FROM deliberacao d JOIN sessao s ON s.id = d.sessao_id GROUP BY ano, sem;
```

## 10.3 O exemplo pedido: "tema Habitação por mês"

**IMPOSSÍVEL.** Sem taxonomia temática (§9) e sem classificação de conteúdos, não há como contar ocorrências por tema. Nem o texto OCR permite contagem por palavra-chave de forma eficiente — não há índice.

## 10.4 Estruturas que permitem análise temporal

| Estrutura | Serve para |
|---|---|
| `sessao(municipio, data)` — índice `ix_sessao_data` | Qualquer agregação temporal de atividade do órgão |
| `sessao.mandato` | Agregação por ciclo político |
| `voto → deliberacao → sessao` | Séries de comportamento de voto (a série mais rica: 5 155 pontos com data, pessoa, bancada e sentido) |
| `presenca → sessao` | Séries de assiduidade |
| `eleicao_territorio(ano, territorio, eleicao)` + `derivado_de` | Comparação entre atos eleitorais |
| `documento(data_reuniao, descoberto_em, atualizado_em)` | Latência entre a reunião e a publicação da ata |

**A série temporal mais valiosa que já existe:** os 5 155 votos nominais com data, pessoa, bancada e sentido, de agosto de 2015 a fevereiro de 2026. Nenhuma outra plataforma municipal portuguesa conhecida tem isto estruturado. `DESCONHECIDO`: se existe equivalente noutro município.

---

# 11. Dados eleitorais

## 11.1 O que existe

| Dimensão | Cobertura |
|---|---|
| Atos eleitorais | **2 anos: 2017 e 2021** (autárquicas). 2021 recolhido diretamente; 2017 vem do campo `previousResults` do ficheiro de 2021 (dado real da fonte, não interpolado — marcado em `derivado_de`) |
| Tipos de eleição | 3: `CM` Câmara Municipal, `AM` Assembleia Municipal, `AF` Assembleia de Freguesia |
| Nível geográfico | **Concelho e freguesia.** 20 territórios (3 concelhos + 17 freguesias) × 3 eleições × 2 anos = 120 registos |
| Resultados por lista | 312 registos em `eleicao_lista` |
| Inscritos | Sim — `eleicao_territorio.inscritos`. **É o denominador válido** |
| Votantes / afluência | Sim |
| Abstenção | Derivável: `inscritos - votantes`; ou `100 - afluencia` |
| Brancos e nulos | Sim, absolutos e percentagem |
| Mandatos | Sim, por território e por lista |
| Candidatos (nominais) | **`INEXISTENTE`.** A DGAI expõe `PartiesCandidates` / `Candidate/candidates`; não foi recolhido |
| Círculos eleitorais | Não aplicável ao nível autárquico modelado |
| Séries históricas longas | **`INEXISTENTE`.** O site da DGAI tem 2001, 2005, 2007, 2009, 2013, 2017, 2021, 2025 — só 2021 é legível pelo conector atual |

## 11.2 Exemplo de dados reais (AM 2021, Lajes do Pico)

Concelho: 4 469 inscritos · 3 142 votantes · 70,31% de afluência · 89 brancos (2,83%) · 41 nulos (1,30%) · 15 mandatos.
Listas: `PS` 1 566 votos / 49,84% / 8 mandatos · `PPD/PSD.CDS-PP.PPM` 1 311 / 41,73% / 7 · `PCP-PEV` 135 / 4,30% / 0.

Freguesias (AM 2021): Calheta de Nesquim 69,7% · Lajes do Pico 70,0% · Piedade 71,3% · Ribeiras 69,2% · Ribeirinha 70,7% · São João 72,0%.

## 11.3 Ligação entre eleições e o órgão

**`INEXISTENTE` como relação explícita.** Não há FK entre `eleicao_lista` e `membro`, nem entre `eleicao_territorio` e `sessao`. A ligação é apenas conceptual: o mandato `2021-2025` corresponde à eleição de 2021, através de `catalog.MANDATOS` (lista Python de 8 pares data→etiqueta baseada nas datas oficiais das eleições autárquicas: 1997-12-14, 2001-12-16, 2005-10-09, 2009-10-11, 2013-09-29, 2017-10-01, 2021-09-26, 2025-10-12).

---

# 12. Frontend e visualizações

## 12.1 Estrutura

**Uma única página.** Não há rotas, não há URLs por entidade, não há histórico de navegação (`pushState` não é usado). Todo o estado é em memória. Recarregar volta ao início.

`pico-civico/index.html` — 677 247 bytes, 1 239 linhas, das quais **5 linhas são blocos de dados** que somam 538 003 bytes (79% do ficheiro).

| Bloco | Bytes | Conteúdo |
|---|---|---|
| `var GEO` | ~48 000 | GeoJSON, 17 freguesias (`properties`: `nome`, `concelho`). **Está dentro de um `<script>` maior com `var GEO = ` (com espaços) — não é reconhecido pelo `inject.py`** |
| `var DEEP` | 218 906 | Forense de contratação por concelho: `for` (agregados) + `contr` (lista de contratos: `id`, `forn`, `proc`, `val`, `data`, `obj`) |
| `var FIN` | 2 348 | Finanças, 3 concelhos × 2017–2019 |
| `var IND` | 7 152 | Indicadores de Lajes: 12 categorias, 38 indicadores (`lab`, `val`, `u`, `ano`, `fonte`, `rel`) |
| `var INFL` | 99 710 | `pesos`, `titulares` (por mandato), `empresas`, `eleicoes`, `revisao` |
| `var ASSEM` | 209 887 | `mandatos` (sumário) + `detalhe` (por mandato: composição, membros, sessões, deliberações) |

Mais duas estruturas **escritas à mão em JavaScript**, não geradas por pipeline:
- `FMETA` — 17 freguesias com `c` (concelho), `pop` (população, string), `t` (descrição)
- `CONC` — 3 concelhos com `name`, `trace`, `pop`, `area`, `dens`, `gov` (presidente), `camara`, `assembleia`, `contratos`, `ind` (fallback de 5 indicadores), `dr` (**dados demonstrativos**)

## 12.2 Separadores do perfil de concelho

`var SECS` define 7, com visibilidade condicional:

| Chave | Nome | Condição |
|---|---|---|
| `geral` | Visão geral | sempre |
| `indic` | Indicadores | sempre (cai no fallback `CONC[].ind` se não houver `IND[id]`) |
| `gov` | Governação | sempre (dados de `CONC`) |
| `assem` | Assembleia | só se `ASSEM[id]` — **hoje só `lajes`** |
| `infl` | Influência | só se `INFL[id]` — **hoje só `lajes`** |
| `contr` | Contratos | sempre |
| `fin` | Finanças | só se `FIN[id]` — os 3 concelhos |

## 12.3 Visualizações implementadas

| Visualização | Função | Técnica |
|---|---|---|
| Mapa da ilha por freguesia, com drill-down | `onGeo`, `drillInto`, `pickFreg` | `<path>` SVG criados com `createElementNS`; projeção equirretangular com correção `cos(lat)` |
| Migalhas de navegação | `crumb` | HTML |
| Sparkline | `spark` | `<polyline>` SVG |
| Barra de bancadas | `seatBar`, `seatBarP` | `<span>` com `width` em % |
| Barras anuais de contratação | `contratosSectionDeep` | `<div>` com `height` em px |
| Cartões de sinais forenses | `sinaisBlock`, `sevChip` | HTML com classes de severidade |
| Rede cross-concelho | `redeBlock` | Lista de chips, **não é um grafo** |
| Tabela pesquisável de contratos | `tabelaBlock`, `renderTabela` | `filter` + `indexOf`, limitada a 150 linhas |
| Cartões de indicadores por categoria | `indCategorias` | Grelha CSS |
| Barras de dívida | `financasBlock` | `<div>` com altura |
| Cobertura do acervo por mandato | `assembleiaCobertura` | Barras empilhadas |
| Tabela de presenças e voto por membro | `membroLinha` | Grelha CSS 4 colunas + barra de presenças + barra de voto tripartida |
| Índice de peso com parcelas | `partesBar`, `influTitulares` | Barra empilhada com largura = valor × peso |
| Cartões de fornecedor | `influEmpresas` | Grelha + barra de parcelas + etiquetas de escrutínio |
| Resultados eleitorais por freguesia | `influEleicoes`, `barras` | Barra empilhada por lista + brancos/nulos |
| Timeline | `INEXISTENTE` | |
| Comparação entre entidades | `INEXISTENTE` | Não há vista de comparação |
| Rankings | `PARCIAL` | As tabelas de influência são ordenadas por peso, mas não há vista de ranking dedicada |
| Filtros | `PARCIAL` | Só o input de texto dos contratos e os chips de mandato/eleição |
| Dashboard agregado da ilha | `PARCIAL` | `renderOverview` mostra população, nº de concelhos e freguesias |

## 12.4 Biblioteca de gráficos

**Nenhuma.** Zero dependências JavaScript. Não há D3, Chart.js, Plotly, Recharts, Leaflet, Mapbox. Todo o SVG e todas as barras são construídos com `document.createElementNS` ou concatenação de strings HTML. A única dependência externa em runtime é a folha de estilo do Google Fonts.

---

# 13. APIs internas

**`INEXISTENTE`.** Não há nenhum endpoint HTTP. Não há `GET /people`, não há `GET /documents`, não há servidor.

A superfície programática existente é a **linha de comandos** e as **funções de exportação**:

## 13.1 CLI (`python3 -m lajes_acervo <comando>`)

| Comando | Estado | Função |
|---|---|---|
| `init-db` | `IMPLEMENTADO` | Cria o esquema |
| `discover [--atas] [--financas] [--all]` | `IMPLEMENTADO` | Descoberta de PDF |
| `atas [--orgao] [--desde] [--workers] [--so-processar]` | `IMPLEMENTADO` | descoberta + descarga + OCR |
| `assembleia [--municipio] [--min-votos] [--mandato] [--export]` | `IMPLEMENTADO` | OCR → sessao/presenca/deliberacao/voto + exportação |
| `eleicoes [--ano] [--municipio] [--export]` | `IMPLEMENTADO` | Resultados DGAI (`--ano` aceita apenas 2021) |
| `influencia [--municipio] [--top] [--export]` | `IMPLEMENTADO` | Métricas de peso público + exportação |
| `indicadores [--municipio] [--export]` | `IMPLEMENTADO` | INE + geoapi |
| `status` | `IMPLEMENTADO` | Contagens por estado |
| `index` | `PLANEADO` (stub) | Imprime a mensagem "M4 — FTS5 léxico; embeddings depois" |
| `run [--all]` | `PARCIAL` | Orquestrador |

## 13.2 Funções de exportação (a "API" real)

```python
db.assembleia_mandatos(conn, municipio, orgao='assembleia') -> list[dict]
    # [{mandato, n_sessoes, de, ate, n_delib, n_votos_nominais}, …]

db.assembleia_export(conn, municipio, mandato=None, orgao='assembleia') -> dict
    # {mandato, n_sessoes, composicao[], membros[], sessoes[]}

db.eleicoes_export(conn, municipio) -> dict
    # {CM: {ano: [território…]}, AM: {…}, AF: {…}}

influencia.membros(conn, municipio, orgao='assembleia') -> dict[mandato, list[dict]]
influencia.empresas(contratos, top=25) -> dict
influencia.colisoes_pessoa(conn, municipio) -> list[dict]
influencia.pessoas_ambiguas(conn, municipio) -> set[str]

db.indicadores_por_categoria(conn, municipio) -> dict
```

## 13.3 Utilitário de injeção

```bash
python3 build/inject.py NOME ficheiro.json /caminho/index.html [--chave lajes]
python3 build/inject.py --verificar /caminho/index.html
```

---

# 14. Código relevante

## 14.1 Ordem de leitura recomendada

```text
1. lajes_acervo/db.py                    (487)  Esquema completo + toda a persistência
2. lajes_acervo/assembleia.py            (869)  Extração estrutural — o módulo central
3. lajes_acervo/influencia.py            (315)  Métricas de peso público
4. lajes_acervo/connectors/eleicoes.py   (152)  Conector DGAI + a razão de 2025 falhar
5. lajes_acervo/cli.py                   (243)  Todos os comandos e o que é stub
6. pico-civico/index.html              (1 239)  Interface + dados inline
7. lajes_acervo/connectors/catalog.py     (57)  Municípios, varcd INE, mandatos
8. lajes_acervo/config.py                 (71)  IDs de página, lacunas conhecidas
9. lajes_acervo/ocr.py                   (121)  Pipeline de OCR + o fix OMP_THREAD_LIMIT
10. lajes_acervo/discover.py             (154)  Descoberta de PDF
11. build/inject.py                      (108)  Padrão de injeção ASCII-safe
12. build/README.md                             Gotchas das fontes, documentados
13. tests/test_assembleia.py             (247)  Excertos REAIS de atas como especificação
14. tests/test_influencia.py              (97)  A definição de pivotalidade, testada
```

## 14.2 Estrutura dos projetos

```text
lajes-acervo/                          (motor de ingestão, Python, 3 720 linhas)
├── lajes_acervo/
│   ├── __init__.py            (3)
│   ├── __main__.py            (2)     python3 -m lajes_acervo
│   ├── cli.py               (243)     argparse; todos os comandos
│   ├── config.py             (71)     BASE, ATAS_CAMARA (29), ATAS_ASSEMBLEIA (25),
│   │                                  FINANCAS, LACUNAS, LINKS_INVALIDOS_CONHECIDOS
│   ├── db.py                (487)     SCHEMA (12 tabelas) + persistência + exportação
│   ├── http.py               (41)     GET educado com retry/backoff
│   ├── discover.py          (154)     HTMLParser; paginação; âncoras de PDF
│   ├── metadata.py          (128)     Rótulo → data/tipo/número (puro, testável)
│   ├── download.py           (95)     GET do PDF, sha256, sonda de páginas
│   ├── ocr.py               (121)     pdftotext | pdftoppm+tesseract
│   ├── assembleia.py        (869)     ★ presenças, deliberações, voto nominal,
│   │                                  roster, canonicalização, atas de instalação
│   ├── influencia.py        (315)     ★ pivotalidade, dissidência, peso económico
│   ├── index.py               (7)     PLANEADO — só docstring (FTS5/embeddings)
│   └── connectors/
│       ├── __init__.py        (7)
│       ├── catalog.py        (57)     MUNICIPIOS, INE_CATALOGO, MANDATOS, mandato_de()
│       ├── ine.py            (58)     API pindica
│       ├── geoapi.py         (72)     Censos via geoapi.pt (gotcha do areaha)
│       └── eleicoes.py      (152)     ★ DGAI; documenta a lacuna de 2025
├── build/
│   ├── README.md                      ★ gotchas de TODAS as fontes
│   ├── inject.py            (108)     Injeção ASCII-safe no index.html
│   ├── fetch_contratos.py    (62)     Portal BASE → contratos_raw.json
│   ├── forensics_all.py      (79)     Forense dos 3 concelhos → deep_all.json
│   ├── forensics.py         (113)     Versão só-Lajes (referência)
│   └── datasets/                      ~1,04 MB, versionado em git
│       ├── contratos_raw.json         373 605 B — 735 contratos
│       ├── deep_all.json              194 599 B — forense (alimenta DEEP)
│       ├── financas.json                4 489 B — DGAL 2017-19 (alimenta FIN)
│       ├── ine_catalog.json             4 017 B — 9 varcd testados
│       ├── lajes_indicadores.json       7 370 B — 38 indicadores (alimenta IND)
│       ├── lajes_assembleia.json      273 784 B — 8 mandatos (alimenta ASSEM)
│       ├── lajes_influencia.json      138 438 B — peso público (alimenta INFL)
│       └── pico_freguesias.geojson     47 988 B — 17 freguesias (alimenta GEO)
├── tests/                             49 testes, pytest
│   ├── test_metadata.py      (88)
│   ├── test_discover.py      (44)     + fixtures/ (2 HTML reais do site)
│   ├── test_assembleia.py   (247)     ★ excertos reais como especificação
│   └── test_influencia.py    (97)
├── data/                              ★ GITIGNORED — não está em git
│   ├── acervo.sqlite                  A base de dados (12 tabelas)
│   └── pdf/assembleia/{1998..2026}/   110 PDF, fonte da verdade
├── requirements.txt                   Apenas comentários — zero deps
├── README.md
└── .gitignore                         data/, *.sqlite*, __pycache__/, .venv/

pico-civico/                           (plataforma, 3 ficheiros)
├── index.html             (1 239)     ★ TUDO: CSS, JS, dados inline
├── README.md                 (50)
└── .gitignore                         .DS_Store
```

**Pastas que a pergunta pressupõe e não existem:** `/backend`, `/frontend`, `/api`, `/models`, `/services`, `/components`, `/jobs`, `/scrapers`, `/migrations`, `/.github/workflows`.

---

# 15. Limitações atuais

## 15.1 Dívida técnica

| # | Problema | Impacto |
|---|---|---|
| D1 | **Não há camada de serviço.** O SQLite é um artefacto de build local, gitignored, e a plataforma consome cópias JSON congeladas. Não há caminho de leitura em runtime | Qualquer funcionalidade dinâmica (pesquisa, filtros do lado do servidor, atualização incremental) exige construir backend de zero |
| D2 | **Dados de governação escritos à mão em JavaScript.** `CONC` (presidente, executivo, câmara, assembleia, população, área) e `FMETA` (17 freguesias) não vêm de base de dados nem de pipeline | Não é reproduzível, não é verificável, não escala, e diverge silenciosamente das fontes |
| D3 | **`indicador` está vazia** mas a plataforma mostra 38 indicadores a partir de `lajes_indicadores.json`. O JSON é a única cópia | Perda de dados se o JSON se corromper. A base não é a fonte da verdade que o esquema sugere |
| D4 | **Contratos nunca entraram no SQLite.** Vivem em `contratos_raw.json` e `deep_all.json` | Não há como cruzar contratos com deliberações em SQL. O cross-link "decisão → contrato" está bloqueado por isto |
| D5 | **`PRAGMA foreign_keys` não é ativado.** As FK estão declaradas mas não impostas | Órfãos possíveis; `ON DELETE CASCADE` não dispara |
| D6 | **Sem migrações.** `CREATE TABLE IF NOT EXISTS` não altera tabelas existentes. Foi preciso criar `refazer_assembleia()` (DROP + recria) | Qualquer alteração de esquema numa tabela não-derivada (`documento`, `pagina`) exige intervenção manual |
| D7 | **`var GEO` não é gerido pelo injetor.** Está declarado como `var GEO = ` (com espaços) dentro de um `<script>` maior, e o `inject.py` procura `<script>var NOME=` | O GeoJSON não é validado pelo `verifica()` nem atualizável pelo pipeline |
| D8 | **Índice de peso composto com pesos arbitrários.** `PESOS` é uma leitura editorial | Mitigado: os pesos estão à vista, as parcelas são mostradas ao lado do total, e a página di-lo. Mas continua a ser uma escolha não empírica |
| D9 | **`sessao.hora_abertura` guarda texto literal** ("eram catorze horas e cinquenta minutos") | Não é ordenável nem comparável |
| D10 | **`execucao` grava `iniciado_em` e `terminado_em` iguais** | Não mede duração; inútil para detetar degradação |
| D11 | **`pagina.confianca_ocr` sempre NULL** — o tesseract é invocado sem `--tsv` | Não há como priorizar revisão pelas páginas de pior OCR |
| D12 | **Ficheiro único de 677 KB com 79% de dados inline** | Cada atualização de dados reescreve o HTML; o `git diff` é ilegível; o crescimento é linear com os dados |

## 15.2 Problemas de arquitetura

- **A1 — Não há separação entre dados e apresentação.** Os dados são parte do ficheiro HTML. Adicionar um segundo consumidor (app, API, outro dashboard) exige duplicar os dados ou construir a camada que não existe.
- **A2 — A ingestão é acoplada a um município.** `discover.py` e `config.py` estão escritos contra `cm-lajesdopico.pt`. Cada novo município é um descobridor novo.
- **A3 — A extração é acoplada à fraseologia.** `assembleia.py` funciona porque as atas das Lajes são rituais e consistentes. Noutro município as expressões regulares terão de ser recalibradas. A arquitetura (roster → presenças casadas → votos) é transferível; os padrões não.
- **A4 — Não há orquestração.** Cada etapa é um comando manual. A ordem correta (`atas` → `assembleia` → `influencia` → `inject`) está documentada mas não imposta.

## 15.3 Escalabilidade

| Limite | Situação atual | Onde quebra |
|---|---|---|
| Volume de documentos | 110 atas / 1 526 páginas / 2,96 MB de texto | SQLite aguenta ordens de magnitude mais. **O gargalo é o OCR:** ≈2–3 s/página com 4 workers ⇒ ≈29 páginas/min. 10 000 páginas ≈ 6 h |
| Tamanho do frontend | 677 KB | Replicar Assembleia + Influência aos 3 concelhos multiplicaria `ASSEM`+`INFL` por ~3 ⇒ **>1,5 MB**. Aos 19 concelhos dos Açores seria impraticável inline |
| Concorrência | Nenhuma — é um ficheiro estático | Não aplicável até haver backend |
| SQLite | 1 escritor | Suficiente para pipeline sequencial; insuficiente para ingestão contínua concorrente |

## 15.4 Inconsistências do modelo de dados

- **I1** — `membro` é por mandato mas chama-se "membro", o que sugere pessoa. A identidade de pessoa é calculada em runtime e não persistida.
- **I2** — Duas listas de `categoria` com nomes divergentes (§9.1 vs §9.2).
- **I3** — `financas.json` usa chaves `lajes_do_pico` / `sao_roque_do_pico`; o resto do sistema usa `lajes` / `sao-roque` (JS) e `4601` / `4603` (Python). **Três convenções para o mesmo concelho.**
- **I4** — `eleicao_lista.sigla_norm` pode ser NULL (correto), mas `membro.partido` também pode — e não há forma de distinguir "não apurado" de "independente".
- **I5** — `documento.orgao` aceita `financas`, que não é um órgão.

## 15.5 Entidades duplicadas (medido)

`influencia.colisoes_pessoa()` devolve **17 grupos** de nomes que a chave primeiro+último reúne. Destes:
- 15 são a mesma pessoa em forma curta e longa (fusão **correta**)
- **2 são ambíguos**: `carlos freitas` (*Carlos Eduardo da Cunha Freitas* vs *Carlos Manuel Llano de Freitas* — duas pessoas) e `nilton goulart` (*Nilton André Cruz Goulart* vs *Nilton Cruz André Goulart* — provavelmente a mesma, ordem de termos trocada pelo OCR)

`assembleia.possiveis_duplicados()` sinaliza pares não fundidos para revisão. Atuais: `Manuel Leal Madruga ~ Manuel Madruga` (2013-2017) e `Fábio Silva ~ Fábio Ávila` (2021-2025).

Fornecedores: agrupados por grafia. O campo `grafias` conta as variantes fundidas. **Empresas do mesmo grupo com nomes diferentes contam separadas** — limitação declarada no próprio export.

## 15.6 Qualidade dos dados

| Aspeto | Situação |
|---|---|
| OCR | Erros reais observados; sem métrica de confiança |
| `deliberacao.assunto` | **NULL em 215 de 723 (29,7%)** — depende da fórmula "Posta à votação, …" aparecer na ata |
| Voto nominal antes de agosto de 2015 | Não existe na fonte. Os mandatos 1997-2001, 2001-2005, 2005-2009 e 2009-2013 têm sessões e deliberações mas **0 votos nominais**; o mandato 2013-2017 só os tem a partir de 2015-08-06 |
| Composição do órgão | `V` (ata de instalação) em 4 mandatos: 2001-2005, 2009-2013, 2013-2017, 2017-2021. `~` (inferida das cláusulas de voto) em 2021-2025 e 2025-2029 — **não existe ata de instalação publicada para estes** |
| Bancada dos presidentes de junta | `?` em 2009-2013 e 2013-2017: o cabeçalho da ata diz "eleitos nas listas do Partido Social Democrata **e** do Partido Socialista" sem dizer quem é de qual. **Deliberadamente não atribuída** |
| Siglas eleitorais | 4 sem normalização (`pm`, `GRUPO CIDADÃOS`, `B.E.`, `MMSR`) |
| `CONC[].dr` | **Demonstrativo, não real.** Assinalado na página |

## 15.7 Processos manuais

1. Invocar cada comando do pipeline
2. Descarregar os ficheiros AIQGP da DGAL
3. Injetar cada dataset no `index.html`
4. Escrever/atualizar `CONC` e `FMETA` à mão no JavaScript
5. Rever os nomes ambíguos sinalizados
6. Publicar (git push)

## 15.8 Componentes frágeis

| Componente | Fragilidade |
|---|---|
| `discover.py` | Depende dos IDs de página do site municipal. Uma reestruturação do site parte tudo |
| `connectors/eleicoes.py` | Depende de um comportamento **não documentado** de uma SPA (`useStaticFiles`). Já quebrou para 2025 |
| `build/fetch_contratos.py` | Depende de `version=139.0.0` no POST do BASE. **Estado não reconfirmado** |
| `assembleia.py` | ~30 expressões regulares contra prosa OCR. Cada melhoria exige reprocessar tudo |
| `inject.py` | Splice por índice de string num ficheiro de 677 KB |
| OCR | `OMP_THREAD_LIMIT=1` é obrigatório; sem ele uma página passou de 3 s para >600 s |

## 15.9 Dependências críticas

- `tesseract` + `tesseract-lang` (por) — sem isto, 107 das 110 atas são ilegíveis
- `poppler-utils` — `pdftoppm`, `pdftotext`, `pdfinfo`
- Disponibilidade e estabilidade de estrutura de: `cm-lajesdopico.pt`, `www.ine.pt`, `json.geoapi.pt`, `www.eleicoes.mai.gov.pt`, `www.base.gov.pt`
- Google Fonts (degradação graciosa: há stacks de fallback declaradas)

---

# 16. Preparação para um módulo de Social Intelligence

Avaliação do que a arquitetura **atual** suporta. Nenhuma solução é proposta aqui.

| | Capacidade | Classificação | Porquê |
|---|---|---|---|
| **A** | **Trends** — atenção a subir/descer | **Exige nova infraestrutura** | Não existe qualquer sinal de atenção. Nenhum corpus de notícias, redes sociais ou pesquisa. Sem taxonomia temática (§9), sem tabela de menções, sem índice de texto. O único proxy possível hoje seria "frequência de um tema nas deliberações", o que exige primeiro classificar 723 deliberações cujo `assunto` é NULL em 29,7% dos casos |
| **B** | **People Intelligence** (qualquer pessoa) | **Exige alteração** | `membro` tem `municipio`, `orgao` e `mandato` como `NOT NULL` — inutilizável para quem não tem mandato. Não há identificador estável de pessoa; a identidade é uma função calculada em runtime (`chave_pessoa`). Exige entidade `pessoa` nova e transformar `membro` em tabela de afiliação. **Mas:** o material para os 162 membros-mandato existentes é sólido e reutilizável |
| **C** | **Organization Intelligence** | **Exige nova infraestrutura** | Não existe entidade de organização. Empresas são strings truncadas a 34 chars sem NIF; partidos são strings; associações, clubes, universidades e meios de comunicação não existem em forma alguma. A rede de administrações partilhadas está bloqueada pelo anti-bot do Registo Comercial |
| **D** | **Influence Graph** | **Parcialmente suportado** (uma aresta de seis) | `Pessoa↔Pessoa`: inferível por co-votação (`voto` partilhando `deliberacao_id`) — 5 155 votos nominais dão um grafo de co-votação real e denso. `Pessoa↔Organização`: só pessoa↔bancada, como string. `Pessoa↔Tema`, `Organização↔Tema`: impossível, sem temas. `Pessoa↔Evento`: `presenca` é exatamente isto (970 arestas pessoa–sessão). `Pessoa↔Conteúdo`: indireta via `presenca→sessao→documento`. **Não existe tabela de arestas nem grafo persistido** |
| **E** | **Influence by Topic** | **Exige nova infraestrutura** | Depende inteiramente de (A) e da taxonomia. Zero suporte |
| **F** | **Emerging People** | **Parcialmente suportado**, num sentido estreito | É calculável hoje para membros de assembleia: primeira aparição (`MIN(sessao.data)`), trajetória de votos e presenças, mudança de pivotalidade entre mandatos. **Não é calculável** para qualquer pessoa fora do órgão, nem em janelas curtas (as sessões são ~4/ano, o que dá resolução trimestral no melhor caso) |
| **G** | **Emerging Topics** | **Exige nova infraestrutura** | Sem temas e sem corpus de atenção. `deliberacao.assunto` é o único texto temático e é NULL em 29,7% dos casos |
| **H** | **Narrative Tracking** | **Exige nova infraestrutura** | Requer corpus longitudinal de texto público (imprensa, redes), classificação e comparação diacrónica. Nada disto existe. O corpus de atas é longitudinal (1997–2026) mas é linguagem administrativa ritual, não narrativa pública |
| **I** | **Sentiment / Stance** | **Stance parcialmente suportado; sentiment exige nova infraestrutura** | **Stance existe e é excelente**: o sentido de voto por membro numa deliberação identificada é *stance declarado e verificável* — 5 155 pontos. É stance institucional, não de opinião pública. **Sentiment não existe** e sobre pessoas privadas não deve existir (§17) |
| **J** | **Network Analysis** (centralidades, PageRank, comunidades) | **Exige alteração** | Os dados para construir um grafo de co-votação existem e são densos, mas: não há tabela de arestas, não há biblioteca de grafos (zero dependências), e o SQL escrito à mão não faz travessias. Calculável com `networkx` sobre um export — **mas nada disso está implementado.** Uma community detection sobre co-votação de 2021-2025 é executável hoje com um script novo |
| **K** | **Predictive Models** | **Exige nova infraestrutura** | Não há framework de ML, não há features persistidas, não há target. **E há um limite estatístico que não é de engenharia:** com 2 atos eleitorais (2017, 2021) em 20 territórios e 4 469 inscritos no maior concelho, previsão eleitoral não tem base amostral. Tendências de comportamento de voto institucional são modeláveis (5 155 observações); notoriedade, aceitação e procura não têm sinal de entrada |
| **L** | **B2B Intelligence** | **Exige nova infraestrutura** | O único ativo reutilizável é a forense de contratação (735 contratos, quota/HHI/persistência/amplitude), que serve análise de concorrência num nicho estreito: fornecedores de 3 municípios. Reviews, dados de mercado, inteligência de clientes, reputação, deteção de problemas e procura: zero fontes, zero modelo |

## 16.1 Resumo da avaliação

| Classificação | Capacidades |
|---|---|
| **Já suportado** | Nenhuma na íntegra |
| **Parcialmente suportado** | D (Influence Graph — 2 de 6 arestas), F (Emerging People — só membros de órgão), I (apenas stance institucional) |
| **Exige alteração** | B (People Intelligence), J (Network Analysis) |
| **Exige nova infraestrutura** | A, C, E, G, H, K, L |

## 16.2 O que já existe e não deve ser reconstruído

1. **Pipeline documental com proveniência** — descoberta, `sha256`, OCR com motor registado, `url_origem` por documento
2. **5 155 votos nominais** com data, pessoa, bancada e sentido (2015-08-06 → 2026-02-23) — stance institucional verificável
3. **970 presenças** pessoa–sessão (1997–2026)
4. **723 deliberações** com resultado, modo e contagens
5. **120 resultados eleitorais** com inscritos como denominador, ao nível da freguesia
6. **735 contratos** com valor, procedimento, data e objeto
7. **O padrão de fiabilidade `V`/`~`/`?`** aplicado consistentemente e visível na interface
8. **A disciplina de não adivinhar** — bancadas ambíguas ficam `?`, siglas não reconhecidas ficam NULL, nomes incompatíveis não são fundidos, e cada caso é listado para revisão

---

# 17. Privacidade e proveniência

## 17.1 Natureza dos dados

| Categoria | Estado |
|---|---|
| **Público por publicação oficial** | Atas municipais (site da câmara), resultados eleitorais (DGAI), contratos (Portal BASE), indicadores (INE/geoapi), finanças (DGAL/dados.gov.pt) |
| **Dados pessoais de titulares de cargos, no exercício público** | Nome, bancada, presenças, sentido de voto. Tratados como escrutínio de função pública |
| **Dados pessoais sensíveis PRESENTES na fonte** | ⚠️ As **atas de instalação** identificam cada eleito com **número de Cartão de Cidadão / Bilhete de Identidade, morada completa, idade, estado civil e profissão** |

## 17.2 Como os dados pessoais sensíveis são tratados

O `parse_instalacao()` extrai **apenas nome + lista**. O mecanismo é duplo:

1. **Corte antes do dado pessoal:** o nome é ancorado no estado civil que o segue (`_NOME_ANCORADO`), pelo que o corte cai exatamente onde os dados pessoais começam
2. **Rede de segurança que falha em vez de publicar:** `_PESSOAL` (regex para `Cartão de Cidadão`, `Bilhete de Identidade`, `contribuinte`, `\d{7,9}`, `residente na`, `freguesia de`, estados civis) é aplicada ao que vai sair e **levanta `AssertionError`** se algo escapar. Disparou realmente durante o desenvolvimento (o OCR trocou uma vírgula por ponto e colou "Divorciado" a um nome) — funcionou como pretendido

**MAS — risco residual documentado:** o texto OCR integral, **incluindo números de Cartão de Cidadão e moradas**, está gravado em `pagina.texto` na base de dados. Mitigação atual: `data/` é gitignored, logo a base **nunca foi versionada nem publicada**. Não há redação no armazenamento, apenas na exportação. **Qualquer módulo futuro que exponha `pagina.texto` (pesquisa de texto integral, RAG, embeddings) reexpõe estes dados.** É a consideração de privacidade mais importante deste dossier.

## 17.3 Proveniência

| Mecanismo | Estado | Onde |
|---|---|---|
| URL de origem por documento | `IMPLEMENTADO` | `documento.url_origem` (UNIQUE) |
| URL de origem por sessão | `IMPLEMENTADO` | `sessao.fonte_url` |
| URL de origem por resultado eleitoral | `IMPLEMENTADO` | `eleicao_territorio.fonte` (URL exato do ficheiro) |
| Fonte por indicador | `IMPLEMENTADO` | `indicador.fonte` + `indicador.url`; no JSON, campo `fonte` |
| Hash de integridade | `IMPLEMENTADO` | `documento.sha256` |
| Motor de extração | `IMPLEMENTADO` | `pagina.motor_ocr`, `sessao.motor_ocr` |
| Excerto-fonte por deliberação | `IMPLEMENTADO` | `deliberacao.excerto` (até 400 chars) |
| Grafia exata lida por presença | `IMPLEMENTADO` | `presenca.nome_como_lido` |
| Marca de fiabilidade | `IMPLEMENTADO` | `sessao.confianca`, `membro.partido_rel`, `indicador.rel`, `mandatos_rel` — escala `V`/`~`/`?` visível na interface |
| Timestamp de recolha | `PARCIAL` | `documento.descoberto_em` e `atualizado_em` existem. **`indicador.recolhido_em` existe mas a tabela está vazia.** `sessao`, `voto`, `eleicao_territorio` **não têm timestamp de recolha** |
| Histórico / versionamento | `PARCIAL` | Só via git nos datasets exportados. Na base, tudo é sobrescrito |
| Possibilidade de correção | `PARCIAL` | Reprocessar corrige, mas não há mecanismo de correção manual persistente: `refazer_assembleia()` apagaria qualquer correção à mão |
| Sistema de auditoria | `PARCIAL` | `execucao` registra corridas (sem duração). Não há trilho de auditoria por registo, nem autor, nem motivo de alteração |
| Listas de revisão | `IMPLEMENTADO` | `possiveis_duplicados()`, `colisoes_pessoa()`, `pessoas_ambiguas()`, siglas sem `sigla_norm`, `documento.needs_review`. **Exportadas em `lajes_influencia.json.revisao`** |

## 17.4 Princípios que o sistema já aplica e devem ser preservados

Estão vertidos em código e em texto visível na interface:

1. **Nunca inventar.** Cada valor tem ano, fonte e fiabilidade
2. **Não adivinhar quando a fonte é ambígua.** Bancada de presidentes de junta em cabeçalho com duas listas → `?`. Sigla eleitoral não interpretável → `sigla_norm` NULL com o rótulo original guardado
3. **Não fundir identidades sem base.** Nomes incompatíveis não são fundidos; os casos são listados
4. **OCR é sinal automático.** A interface diz, no separador Assembleia: *"sinal automático, rever antes de citar"*
5. **Sinais forenses são para escrutínio, não prova.** Ajuste direto e clustering junto a limiares são mostrados a laranja e **excluídos do índice de influência**
6. **Não pontuar pessoas privadas.** O separador Influência declara-o explicitamente: *"Este separador mede peso documentado na decisão pública — não popularidade nem aceitação. (…) Sobre pessoas privadas não se pontua nada."*
7. **Pesos editoriais à vista.** `PESOS` está no código e as parcelas são mostradas ao lado do total

---

# 18. Entrega final

## A. Executive Summary

**Pico Cívico** é uma plataforma de fiscalização municipal da ilha do Pico (Açores). Consiste em **dois repositórios e nenhuma infraestrutura**: `lajes-acervo`, um pipeline de ingestão em Python (3 720 linhas, **zero dependências externas**, apenas stdlib + tesseract + poppler), e `pico-civico`, uma **página HTML única de 677 KB** que traz todos os dados embutidos inline em cinco blocos `<script>var`.

**O que existe, e é sólido:** um acervo de 110 atas da Assembleia Municipal das Lajes do Pico (1998–2026, 1 526 páginas OCRizadas, 2,96 M caracteres), do qual foram extraídas por expressões regulares **109 sessões, 723 deliberações, 970 presenças e 5 781 votos — dos quais 5 155 são voto nominal por membro com bancada identificada, de agosto de 2015 a fevereiro de 2026**. Mais 120 resultados eleitorais ao nível da freguesia (2017 e 2021, com inscritos como denominador), 735 contratos públicos (2021–2026, 3 concelhos) e 38 indicadores estatísticos. Tudo com URL de origem, `sha256`, motor de extração registado e uma marca de fiabilidade `V`/`~`/`?` visível na interface.

**O que não existe:** backend, API, autenticação, alojamento aplicacional, agendador, fila de processamento, índice de texto, base vetorial, embeddings, e **qualquer utilização de IA ou LLM** — verificado por busca exaustiva. Não existe entidade genérica `Pessoa` (só `membro`, que é *pessoa-num-mandato* com `municipio`/`orgao`/`mandato` como `NOT NULL`), não existe entidade `Organização` (empresas são strings sem NIF), não existe taxonomia temática, e não existe **uma única linha** de conteúdo de notícias, imprensa ou redes sociais.

**Consequência para um módulo de Social Intelligence:** o substrato de atenção pública — corpus longitudinal de texto, temas, menções, sentimento — **está inteiramente ausente**. Das doze capacidades avaliadas (§16), nenhuma está plenamente suportada, três estão parcialmente suportadas, duas exigem alteração de esquema e sete exigem infraestrutura nova. Em contrapartida, o sistema traz duas coisas que raramente se encontram e que **não devem ser reconstruídas**: um grafo de co-votação real e denso (5 155 arestas pessoa–deliberação com sentido declarado, que é *stance* verificável e não inferido), e uma disciplina de proveniência e de não-adivinhação já vertida em código — bancadas ambíguas ficam `?`, siglas não interpretáveis ficam NULL com o rótulo original guardado, identidades incompatíveis não são fundidas, e cada caso duvidoso é exportado numa lista de revisão.

**Aviso de privacidade que condiciona o desenho:** as atas de instalação contêm números de Cartão de Cidadão, moradas, idades e estados civis. A exportação está protegida por um extractor que **falha em vez de publicar**, mas o **texto OCR integral, com esses dados, está gravado em `pagina.texto`**. A base nunca foi versionada (é gitignored). Qualquer funcionalidade que exponha `pagina.texto` — pesquisa de texto integral, RAG, embeddings — **reexpõe dados pessoais sensíveis**.

## B. Arquitetura atual

```text
┌─────────────────────────── FONTES EXTERNAS ────────────────────────────┐
│ cm-lajesdopico.pt   HTML paginado + 110 PDF de atas    [IMPLEMENTADO]  │
│ www.ine.pt/pindica  JSON, 9 varcd municipais           [IMPLEMENTADO]  │
│ json.geoapi.pt      JSON, Censos 2011/2021             [IMPLEMENTADO]  │
│ eleicoes.mai.gov.pt JSON estático de SPA (só 2021)     [PARCIAL]       │
│ base.gov.pt         POST, 735 contratos                [NÃO RECONFIRM.]│
│ dados.gov.pt/DGAL   AIQGP 2017-19                      [MANUAL]        │
│ notícias · redes sociais · Google Trends               [INEXISTENTE]   │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │  urllib + retry/backoff (http.py)
                                 │  ⚠ SEM AGENDADOR — tudo invocado à mão
                                 ▼
┌──────────────────────── INGESTÃO (Python, stdlib) ─────────────────────┐
│ discover.py   HTMLParser → âncoras de PDF → UNIQUE(url_origem)        │
│ metadata.py   rótulo da listagem → data/tipo/nº  (puro, testável)     │
│ download.py   GET + sha256 + pdfinfo/pdftotext → tem_camada_texto     │
│ ocr.py        pdftotext │ pdftoppm 300dpi + tesseract por --psm 3     │
│               (OMP_THREAD_LIMIT=1 obrigatório) · 4 workers            │
└────────────────────────────────┬───────────────────────────────────────┘
                                 ▼
┌──────────────────── PROCESSAMENTO (regex + difflib) ───────────────────┐
│ assembleia.py  atas de instalação → composição OFICIAL                │
│                cláusulas de voto  → roster (quem é membro, que banc.) │
│                reconciliação      → identidade = nome oficial          │
│                presenças casadas contra o roster                       │
│                deliberações + voto nominal por membro                  │
│ influencia.py  assiduidade · dissidência · pivotalidade · longevidade  │
│                quota · persistência · amplitude (empresas)             │
│ forensics_all.py  HHI · ascensões · clustering · rede cross-concelho   │
│               ⚠ ZERO IA / ZERO LLM / ZERO EMBEDDINGS                   │
└────────────────────────────────┬───────────────────────────────────────┘
                                 ▼
┌──────── BASE DE DADOS: SQLite, 12 tabelas, ficheiro local ────────────┐
│ documento 110 · pagina 1 526 · bloco 0(planeado) · indicador 0(vazia) │
│ membro 162 · sessao 109 · presenca 970 · deliberacao 723 · voto 5 781 │
│ eleicao_territorio 120 · eleicao_lista 312 · execucao 6               │
│ ⚠ GITIGNORED · NÃO SERVIDA · SEM ÍNDICE DE TEXTO · SEM FK IMPOSTAS    │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │  --export  (funções *_export em db.py)
                                 ▼
┌────────── DATASETS JSON — build/datasets/ (versionados em git) ────────┐
│ 8 ficheiros, ~1,04 MB. É AQUI que a plataforma lê, não na base.       │
└────────────────────────────────┬───────────────────────────────────────┘
                                 │  build/inject.py — splice ASCII-safe
                                 │  ensure_ascii=True + "</"→"<\/" + valida
                                 ▼
┌────────── FRONTEND: pico-civico/index.html (677 KB, 1 239 linhas) ─────┐
│ HTML + CSS + JS ES5 · sem build · sem framework · sem lib de gráficos │
│ Dados INLINE: GEO 48K · DEEP 219K · FIN 2K · IND 7K · INFL 100K       │
│               ASSEM 210K  = 79% do ficheiro                            │
│ Mapa SVG à mão · 7 separadores · filtro de texto no cliente           │
│ ⚠ SEM BACKEND · SEM API · SEM AUTENTICAÇÃO · SEM ROTAS · SEM ESTADO   │
└────────────────────────────────────────────────────────────────────────┘
```

## C. Data Model — entidades e relações

```text
                        ┌──────────────┐
                        │  documento   │  110   UNIQUE(url_origem)
                        │  sha256      │        estado: descoberto→
                        │  url_origem  │        descarregado→ocr_feito
                        └──┬────────┬──┘
                 1:N       │        │      1:N
              ┌────────────┘        └───────────────┐
              ▼                                     ▼
      ┌──────────────┐                      ┌──────────────┐
      │   pagina     │ 1 526                │    bloco     │ 0  PLANEADO
      │ texto (OCR)  │ ⚠ contém dados       │ embedding    │ nunca escrito
      │ motor_ocr    │   pessoais sensíveis │   BLOB       │
      └──────────────┘                      └──────────────┘
              │
              │  assembleia.processar()  — regex, não IA
              ▼
      ┌──────────────┐         ┌──────────────────────────────┐
      │   sessao     │ 109 ────┤ UNIQUE(municipio,orgao,      │
      │ data,mandato │         │        data,tipo)            │
      │ tipo,confianca│        └──────────────────────────────┘
      └───┬──────┬───┘
     1:N  │      │  1:N
    ┌─────┘      └──────────┐
    ▼                       ▼
┌──────────┐          ┌──────────────┐
│ presenca │ 970      │ deliberacao  │ 723
│ estado   │          │ assunto (⚠ NULL em 215/723)
│nome_como_│          │ resultado,modo,n_favor/contra/abstencao
│  lido    │          │ excerto (texto-fonte)
└────┬─────┘          └───────┬──────┘
     │  N:1                   │  1:N
     ▼                        ▼
┌──────────────────┐    ┌──────────────┐
│     membro       │◄───┤     voto     │ 5 781
│ 162              │N:1 │ sentido      │  ├─ granularidade='membro'  5 155
│ ⚠ É PESSOA-NUM-  │    │ granularidade│  └─ granularidade='bancada'   626
│   MANDATO, não   │    │ partido      │  membro_id NULL ⇔ voto de bancada
│   pessoa         │    └──────────────┘
│ UNIQUE(municipio,│
│  orgao,mandato,  │    Identidade entre mandatos:
│  nome_norm)      │    influencia.chave_pessoa() = primeiro+último termo
│ partido (string) │    ⚠ CALCULADA EM RUNTIME, NÃO PERSISTIDA
│ partido_rel V/~/?│    ⚠ 2 chaves ambíguas conhecidas
└──────────────────┘

┌───────────────────────┐        ┌──────────────┐
│  eleicao_territorio   │ 120 ───┤ eleicao_lista│ 312
│ ano(2017,2021)        │  1:N   │ sigla        │ ⚠ sigla_norm NULL em 4
│ territorio LOCAL-DDMMFF│       │ votos,pct    │   siglas não interpretáveis
│ nivel: concelho|fregues│       │ mandatos     │
│ eleicao: CM|AM|AF     │        └──────────────┘
│ inscritos ← DENOMINADOR│
│ fonte (URL exato)     │        ⚠ SEM FK para membro nem sessao.
└───────────────────────┘           A ligação é só conceptual, via
                                    catalog.MANDATOS (lista Python)

┌──────────────┐   ┌──────────────┐
│  indicador   │ 0 │   execucao   │ 6
│ ⚠ VAZIA — os │   │ etapa,estado │ ⚠ iniciado_em == terminado_em
│ dados estão  │   │ itens_*      │   (não mede duração)
│ só no JSON   │   └──────────────┘
└──────────────┘

SÓ EM JSON, NUNCA NO SQLITE:
  contratos_raw.json  735 contratos (id, contracted ⚠ sem NIF, contracting,
                      contractingProcedureType, initialContractualPrice,
                      objectBriefDescription, publicationDate, signingDate)
  deep_all.json       forense por concelho
  financas.json       DGAL 2017-19, 3 concelhos ⚠ chaves lajes_do_pico /
                      sao_roque_do_pico (3.ª convenção de nomes)

SÓ EM JAVASCRIPT, ESCRITO À MÃO (não reproduzível):
  CONC   presidente, executivo, câmara, assembleia, pop, área, densidade
         + dr = DADOS DEMONSTRATIVOS, assinalados como tal
  FMETA  17 freguesias: concelho, população, descrição

NÃO EXISTEM: Person · Organization · Partido · Empresa · Município ·
             Freguesia · Tema · Fonte · Candidato · Intervenção · Notícia ·
             Menção · Aresta de grafo
```

## D. Technology Stack

| Camada | Tecnologia | Versão | Função | Estado |
|---|---|---|---|---|
| Frontend | HTML5 + CSS3 | — | Interface completa | `IMPLEMENTADO` |
| Frontend | JavaScript ES5 | — | Lógica, mapa, gráficos | `IMPLEMENTADO` |
| Frontend | Google Fonts | — | Space Grotesk, IBM Plex Mono | `IMPLEMENTADO` |
| Frontend framework | — | — | — | `INEXISTENTE` |
| Lib. de gráficos | — | — | SVG e `<div>` à mão | `INEXISTENTE` |
| Build / bundler | — | — | — | `INEXISTENTE` (deliberado) |
| Backend | — | — | — | `INEXISTENTE` |
| API | — | — | — | `INEXISTENTE` |
| Autenticação | — | — | — | `INEXISTENTE` |
| Ingestão | Python | 3.9+ (corrido em 3.11) | Todo o pipeline | `IMPLEMENTADO` |
| HTTP | `urllib.request` | stdlib | GET com retry/backoff | `IMPLEMENTADO` |
| Parsing HTML | `html.parser` | stdlib | Descoberta de PDF | `IMPLEMENTADO` |
| Paralelismo | `ThreadPoolExecutor` | stdlib | 4 workers (descarga, OCR) | `IMPLEMENTADO` |
| Semelhança de strings | `difflib` | stdlib | Fusão de variantes de OCR | `IMPLEMENTADO` |
| OCR | tesseract | 5.3.4, lang `por` | Texto de PDF digitalizado | `IMPLEMENTADO` |
| PDF | poppler-utils | — | `pdftoppm`, `pdftotext`, `pdfinfo` | `IMPLEMENTADO` |
| Base de dados | SQLite | 3 | 12 tabelas | `IMPLEMENTADO` |
| ORM | — | — | SQL à mão em `db.py` | `INEXISTENTE` |
| Migrações | — | — | `refazer_assembleia()` (DROP+recria) | `INEXISTENTE` |
| Pesquisa | — | — | FTS5 planeado; filtro no cliente | `PLANEADO` |
| Base vetorial | — | — | — | `INEXISTENTE` |
| Embeddings | — | — | Coluna `bloco.embedding` vazia | `INEXISTENTE` |
| IA / LLM | — | — | — | `INEXISTENTE` |
| Agendador | — | — | Invocação manual | `INEXISTENTE` |
| Filas | — | — | — | `INEXISTENTE` |
| Logging | tabela `execucao` | — | 1 linha por corrida, sem duração | `PARCIAL` |
| Monitorização / APM | — | — | — | `INEXISTENTE` |
| Testes | pytest | — | 49 testes | `IMPLEMENTADO` |
| CI/CD | — | — | Sem `.github/workflows` | `INEXISTENTE` |
| Containers | — | — | — | `INEXISTENTE` |
| Alojamento | — | — | Ficheiro local + git | `INEXISTENTE` |
| Controlo de versões | git + GitHub | — | 2 repos privados | `IMPLEMENTADO` |

## E. Data Sources

| # | Fonte | Acesso | Método | Automatizado | Volume atual | Histórico | Estado |
|---|---|---|---|---|---|---|---|
| F1 | Atas AM Lajes (`cm-lajesdopico.pt`) | HTML + PDF | `discover`+`download`+`ocr` | Comando manual | 110 PDF · 1 526 pág. | 1997-12-31 → 2026-02-23 | `IMPLEMENTADO` |
| F2 | INE `pindica.jsp` | JSON API | `connectors/ine.py` | Comando manual | 9 varcd | anual | `IMPLEMENTADO` · 200 OK |
| F3 | geoapi.pt (Censos) | JSON API | `connectors/geoapi.py` | Comando manual | Censos 2011+2021 | 2011, 2021 | `IMPLEMENTADO` · 200 OK |
| F4 | DGAI eleições | JSON estático de SPA | `connectors/eleicoes.py` | Comando manual | 120 territórios · 312 listas | **só 2017+2021** | `PARCIAL` · 2025 bloqueado |
| F5 | Portal BASE | POST | `build/fetch_contratos.py` | Comando manual | 735 contratos | 2021 → 2026 | ⚠ **não reconfirmado** |
| F6 | DGAL AIQGP | Ficheiro | **manual** | Não | 3 concelhos | **só 2017–2019** | `PARCIAL` |
| — | Atas da Câmara Municipal | HTML + PDF | 29 IDs mapeados | Não corrido | 0 | 1998–2026 disponível | `PLANEADO` |
| — | Documentos financeiros municipais | HTML + PDF | `discover_financas()` pronto | Não corrido | 0 | — | `PLANEADO` |
| — | Notícias / imprensa / RSS | — | — | — | 0 | — | `INEXISTENTE` |
| — | Redes sociais | — | — | — | 0 | — | `INEXISTENTE` |
| — | Google Trends | — | — | — | 0 | — | `INEXISTENTE` |
| — | Parlamento / ALRAA | — | — | — | 0 | — | `INEXISTENTE` |
| — | SREA (conector) | — | citado como texto de `fonte` | — | 0 | — | `INEXISTENTE` · 403 no teste |
| — | Registo Comercial | HTML | anti-bot `NoBotExtender` | **decisão de não contornar** | 0 | — | `BLOQUEADO` |

## F. AI Capabilities — o que existe atualmente

**Nenhuma capacidade de IA generativa ou de NLP moderno.** Verificado por busca exaustiva no código.

| Capacidade | Implementação real | Não é |
|---|---|---|
| OCR | tesseract 5.3.4 LSTM, `por`, 300 dpi, `--psm 3` | Não é LLM; confiança não capturada |
| "Reconhecimento de pessoas" | ~30 expressões regulares + `difflib.SequenceMatcher` (corte 0.90–0.92) | Não é NER |
| "Identificação de organizações" | Regex de bancadas + normalização de strings de empresa | Não é entity linking; sem NIF |
| "Extração de relações" | Regex sobre a fraseologia ritual das atas | Não é relation extraction |
| Deduplicação | `UNIQUE` no SQL + `canonicaliza()` conservadora | Não é record linkage probabilístico |
| Classificação temática | — | `INEXISTENTE` |
| Sumarização | — | `INEXISTENTE` |
| Sentimento | — | `INEXISTENTE` |
| Embeddings / pesquisa semântica / RAG | — | `INEXISTENTE` |

## G. Gaps para Social Intelligence

| Prioridade | Lacuna | O que falta concretamente |
|---|---|---|
| **1** | **Corpus de atenção pública** | Zero conteúdo de imprensa, redes sociais ou pesquisa. É a lacuna fundadora: sem isto, Trends, Emerging Topics, Narrative Tracking e Sentiment não têm entrada |
| **2** | **Entidade `pessoa` com identificador estável** | `membro` é pessoa-num-mandato com 3 campos `NOT NULL` que excluem não-políticos. A identidade é uma função em runtime |
| **3** | **Entidade `organizacao`** | Não existe. Empresas são strings truncadas sem NIF |
| **4** | **Taxonomia temática** | Não existe. Duas listas de strings divergentes, aplicadas só a indicadores |
| **5** | **Camada de serviço** | Sem backend/API não há como servir consultas dinâmicas nem alimentar um segundo consumidor |
| **6** | **Índice de texto** | 2,96 M caracteres em `pagina.texto` sem índice. FTS5 é stub. ⚠ Indexar reexpõe dados pessoais (§17.2) |
| **7** | **Tabela de arestas / grafo** | Os dados de co-votação existem (5 155 votos) mas não há grafo persistido nem biblioteca |
| **8** | **Modelo temporal de menções** | Não existe o conceito de menção nem série de atenção |
| **9** | **Agendamento e observabilidade** | Sem agendador, sem duração de etapas, sem alertas, sem retry queue |
| **10** | **Contratos em base de dados** | Só em JSON — impede cruzar decisão↔contrato em SQL |
| **11** | **Redação no armazenamento** | Dados pessoais sensíveis em `pagina.texto`; a proteção é só na exportação |
| **12** | **Framework de ML e features persistidas** | Nada. E limite estatístico real: 2 atos eleitorais em 20 territórios não sustentam previsão |

## H. Integration Readiness Score

### **29 / 100**

Pontuação composta. Os pesos estão à vista e a aritmética é verificável — a mesma disciplina que o projeto aplica ao seu próprio índice de influência. Quem discorde dos pesos pode ler as parcelas.

| Dimensão | Peso | Nota (0–10) | Pontos | Justificação |
|---|---|---|---|---|
| Fontes públicas e proveniência | 10% | 8 | 8.0 | URL de origem, `sha256`, motor de extração e escala `V`/`~`/`?` aplicados de forma consistente |
| Modelo institucional (votos, sessões, presenças) | 10% | 8 | 8.0 | Normalizado, com chaves de unicidade pensadas; 5 155 votos nominais com data, pessoa, bancada e sentido |
| Dados eleitorais | 6% | 5 | 3.0 | Nível de freguesia com inscritos como denominador, mas só 2 atos eleitorais e 2025 bloqueado |
| Qualidade e coerência dos dados | 5% | 5 | 2.5 | Boa disciplina de fiabilidade; mas 3 convenções de nome de concelho, `indicador` vazia, `assunto` NULL em 29,7% |
| Entidade genérica Pessoa | 10% | 1 | 1.0 | Não existe. `membro` é pessoa-num-mandato com `municipio`/`orgao`/`mandato` `NOT NULL` bloqueantes; identidade calculada em runtime |
| Entidade genérica Organização | 8% | 0 | 0.0 | Não existe em forma alguma. Empresas são strings truncadas a 34 chars, sem NIF |
| Taxonomia temática | 7% | 0 | 0.0 | Não existe. Duas listas de strings divergentes, aplicadas só a indicadores |
| Corpus de atenção pública | 11% | 0 | 0.0 | Zero notícias, zero redes sociais, zero trends. É a lacuna fundadora |
| Texto indexado / pesquisa | 6% | 1 | 0.6 | 2,96 M caracteres em `pagina.texto` sem índice. FTS5 é stub; só há filtro no cliente |
| IA / NLP / embeddings | 6% | 0 | 0.0 | Zero. Verificado por busca exaustiva no código |
| Camada de serviço (API/backend) | 6% | 0 | 0.0 | Zero. O SQLite é artefacto de build local; a plataforma lê JSON congelado |
| Grafo de relações | 5% | 2 | 1.0 | Dados de co-votação e de presença existem e são densos; grafo persistido e biblioteca não existem |
| Operacionalização (agendamento, observabilidade) | 3% | 2 | 0.6 | `execucao` sem duração, sem alertas, sem retry queue; tudo invocado à mão |
| Testes e reprodutibilidade | 3% | 6 | 1.8 | 49 testes com excertos reais de atas; mas `CONC`/`FMETA` à mão e `indicador` vazia |
| Privacidade e conformidade | 4% | 6 | 2.4 | Redação na exportação que falha-em-vez-de-publicar; mas dados sensíveis persistidos em `pagina.texto` |
| **Total** | **100%** | — | **28.9** | Pontos = peso × nota ÷ 10 |

**Leitura do número.** 29/100 **não é uma crítica à qualidade do que existe.** O que existe é limpo, testado e com proveniência acima da média para um projeto deste porte. O número é baixo porque mede *prontidão para Social Intelligence especificamente*, e o sistema atual é um pipeline de forense documental municipal: quase nenhum dos componentes que um módulo de atenção pública exige — pessoas genéricas, organizações, temas, corpus de menções, índice de texto, IA, camada de serviço — foi construído, porque nunca foi o objetivo.

**Onde estão os pontos.** As duas dimensões que pontuam 8/10 (proveniência e modelo institucional) valem 16 dos 29 pontos. É aí que está o ativo a preservar. As quatro dimensões que pontuam 0 (organização, taxonomia, corpus de atenção, IA) somam 32% do peso e 0 pontos — é aí que está o trabalho.

**Um score alto exigiria que o sistema já fosse aquilo que se quer construir.** O valor prático deste dossier não está no 29: está em §16.2 (o que não deve ser reconstruído) e em §G (a fundação a construir).

## I. Ficheiros a analisar primeiro

Por ordem de importância para quem vai desenhar a integração:

| # | Ficheiro | Porquê |
|---|---|---|
| 1 | `lajes-acervo/lajes_acervo/db.py` (487) | O `SCHEMA` completo das 12 tabelas e toda a persistência. **Ler antes de qualquer decisão de modelo** |
| 2 | `lajes-acervo/lajes_acervo/assembleia.py` (869) | O módulo central. Contém a arquitetura de extração (roster → reconciliação → presenças casadas) e as decisões de não-adivinhação que devem ser preservadas |
| 3 | `lajes-acervo/build/README.md` | Os gotchas de **todas** as fontes, incluindo as duas lacunas bloqueantes (2025 eleitoral; Registo Comercial anti-bot). Poupa dias de redescoberta |
| 4 | `lajes-acervo/lajes_acervo/influencia.py` (315) | As definições de pivotalidade e dissidência, e a fronteira explícita entre influência e sinais de escrutínio. `PESOS` e `SINAIS_ESCRUTINIO` |
| 5 | `lajes-acervo/tests/test_assembleia.py` (247) | Excertos **reais** de atas usados como especificação. É a melhor documentação do que a fonte parece |
| 6 | `lajes-acervo/lajes_acervo/connectors/eleicoes.py` (152) | O conector DGAI e a documentação exata de por que 2025 falha |
| 7 | `lajes-acervo/lajes_acervo/connectors/catalog.py` (57) | Municípios, `varcd` do INE, as 8 datas de eleições autárquicas e `mandato_de()` |
| 8 | `pico-civico/index.html` (1 239) | A interface e o formato exato dos 5 blocos de dados inline que ela consome |
| 9 | `lajes-acervo/lajes_acervo/cli.py` (243) | Todos os comandos e — importante — quais são stubs |
| 10 | `lajes-acervo/lajes_acervo/config.py` (71) | IDs de página, `LACUNAS` conhecidas, links inválidos conhecidos |
| 11 | `lajes-acervo/build/inject.py` (108) | O padrão de injeção ASCII-safe. Se o novo módulo alimentar a página, tem de o respeitar |
| 12 | `lajes-acervo/lajes_acervo/ocr.py` (121) | O pipeline de OCR e o `OMP_THREAD_LIMIT=1` sem o qual o débito colapsa |
| 13 | `lajes-acervo/build/datasets/lajes_influencia.json` | O formato de saída atual, incluindo o bloco `revisao` |
| 14 | `lajes-acervo/lajes_acervo/index.py` (7) | 7 linhas de docstring que descrevem a intenção de pesquisa/embeddings. **Ponto de entrada natural para o novo módulo** |

## J. Informação em falta — `DESCONHECIDO`

O que não foi possível determinar a partir do projeto:

1. **Termos de uso e licenças** de todas as fontes. Não foi verificado `robots.txt` nem os termos de `cm-lajesdopico.pt`, INE, geoapi.pt, DGAI ou base.gov.pt. O código pratica crawling educado por escolha, não por conformidade verificada
2. **Licença dos repositórios.** Não existe ficheiro `LICENSE` em nenhum dos dois
3. **Estado real do endpoint do Portal BASE.** Devolveu 404 na raiz a partir do ambiente de teste, o que é compatível com restrição de rede desse ambiente. **A confirmar de outra rede**
4. **Frequência de publicação das atas** na origem. Observou-se ≈3 meses de atraso num caso; não há série suficiente para caracterizar
5. **Existe equivalente noutro município** ao registo de voto nominal em ata? Determinaria se o ativo é único ou replicável
6. **Base jurídica** para o tratamento: não há registo de análise RGPD, avaliação de impacto, política de privacidade, nem responsável pelo tratamento identificado
7. **Alojamento e domínio pretendidos.** GitHub Pages é intenção declarada; não está configurado. Não há evidência de conta cloud
8. **Público-alvo operacional e requisitos de acesso.** Não há modelo de utilizadores; não é possível determinar se se prevê acesso autenticado, API pública ou apenas leitura anónima
9. **Orçamento e limites de custo** para serviços externos (LLM, social listening, base vetorial)
10. **Se existe consentimento ou base legal** para tratar conteúdo de redes sociais, que é o pressuposto do módulo pretendido
11. **Cobertura da Câmara Municipal e dos documentos financeiros.** Os IDs estão mapeados mas nunca correram: o volume e a qualidade reais são desconhecidos
12. **Idioma e qualidade do OCR nas atas da Câmara** — nunca processadas
13. **Se `deep_all.json` e `financas.json` são integralmente reproduzíveis** pelos scripts em `build/`, dado que dependem do endpoint do BASE e de descarga manual da DGAL
14. **Volume de tráfego esperado**, que determinaria se um ficheiro estático continua a ser adequado

---

*Documento gerado por levantamento direto do código e dos dados em 2026-09-03. Commits de referência: `pico-civico@e674919`, `lajes-acervo@342b161`.*
