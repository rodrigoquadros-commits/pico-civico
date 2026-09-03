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
- **Assembleia** — **8 mandatos, 1998–2026**: cobertura do acervo, presenças por membro,
  **sentido de voto por membro e por bancada**, sessões com as respetivas deliberações e
  ligação à ata em PDF. Extraído por OCR das atas — dados que não existem estruturados a
  nível municipal em Portugal. O voto nominal só é registado nas atas a partir do mandato
  2013–2017; a plataforma mostra essa fronteira em vez de a esconder.
- **Contratos Públicos** — 2021–2026 (base.gov.pt): série anual, maiores fornecedores,
  **Sinais** forenses (ajuste direto, concentração/HHI, ascensões, clustering), Rede
  cross-concelho e tabela pesquisável.
- **Influência** — *peso público documentado*, não popularidade. Por titular de cargo:
  assiduidade, voto nominal, dissidência face à própria bancada e **pivotalidade** (em
  quantas maiorias a sua bancada foi decisiva). Por empresa: quota na despesa do concelho,
  persistência, amplitude. E **apoio eleitoral por freguesia** (DGAI) — a única medida de
  apoio popular com denominador válido. Sobre pessoas privadas não se pontua nada.
- **Finanças** — dívida, endividamento, prazo de pagamento (DGAL).

## Fontes de dados
Censos 2021 (INE via geoapi.pt) · INE API `pindica.jsp` · SREA-Açores · DGAL (AIQGP,
dados.gov.pt) · Portal BASE / IMPIC (contratação) · atas municipais (OCR, 110 atas da
Assembleia de 1998 a 2026).

Nada sobre titulares de cargos vai além do exercício público do mandato — presenças,
deliberações e sentido de voto. Os dados pessoais que as atas de instalação contêm
(Cartão de Cidadão, morada, idade, estado civil, profissão) são descartados na ingestão
e nunca chegam à plataforma.

## Motor de ingestão
Ver o repositório **`lajes-acervo`** — conectores (geoapi, INE), ingestão de atas + OCR,
extração da Assembleia, e catálogo de indicadores. A plataforma consome o que esse pipeline produz.

## Estado
Molde concebido e completo para **Lajes do Pico**. Madalena e São Roque do Pico já têm
Contratos e Finanças; faltam-lhes os Indicadores por área e a Assembleia. Valores marcados
por fiabilidade; nada é apresentado como facto sem fonte.

Uma iniciativa QuadrosGroup para a comunidade do Pico.
