# Investigando ocorrências aeronáuticas no Brasil com Python

**Autor:** José Figueredo
*(projeto do desafio "Investigue uma pergunta com Python" do bootcamp)*

Análise exploratória e teste de hipótese sobre acidentes e incidentes graves da aviação civil brasileira, feitos com Pandas, Matplotlib/Seaborn e SciPy.

## A base

| | |
|---|---|
| **Arquivo** | `dados/V_OCORRENCIA_AMPLA.csv` (original, **sem alterações**) |
| **Fonte** | Ocorrências Aeronáuticas, ANAC / CENIPA, Portal Brasileiro de Dados Abertos (dados.gov.br) |
| **Tamanho** | 5.021 linhas × 45 colunas |
| **Período** | 1959 a jun/2024 (a análise temporal usa 2009–2023) |
| **Conteúdo** | Acidentes e incidentes graves: data, local, aeronave, tipo de operação, fase do voo, danos e lesões |

**Peculiaridades tratadas:** cabeçalho extra na 1ª linha (`skiprows=1`), separador `;`, UTF-8 com BOM, ausentes escritos como `"null"`, coordenadas com vírgula decimal, 10 linhas 100% duplicadas e números de ocorrência repetidos.

**Granularidade (descoberta na limpeza):** cada linha é **1 tipo de ocorrência × 1 aeronave × 1 ocorrência**. A maior parte das repetições de `Numero_da_Ocorrencia` vem de uma mesma aeronave classificada em vários tipos de ocorrência. Só 36 ocorrências têm mais de uma aeronave. Removemos apenas as duplicatas exatas e analisamos três visões: 5.011 linhas (tipos), **4.338 aeronave-ocorrências (unidade de análise)** e 4.301 ocorrências.

## Perguntas

1. **Evolução temporal e geográfica:** o número de ocorrências por ano (2009–2023) está subindo ou caindo? Há sazonalidade? Quais regiões/UFs concentram mais ocorrências?
2. **Operação × fase do voo:** em que tipo de operação e em que fase as ocorrências mais acontecem? Quais combinações concentram mais acidentes?
3. **Teste de hipótese:** a proporção de ocorrências com pelo menos uma vítima fatal é diferente entre **Voo Privado** e **Operação Agrícola**?

## Principais conclusões

- **Tendência:** a média de 2009–2023 é de **227,9 ocorrências/ano** (mediana 224; desvio padrão 36,8). Não há tendência linear no período inteiro (+1,0/ano; p = 0,680). Houve alta até o pico de 2012 (289) e depois **queda significativa de −4,5/ano até 2023** (p = 0,027). A queda vem dos **acidentes** (209 → 156; −5,4/ano, p < 0,001); os incidentes graves ficaram estáveis (p = 0,481).
- **Sazonalidade:** janeiro tem a maior média mensal (22,7; 9,9% do total) e junho a menor (14,2; 6,2%). O padrão vem da **aviação agrícola**, que concentra **61,1%** das suas ocorrências entre dezembro e março (safra). O sábado é o dia com mais ocorrências (590).
- **Onde:** Sudeste (33,6%) e Centro-Oeste (22,1%). As UFs líderes são **SP (19,6%), MT (8,9%) e MG (8,5%)**.
- **Operação e fase:** o Voo Privado responde por **46,4%** das aeronaves envolvidas. **Pouso (20,2%)** e **Decolagem (16,3%)** são as fases mais frequentes. Entre os acidentes, as combinações líderes são **Voo Privado / Pouso (188; 9,1%)**, Voo Privado / Em rota (172; 8,4%) e Voo Privado / Decolagem (168; 8,2%). Na aviação agrícola, a fase mais crítica é a **decolagem** (24,7% dos acidentes agrícolas).

## Teste de hipótese

- **H₀:** a proporção de aeronave-ocorrências com ≥ 1 vítima fatal é igual entre Voo Privado e Operação Agrícola. **H₁:** as proporções são diferentes. α = 0,05.
- Usamos a variável `teve_fatalidade`, construída a partir das colunas `Lesoes_Fatais_*`. **Ausente não foi tratado como zero:** linhas com desfecho desconhecido ficaram fora do teste (17,4% do Voo Privado e 25,9% da Operação Agrícola).

| Grupo | Com fatalidade | Total | % |
|---|---|---|---|
| Voo Privado | 480 | 1.629 | **29,5%** |
| Operação Agrícola | 110 | 559 | **19,7%** |

- Qui-quadrado de independência: χ² = 20,25, gl = 1, **valor-p = 6,8 × 10⁻⁶**. O teste z de duas proporções dá o mesmo resultado (z = 4,50).
- Premissas: menor contagem esperada = 150,7 (≥ 5 ✔). Independência razoável, mas não perfeita: 15% das linhas são de aeronaves que aparecem em mais de uma ocorrência.
- Tamanho do efeito: diferença de **+9,8 p.p. (IC 95%: +5,8 a +13,8)**, risco relativo 1,50, Cramér's V = 0,10 (efeito fraco).
- **Interpretação:** rejeitamos H₀. A diferença é real e não fruto do acaso: proporcionalmente, as ocorrências do voo privado terminam com mortes com mais frequência que as da aviação agrícola. O resultado se mantém em todos os cenários de sensibilidade. Restrito a acidentes, a diferença sobe para 14,4 p.p.

**Resultado inesperado:** a aviação agrícola, considerada de alto risco, tem **menos** fatalidade por ocorrência que o voo privado. Uma explicação plausível, ainda não testada, é que as aeronaves agrícolas levam uma única pessoa (mediana de 1 assento, contra 4 no voo privado).

**Limitações:** não há denominador de exposição (horas voadas ou decolagens), a base só contém acidentes e incidentes graves, o critério de notificação muda ao longo do tempo, existem ausentes relevantes (~29% sem fase do voo) e associação não implica causa. Os detalhes estão na seção 8 do notebook.

## Estrutura

```
projeto-ocorrencias-aeronauticas/
├── dados/V_OCORRENCIA_AMPLA.csv   # base original, intocada
├── analise_ocorrencias.ipynb      # notebook executado e comentado (8 seções)
├── figuras/                       # gráficos gerados pelo notebook
├── requirements.txt
└── README.md
```

## Como executar

```bash
python -m pip install -r requirements.txt
python -m notebook analise_ocorrencias.ipynb
```

No Jupyter, use **Kernel → Restart & Run All**. Para executar tudo pela linha de comando, sem abrir o navegador:

```bash
python -m nbconvert --to notebook --execute --inplace analise_ocorrencias.ipynb
```

(Usar `python -m ...` funciona mesmo quando o comando `jupyter` não está no PATH, o que é comum no Windows.)

O notebook lê a base por caminho relativo, então execute-o a partir da pasta do projeto.
