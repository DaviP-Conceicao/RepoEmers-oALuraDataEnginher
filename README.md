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

---

**Fonte**: [ANAC - Agência Nacional de Aviação Civil](https://www.gov.br/anac/) | **Status**: ✅ Produção
