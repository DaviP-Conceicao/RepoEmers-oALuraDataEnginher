# ✈️ voebem | Pipeline de Dados de Aviação Civil

> Imersão Alura + Databricks | Arquitetura Medalhão completa

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   🥉 BRONZE  │ ───▶ │   🥈 SILVER  │ ───▶ │   🥇 GOLD    │
│   Landing   │      │  Validação  │      │  Analytics  │
└─────────────┘      └─────────────┘      └─────────────┘
     CSV                Delta Lake           Star Schema
  1.014.882           1.014.841 voos         985.524 voos
   registros          (41 duplicatas)       255 aeroportos
```

## 📊 Números do Projeto

| Métrica | Valor |
|---------|-------|
| 🗂️ **Tabelas criadas** | 11 (4 bronze + 4 silver + 3 gold) |
| 📝 **Colunas documentadas** | 129 COMMENTs aplicados |
| 🏷️ **Tags de governança** | 38 tags (camada, domínio, consumo) |
| ✅ **Dados validados** | 985.524 voos (97,1% qualidade) |
| 🚨 **Quarentena** | 29.317 registros (2,9%) |
| 📈 **Cobertura lineage** | 100% (bronze → silver → gold) |
| 🤖 **Agente Genie** | 1 space configurado (5 starter questions + 4 SQL examples) |

## 🏗️ Arquitetura

### 🥉 Bronze: Ingestão
```
CSV files (ANAC)
  ├─ vra.csv (1.014.882 linhas)         → bronze.vra
  ├─ empresas_aereas.csv (189 empresas) → bronze.empresas_aereas
  ├─ aerodromos.csv (278 aeroportos)    → bronze.aerodromos
  └─ codigos.csv (seed table)           → bronze.codigos_operacao
```

### 🥈 Silver: Qualidade + Tipagem
```
Pipeline de Qualidade:
  ├─ 01_vra-marcados.sql    → Marca problemas (atraso fora de faixa, ICAO inválido)
  ├─ 02_vra_auditados.sql   → Aprova (985.524) vs Quarentena (29.317)
  └─ 03_vra_quarentena.sql  → Isola dados suspeitos

Tabelas Governadas:
  ├─ silver.vra (26 cols)             → Métricas de atraso validadas
  ├─ silver.empresas (11 cols)        → Cadastro unificado
  ├─ silver.aerodromos (13 cols)      → Lat/long + classificação
  └─ silver.codigos_operacao (4 cols) → Dicionário de códigos DI
```

### 🥇 Gold: Analytics
```
Modelo Estrela:
  ├─ gold.fato_voos (985.524 linhas)   → Métricas + dimensões degeneradas
  ├─ gold.dim_aeroporto (255 registros)→ Brasil (249) + Exterior (6)
  └─ gold.obt_voos (985.524 linhas)    → Tabela para consumo de IA
                                          (tudo desnormalizado)
```

## 🤖 Agente Genie: VoeBem Analytics

### Configuração do Agente

**Nome**: VoeBem Analytics  
**Tipo**: Genie Space (Agent Bricks)  
**Warehouse**: Serverless Starter Warehouse  
**Tabela**: `voebem.gold.obt_voos` (One Big Table desnormalizada)  
**Período**: Agosto 2025 a Julho 2026

### Capacidades

O agente responde perguntas em linguagem natural sobre:
- ✅ Pontualidade de voos (critério: 15 minutos)
- ✅ Atrasos de partida e chegada
- ✅ Taxas de cancelamento
- ✅ Rankings de companhias e aeroportos
- ✅ Análises temporais (por hora, dia, mês)
- ✅ Recuperação de tempo em voo
- ✅ Comparação doméstico x internacional

### Definições de Negócio Implementadas

**ATRASO**: Diferença em minutos entre horário real e programado. Negativo = adiantamento.

**PONTUAL**: Atraso ≤ 15 minutos. Pré-calculado em `partida_pontual` e `chegada_pontual`.

**RECUPERAÇÃO EM VOO**: `minutos_recuperados` = atraso_partida - atraso_chegada. Positivo significa que chegou **menos atrasado** (não necessariamente no horário).

**CANCELADO**: Não entra em métricas de atraso, apenas em taxa de cancelamento.

### Fórmulas SQL Padronizadas

**Percentual de Atraso** (sempre usar):
```sql
ROUND(100.0 * try_divide(
  SUM(CASE WHEN partida_pontual = false THEN 1 ELSE 0 END),
  SUM(CASE WHEN partida_pontual IS NOT NULL THEN 1 ELSE 0 END)
), 2)
```

### Starter Questions

1. Quais as 10 companhias com maior percentual de atraso na partida?
2. Qual o ranking de aeroportos brasileiros por pontualidade?
3. Qual a taxa de cancelamento por mês ao longo do período?
4. Quais as 10 rotas com maior atraso médio na chegada?
5. Como varia o atraso ao longo do dia por hora de partida?

### Exemplos SQL

O agente usa 4 SQL examples como referência:
- Top 10 companhias por % de atraso (HAVING COUNT(*) >= 10000)
- Taxa de cancelamento mensal
- Top 10 aeroportos brasileiros por pontualidade (HAVING COUNT(*) >= 5000)
- Evolução do atraso ao longo do dia

### Regras de Qualidade

✅ **Cortes de volume em rankings**:
- Companhias: ≥ 10.000 voos
- Aeroportos: ≥ 5.000 voos
- Rotas: ≥ 2.000 voos

✅ **NULL-safety**: Nunca usa `NOT partida_pontual` (NULL = não avaliável, não = atrasado)

✅ **Denominador correto**: Percentuais usam apenas voos com a métrica, não o total

✅ **Aeroportos brasileiros**: Filtra `pais_origem = 'Brasil'` quando não especificado

### Capturas do Agente em Ação

#### Interface Principal
![VoeBem Analytics - Interface](./docs/images/voebem-interface.png)
*Tela inicial com starter questions e configuração do agente*

#### Exemplo de Resposta
![Análise de Melhores Horários](./docs/images/voebem-resposta-horarios.png)
*Resposta com insights sobre melhores horários para pegar voo*

#### Visualização Gerada
![Top 10 Horários por Pontualidade](./docs/images/voebem-chart-horarios.png)
*Gráfico de barras mostrando os horários com maior pontualidade*

### Diferenciais do Agente

🎯 **SQL determinístico**: Fórmulas padronizadas garantem respostas consistentes  
🎯 **Validação semântica**: Definições testadas contra 985.524 voos reais  
🎯 **Segurança de tipos**: NULL-safety em todas as métricas booleanas  
🎯 **Context-aware**: Cortes de volume automáticos por tipo de entidade  
🎯 **Nomes legíveis**: Sempre retorna `nome_companhia`/`nome_origem`, nunca códigos ICAO  

---

## 🔍 Governança

### Documentação validada contra dados
```python
# Exemplo: minutos_recuperados
❌ IA sugeriu: "Positivo indica que chegou adiantado"
✅ Validado:   "Positivo = chegou MENOS ATRASADO (não no horário)"

Prova: 164.895 voos recuperaram tempo e AINDA ASSIM chegaram atrasados
```

### Tags aplicadas
| Tabela | Tags |
|--------|------|
| `gold.obt_voos` | `camada=gold` `dominio=aviacao` `consumo=genie` `tipo=obt` `consumidor=ia` |
| `gold.fato_voos` | `camada=gold` `dominio=aviacao` `consumo=bi` `tipo=fato` |
| `gold.dim_aeroporto` | `camada=gold` `dominio=aviacao` `consumo=bi` `tipo=dimensao` |

### Lineage rastreado
```
(volumes CSV)
    ↓
bronze.vra ──→ silver.vra ──→ gold.fato_voos ──┐
                                                 ├──→ gold.obt_voos
bronze.aerodromos ──→ silver.aerodromos ──→ gold.dim_aeroporto ──┘
```

## 🚀 Stack

| Componente | Tecnologia |
|------------|------------|
| **Plataforma** | Databricks (Serverless Compute) |
| **Processamento** | Apache Spark 3.5 |
| **Storage** | Delta Lake (Unity Catalog) |
| **Linguagens** | Python, SQL |
| **Orquestração** | SQL pipelines (bronze → silver → gold) |
| **Governança** | Unity Catalog (COMMENTs + Tags + Lineage) |

## 📂 Estrutura de Notebooks

```
.
├── bronze_vra.ipynb                      # Ingestão de voos
├── bronze_referencias.ipynb              # Ingestão de cadastros
├── Silver - Espelho governado.ipynb      # Transformações Silver
├── Fatos x Dim Gold.ipynb                # Modelagem estrela
├── Governancia-gold.ipynb                # COMMENTs + Tags + Lineage
├── Avaliação do agente.ipynb             # Métricas de qualidade IA
└── voebem-qualidade-silver/              # Pipeline de qualidade
    ├── 01_vra-marcados.sql               # Detecção de problemas
    ├── 02_vra_auditados.sql              # Decisão: aprovar/quarentena
    └── 03_vra_quarentena.sql             # Isolamento de suspeitos
```

## 🎯 Diferenciais do Projeto

✅ **Governança para IA**: OBT com 38 colunas documentadas para consumo de LLM  
✅ **Pipeline de qualidade**: Detecção + auditoria + quarentena automática  
✅ **Validação semântica**: Descrições testadas contra dados reais (164k exemplos)  
✅ **Lineage completo**: Rastreabilidade de ponta a ponta (CSV → Gold)  
✅ **Modelo híbrido**: Fato/Dim para BI + OBT para IA no mesmo catálogo  
✅ **Agente Genie configurado**: Interface conversacional com SQL determinístico e validação semântica  

---

**Fonte**: [ANAC - Agência Nacional de Aviação Civil](https://www.gov.br/anac/) | **Status**: ✅ Produção
