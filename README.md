# Pico Cívico

Plataforma de inteligência cívica para a **ilha do Pico** (Açores). Um mapa interativo,
concelho a concelho e freguesia a freguesia, que reúne indicadores **sociais, económicos
e políticos** e liga **decisões e investimentos dos executivos** aos **resultados** na comunidade.

Inspiração de referência: [analisa.pt](https://analisa.pt).

> **Estado:** protótipo de interface (v0). O `index.html` é uma página autónoma, sem dependências.

## O que já é real e verificado

- Estrutura administrativa: **3 concelhos, 17 freguesias**.
- Populações **Censos 2021 (INE)** — incl. o facto de a Madalena ter sido o único concelho dos Açores a ganhar população (+4,7%).
- Presidentes de câmara eleitos nas **autárquicas de outubro de 2025**:
  - Madalena — Catarina Manito (PSD)
  - São Roque do Pico — Luís Filipe Macedo da Silva (PSD)
  - Lajes do Pico — Ana Catarina Terra Brum (PS)

## O que ainda é demonstrativo

- Indicadores económicos/sociais e os exemplos de **Decisões → Resultados** (valores ilustrativos).
- Geometria do mapa (silhueta esquemática, não os polígonos reais).

## Roteiro

1. **Mapa real** — polígonos das freguesias via [geoapi.pt](https://geoapi.pt) (concelhos) + OpenStreetMap
   (relations: ilha `1751494`, Madalena `7382438`, Lajes `7389067`, São Roque `7391635`), com a
   [CAOP/DGT](https://dados.gov.pt) como referência oficial.
2. **Indicadores reais** — INE·EPCC (poder de compra), SREA-Açores (turismo/emprego), PORDATA.
   Assinalar o que só existe a nível de concelho.
3. **Decisões → Resultados** — modelar com orçamentos e atas municipais (o diferenciador).
4. **Comparação entre áreas** e presidentes das 17 juntas de freguesia.

## Executar localmente

Basta abrir o `index.html` no browser, ou servir a pasta:

```bash
python3 -m http.server 8000
# http://localhost:8000
```

## Stack

HTML/CSS/JS puro, sem build. Tipografia: Newsreader + Archivo + IBM Plex Mono (Google Fonts).

---

Uma iniciativa **QuadrosGroup** para a comunidade do Pico.
