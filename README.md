# Pico Cívico

Plataforma de inteligência cívica e **fiscalização** da ilha do Pico (Açores). Um mapa
interativo, concelho a concelho e freguesia a freguesia, que reúne indicadores sociais,
económicos e políticos e liga **quem governa · o que investe · como estão as contas · o
que muda**. Inspiração de referência: [analisa.pt](https://analisa.pt).

`index.html` é uma página autónoma (sem build). Protótipo.

## O que tem (perfil de concelho, por secções)
- **Visão geral** — KPIs-chave.
- **Indicadores por área** — Demografia, Habitação, Economia, Emprego, Educação, Saúde,
  Segurança/Justiça, Ambiente, Cultura, Território, Governação/Finanças. Cada indicador com
  ano, fonte e fiabilidade.
- **Governação** — presidente + executivo (mandatos por partido), câmara/assembleia.
- **Assembleia** — sessões, presenças e **sentido de voto por bancada**, extraídos das atas
  por OCR (dados que não existem estruturados a nível municipal).
- **Contratos Públicos** — 2021–2026 (base.gov.pt): série anual, maiores fornecedores,
  **Sinais** forenses (ajuste direto, concentração/HHI, ascensões, clustering), Rede
  cross-concelho e tabela pesquisável.
- **Finanças** — dívida, endividamento, prazo de pagamento (DGAL).

## Fontes de dados
Censos 2021 (INE via geoapi.pt) · INE API `pindica.jsp` · SREA-Açores · DGAL (AIQGP,
dados.gov.pt) · Portal BASE / IMPIC (contratação) · atas municipais (OCR).

## Motor de ingestão
Ver o repositório **`lajes-acervo`** — conectores (geoapi, INE), ingestão de atas + OCR,
extração da Assembleia, e catálogo de indicadores. A plataforma consome o que esse pipeline produz.

## Estado
Molde concebido e completo para **Lajes do Pico**; replicar aos outros concelhos é
trocar o município. Valores marcados por fiabilidade; nada é apresentado como facto sem fonte.

Uma iniciativa QuadrosGroup para a comunidade do Pico.
