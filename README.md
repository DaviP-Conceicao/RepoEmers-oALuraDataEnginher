# 🚀 Imersão Alura + Databricks - Data Engineering

> 🏗️ **Projeto em andamento** - Imersão de Engenharia de Dados

## 📋 Sobre o Projeto

Repositório do projeto desenvolvido durante a **Imersão Alura de Data Engineering**, aplicando conceitos de arquitetura medalhão (Bronze → Silver → Gold) para análise de dados de aviação civil brasileira.

## 🗂️ Estrutura do Projeto

### Notebooks Implementados

#### 🥉 Camada Bronze (Ingestão)
- **`bronze_vra.ipynb`**: Ingestão de dados de voos (VRA - Voos Regulares Ativos) da ANAC
- **`bronze_referencias.ipynb`**: Ingestão de cadastros de referência
  - Empresas aéreas (nacionais e estrangeiras)
  - Aeródromos públicos
  - Códigos de operação

#### 🥈 Camada Silver (Transformação e Governança)
- **`Silver - Espelho governado do bronze.ipynb`**: Transformações com tipagem, cálculos e documentação completa
  - ✅ Tipagem de colunas (timestamps, numéricos)
  - ✅ Separação de data e hora
  - ✅ Cálculos de atraso (partida, chegada, recuperação)
  - ✅ Unificação de cadastros
  - ✅ 100% de documentação (54 colunas comentadas)
  - ✅ Metadados e tags de governança

## 📊 Dados Tratados

### Catálogo Unity: `voebem`

**Schema Bronze:**
- `bronze.vra` - Voos regulares ativos
- `bronze.empresas_aereas` - Operadores aéreos
- `bronze.aerodromos` - Aeroportos públicos
- `bronze.codigos_operacao` - Seed table de códigos

**Schema Silver (4 tabelas governadas):**
- `silver.vra` (26 colunas) - Etapas de voo com métricas de atraso
- `silver.empresas` (11 colunas) - Cadastro unificado de operadores
- `silver.aerodromos` (13 colunas) - Aeródromos com coordenadas
- `silver.codigos_operacao` (4 colunas) - Dicionário de códigos

## 🎯 Próximos Passos

- [ ] Camada Gold (Agregações e Métricas de Negócio)
- [ ] Análises de pontualidade
- [ ] Dashboards e visualizações
- [ ] Automação e orquestração

## 🛠️ Tecnologias

- **Databricks** - Plataforma de dados unificada
- **Apache Spark** - Processamento distribuído
- **Delta Lake** - Armazenamento confiável
- **Unity Catalog** - Governança de dados
- **PySpark & SQL** - Transformações de dados

## 📝 Governança de Dados

Todas as tabelas Silver incluem:
- ✅ Comentários descritivos em todas as colunas
- ✅ Tags de metadados (camada, domínio, fonte, grão)
- ✅ Auditoria completa (arquivo origem, timestamps de ingestão/transformação)
- ✅ Validações de contagem entre camadas

---

**Status**: 🟢 Em desenvolvimento ativo  
**Fonte de dados**: [ANAC - Agência Nacional de Aviação Civil](https://www.gov.br/anac/)
