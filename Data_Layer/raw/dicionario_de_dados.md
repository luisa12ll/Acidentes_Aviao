# 📖 Dicionário de Dados - Aviation Data

## 📋 Informações do Projeto

| Item | Descrição |
|------|-----------|
| **Projeto** | Data Warehouse - Arquitetura Medalhão |
| **Disciplina** | Sistemas de Banco de Dados 2 |
| **Professor** | Thiago Luiz de Souza Gomes |
| **Instituição** | UnB - FCTE - Faculdade de Ciências e Tecnologias em Engenharias  |
| **Semestre** | 2025.4 |
| **Data** | Janeiro 2026 |

### 👥 Grupo 15

- Caio Ferreira Duarte - 231026901
- Laryssa Felix Ribeiro Lopes - 231026840
- Luísa de Souza Ferreira - 232014807
- Henrique Fontenelle Galvão Passos - 231030771
- Marjorie Mitzi Cavalcante Rodrigues - 231039140

---

## 🎯 Objetivo

Este documento descreve a estrutura e o significado de cada campo do dataset de **acidentes e incidentes aéreos**, fornecendo informações essenciais para compreensão, análise e transformação dos dados nas camadas Bronze (RAW), Silver e Gold do projeto.

---

## 📊 Informações Gerais do Dataset

| Característica | Valor |
|----------------|-------|
| **Formato Original** | CSV |
| **Encoding** | cp1252 (Windows-1252) |
| **Total de Registros** | ~88.889 |
| **Total de Colunas** | 31 |
| **Tamanho Aproximado** | 21 MB |
| **Período dos Dados** | 1948 - 2025 |
| **Granularidade** | 1 registro = 1 evento único |
| **Fonte** | NTSB (National Transportation Safety Board) |

---

## 📑 Estrutura dos Dados

### Índice de Campos

1. [Identificação](#1-identificação)
2. [Informações Temporais](#2-informações-temporais)
3. [Localização Geográfica](#3-localização-geográfica)
4. [Informações da Aeronave](#4-informações-da-aeronave)
5. [Severidade e Danos](#5-severidade-e-danos)
6. [Informações de Vítimas](#6-informações-de-vítimas)
7. [Informações Operacionais](#7-informações-operacionais)
8. [Condições do Evento](#8-condições-do-evento)

---

## 1. Identificação

### 🔑 Event.Id

| Atributo | Valor |
|----------|-------|
| **Nome** | Event.Id |
| **Tipo de Dado** | TEXT (String) |
| **Função** | Chave Primária |
| **Descrição** | Identificador único de cada evento de aviação (acidente ou incidente) |
| **Formato** | Alfanumérico (ex: 20001218X45444) |
| **Obrigatório** | ✅ Sim (100% preenchido) |
| **Valores Únicos** | Sim |
| **Uso no Star Schema** | Chave da tabela fato |
| **Observações** | Gerado pela agência investigadora; garante unicidade absoluta |

**Exemplo:** `20001218X45444`

---

### 🔖 Investigation.Type

| Atributo | Valor |
|----------|-------|
| **Nome** | Investigation.Type |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Tipo de investigação realizada sobre o evento |
| **Valores Possíveis** | `Accident`, `Incident` |
| **Obrigatório** | ✅ Sim (100% preenchido) |
| **Uso no Star Schema** | Dimensão (dim_event_type) |
| **Observações** | Diferencia acidentes graves de incidentes menores |

**Exemplos:**
- `Accident` - Evento com danos significativos ou vítimas
- `Incident` - Evento sem consequências graves

---

### 📋 Accident.Number

| Atributo | Valor |
|----------|-------|
| **Nome** | Accident.Number |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Número oficial do acidente atribuído pela agência investigadora |
| **Formato** | Código alfanumérico (ex: SEA87LA080) |
| **Obrigatório** | ✅ Sim (100% preenchido) |
| **Valores Únicos** | Geralmente sim (pode haver duplicatas em casos de múltiplas aeronaves) |
| **Uso no Star Schema** | Atributo descritivo |

**Exemplo:** `SEA87LA080`

---

## 2. Informações Temporais

### 📅 Event.Date

| Atributo | Valor |
|----------|-------|
| **Nome** | Event.Date |
| **Tipo de Dado** | DATE (armazenado como TEXT no RAW) |
| **Descrição** | Data em que o evento ocorreu |
| **Formato Original** | YYYY-MM-DD |
| **Obrigatório** | ✅ Sim (100% preenchido) |
| **Uso no Star Schema** | Chave estrangeira para dim_time |
| **Transformações ETL** | Converter de TEXT para DATETIME; Extrair: ano, mês, dia, dia_semana, trimestre |

**Exemplo:** `1948-10-24`

**Componentes Derivados (ETL):**
- `year` - Ano (1948-2025)
- `month` - Mês (1-12)
- `day` - Dia (1-31)
- `day_of_week` - Dia da semana (Monday-Sunday)
- `quarter` - Trimestre (1-4)
- `decade` - Década (1940s, 1950s, etc.)

---

### 📄 Publication.Date

| Atributo | Valor |
|----------|-------|
| **Nome** | Publication.Date |
| **Tipo de Dado** | DATE (armazenado como TEXT no RAW) |
| **Descrição** | Data de publicação do relatório final de investigação |
| **Formato Original** | DD-MM-YYYY |
| **Completude** | ~84% (16% nulos) |
| **Uso no Star Schema** | Atributo complementar ou dimensão temporal secundária |
| **Transformações ETL** | Converter de TEXT para DATETIME |

**Exemplo:** `19-09-1996`

---

## 3. Localização Geográfica

### 📍 Location

| Atributo | Valor |
|----------|-------|
| **Nome** | Location |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Local do evento (geralmente cidade e estado) |
| **Formato** | Texto livre, normalmente "CIDADE, ESTADO" |
| **Completude** | ~99.9% |
| **Uso no Star Schema** | Dimensão geográfica (dim_location) |
| **Transformações ETL** | Normalizar formato; Extrair cidade e estado separadamente |

**Exemplos:**
- `MOOSE CREEK, ID`
- `BRIDGEPORT, CA`
- `Saltville, VA`

---

### 🌍 Country

| Atributo | Valor |
|----------|-------|
| **Nome** | Country |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | País onde o evento ocorreu |
| **Completude** | ~99.7% |
| **Uso no Star Schema** | Dimensão geográfica (dim_location) - nível hierárquico superior |
| **Observações** | Predominância de "United States" (~95% dos registros) |

**Exemplos:**
- `United States`
- `Canada`
- `Brazil`

---

### 🗺️ Latitude

| Atributo | Valor |
|----------|-------|
| **Nome** | Latitude |
| **Tipo de Dado** | DECIMAL (armazenado como TEXT no RAW) |
| **Descrição** | Coordenada de latitude do local do evento |
| **Formato** | Graus decimais (-90 a +90) |
| **Completude** | ~39% (61% nulos) |
| **Uso no Star Schema** | Dimensão geográfica (dim_location) |
| **Transformações ETL** | Converter de TEXT para FLOAT; Criar flag `has_coordinates` |

**Exemplo:** `36.922223`

---

### 🗺️ Longitude

| Atributo | Valor |
|----------|-------|
| **Nome** | Longitude |
| **Tipo de Dado** | DECIMAL (armazenado como TEXT no RAW) |
| **Descrição** | Coordenada de longitude do local do evento |
| **Formato** | Graus decimais (-180 a +180) |
| **Completude** | ~39% (61% nulos) |
| **Uso no Star Schema** | Dimensão geográfica (dim_location) |
| **Transformações ETL** | Converter de TEXT para FLOAT; Criar flag `has_coordinates` |

**Exemplo:** `-81.878056`

**⚠️ Observação:** Apenas ~39% dos eventos têm coordenadas precisas. Para análises geográficas, usar hierarquia Country > State > City.

---

### ✈️ Airport.Code

| Atributo | Valor |
|----------|-------|
| **Nome** | Airport.Code |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Código IATA/ICAO do aeroporto relacionado ao evento |
| **Formato** | 3-4 caracteres alfanuméricos |
| **Completude** | ~56% |
| **Uso no Star Schema** | Dimensão geográfica (dim_airport) ou atributo de dim_location |

**Exemplos:**
- `LAX` - Los Angeles International
- `JFK` - John F. Kennedy International
- `5G6` - Cherry Springs

---

### 🛫 Airport.Name

| Atributo | Valor |
|----------|-------|
| **Nome** | Airport.Name |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Nome completo do aeroporto |
| **Completude** | ~59% |
| **Uso no Star Schema** | Dimensão geográfica (dim_airport) |

**Exemplos:**
- `Los Angeles International`
- `Blackburn Ag Strip`
- `Westchester County`

---

## 4. Informações da Aeronave

### 🛩️ Aircraft.Category

| Atributo | Valor |
|----------|-------|
| **Nome** | Aircraft.Category |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Categoria ou tipo de aeronave |
| **Valores Possíveis** | `Airplane`, `Helicopter`, `Glider`, `Balloon`, etc. |
| **Completude** | ~36% (64% nulos) ⚠️ |
| **Uso no Star Schema** | Dimensão aeronave (dim_aircraft) |
| **Transformações ETL** | Preencher "Unknown" para nulos; Tentar inferir de Model quando possível |

**Exemplos:**
- `Airplane`
- `Helicopter`
- `Glider`

---

### 🔖 Registration.Number

| Atributo | Valor |
|----------|-------|
| **Nome** | Registration.Number |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Número de registro/matrícula da aeronave |
| **Formato** | Varia por país (EUA: N + números/letras) |
| **Completude** | ~98% |
| **Uso no Star Schema** | Dimensão aeronave (dim_aircraft) |
| **Observações** | Identificador único da aeronave específica |

**Exemplos:**
- `N6404`
- `N5069P`
- `CF-TLU` (Canadá)

---

### 🏭 Make

| Atributo | Valor |
|----------|-------|
| **Nome** | Make |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Fabricante da aeronave |
| **Completude** | ~99.9% |
| **Uso no Star Schema** | Dimensão aeronave (dim_aircraft) - nível hierárquico |
| **Observações** | Cessna e Piper são os fabricantes mais comuns |

**Exemplos:**
- `Cessna`
- `Piper`
- `Boeing`
- `Beech`

---

### 🔧 Model

| Atributo | Valor |
|----------|-------|
| **Nome** | Model |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Modelo específico da aeronave |
| **Completude** | ~99.9% |
| **Uso no Star Schema** | Dimensão aeronave (dim_aircraft) |
| **Observações** | Hierarquia: Make > Model |

**Exemplos:**
- `172M` (Cessna 172M)
- `PA-28-161` (Piper Cherokee)
- `DC9` (McDonnell Douglas DC-9)

---

### 🔨 Amateur.Built

| Atributo | Valor |
|----------|-------|
| **Nome** | Amateur.Built |
| **Tipo de Dado** | BOOLEAN (armazenado como TEXT no RAW) |
| **Descrição** | Indica se a aeronave foi construída por amador (homebuilt) |
| **Valores Possíveis** | `Yes`, `No` |
| **Completude** | ~99.9% |
| **Uso no Star Schema** | Dimensão aeronave (dim_aircraft) - atributo classificatório |
| **Transformações ETL** | Converter para BOOLEAN (True/False) |

---

### ⚙️ Number.of.Engines

| Atributo | Valor |
|----------|-------|
| **Nome** | Number.of.Engines |
| **Tipo de Dado** | INTEGER (pode estar como FLOAT no RAW) |
| **Descrição** | Número de motores da aeronave |
| **Valores Típicos** | 1, 2, 3, 4 |
| **Completude** | ~93% |
| **Uso no Star Schema** | Dimensão aeronave (dim_aircraft) |

**Distribuição Típica:**
- 1 motor: ~70% (aviação geral)
- 2 motores: ~25%
- 3+ motores: ~5% (aviação comercial)

---

### 🚀 Engine.Type

| Atributo | Valor |
|----------|-------|
| **Nome** | Engine.Type |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Tipo de motor da aeronave |
| **Valores Possíveis** | `Reciprocating`, `Turbo Fan`, `Turbo Prop`, `Turbo Shaft`, `Electric`, etc. |
| **Completude** | ~93% |
| **Uso no Star Schema** | Dimensão aeronave (dim_aircraft) |

**Exemplos:**
- `Reciprocating` - Motor a pistão (mais comum em aviação geral)
- `Turbo Fan` - Turbofan (aviação comercial)
- `Turbo Prop` - Turboélice

---

## 5. Severidade e Danos

### 🚨 Injury.Severity

| Atributo | Valor |
|----------|-------|
| **Nome** | Injury.Severity |
| **Tipo de Dado** | TEXT (Categorical com contador) |
| **Descrição** | Classificação da severidade das lesões no evento |
| **Formato** | Categoria(número) - ex: "Fatal(2)" |
| **Completude** | ~99% |
| **Uso no Star Schema** | Dimensão severidade (dim_severity) |
| **Transformações ETL** | Extrair categoria e número separadamente |

**Valores Possíveis:**
- `Fatal(N)` - Evento com N mortes
- `Non-Fatal` - Evento sem mortes
- `Incident` - Incidente sem lesões significativas
- `Unavailable` - Informação não disponível

**Exemplos:**
- `Fatal(2)` - 2 vítimas fatais
- `Fatal(4)` - 4 vítimas fatais
- `Non-Fatal`

---

### ✈️ Aircraft.damage

| Atributo | Valor |
|----------|-------|
| **Nome** | Aircraft.damage |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Extensão dos danos à aeronave |
| **Valores Possíveis** | `Destroyed`, `Substantial`, `Minor`, `None` |
| **Completude** | ~96% |
| **Uso no Star Schema** | Dimensão severidade (dim_severity) |

**Classificação:**
- `Destroyed` - Aeronave totalmente destruída (perda total)
- `Substantial` - Danos significativos que afetam integridade estrutural
- `Minor` - Danos superficiais ou reparáveis facilmente
- `None` - Sem danos (raro em acidentes, comum em incidentes)

---

### 📋 Report.Status

| Atributo | Valor |
|----------|-------|
| **Nome** | Report.Status |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Status do relatório de investigação |
| **Valores Comuns** | `Probable Cause`, `Factual`, `Foreign`, etc. |
| **Completude** | ~93% |
| **Uso no Star Schema** | Dimensão severidade ou atributo do fato |

**Exemplo:** `Probable Cause` - Relatório final com causa provável determinada

---

## 6. Informações de Vítimas

### 💔 Total.Fatal.Injuries

| Atributo | Valor |
|----------|-------|
| **Nome** | Total.Fatal.Injuries |
| **Tipo de Dado** | INTEGER |
| **Descrição** | Número total de vítimas fatais no evento |
| **Valores** | 0 a N |
| **Completude** | ~100% (nulos = 0) |
| **Uso no Star Schema** | **FATO** - Medida agregável (somável) |
| **Transformações ETL** | Substituir nulos por 0 |

**Estatística:** Soma total no dataset representa o total de mortes em acidentes aéreos no período.

---

### 🤕 Total.Serious.Injuries

| Atributo | Valor |
|----------|-------|
| **Nome** | Total.Serious.Injuries |
| **Tipo de Dado** | INTEGER |
| **Descrição** | Número total de feridos graves no evento |
| **Valores** | 0 a N |
| **Completude** | ~100% (nulos = 0) |
| **Uso no Star Schema** | **FATO** - Medida agregável (somável) |
| **Transformações ETL** | Substituir nulos por 0 |

**Definição:** Ferimentos que requerem hospitalização prolongada.

---

### 🩹 Total.Minor.Injuries

| Atributo | Valor |
|----------|-------|
| **Nome** | Total.Minor.Injuries |
| **Tipo de Dado** | INTEGER |
| **Descrição** | Número total de feridos leves no evento |
| **Valores** | 0 a N |
| **Completude** | ~100% (nulos = 0) |
| **Uso no Star Schema** | **FATO** - Medida agregável (somável) |
| **Transformações ETL** | Substituir nulos por 0 |

**Definição:** Ferimentos superficiais ou que não requerem internação.

---

### ✅ Total.Uninjured

| Atributo | Valor |
|----------|-------|
| **Nome** | Total.Uninjured |
| **Tipo de Dado** | INTEGER |
| **Descrição** | Número total de pessoas ilesas no evento |
| **Valores** | 0 a N |
| **Completude** | ~100% (nulos = 0) |
| **Uso no Star Schema** | **FATO** - Medida agregável (somável) |
| **Transformações ETL** | Substituir nulos por 0 |

**Observação:** Pessoas a bordo que não sofreram nenhum ferimento.

---

## 7. Informações Operacionais

### 📜 FAR.Description

| Atributo | Valor |
|----------|-------|
| **Nome** | FAR.Description |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Parte regulatória aplicável ao voo (Federal Aviation Regulations) |
| **Valores Comuns** | `Part 91: General Aviation`, `Part 121: Air Carrier`, `Part 135: Air Taxi & Commuter`, etc. |
| **Completude** | ~89% |
| **Uso no Star Schema** | Dimensão operacional (dim_flight) |

**Exemplos:**
- `Part 91: General Aviation` - Aviação geral
- `Part 121: Domestic Flag & Supplemental` - Aviação comercial regular
- `Part 135: Air Taxi & Commuter` - Táxi aéreo

---

### 📋 Schedule

| Atributo | Valor |
|----------|-------|
| **Nome** | Schedule |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Tipo de operação (programada ou não-programada) |
| **Valores Possíveis** | `SCHD` (Scheduled), `NSCH` (Non-Scheduled) |
| **Completude** | ~47% |
| **Uso no Star Schema** | Dimensão operacional (dim_flight) |

---

### 🎯 Purpose.of.flight

| Atributo | Valor |
|----------|-------|
| **Nome** | Purpose.of.flight |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Propósito ou objetivo do voo |
| **Valores Comuns** | `Personal`, `Business`, `Instructional`, `Ferry`, `Aerial Observation`, etc. |
| **Completude** | ~88% |
| **Uso no Star Schema** | Dimensão operacional (dim_flight) |

**Exemplos:**
- `Personal` - Voo recreativo/pessoal (mais comum)
- `Business` - Voo corporativo
- `Instructional` - Treinamento/instrução
- `Ferry` - Reposicionamento de aeronave

---

### 🏢 Air.carrier

| Atributo | Valor |
|----------|-------|
| **Nome** | Air.carrier |
| **Tipo de Dado** | TEXT (String) |
| **Descrição** | Nome da companhia aérea ou operador |
| **Completude** | ~17% (maioria é aviação geral sem operador comercial) |
| **Uso no Star Schema** | Dimensão operacional (dim_airline) ou atributo |

**Exemplos:**
- `Delta Airlines`
- `Rocky Mountain Helicopters, Inc`
- `Empire Airlines`

---

### 🛫 Broad.phase.of.flight

| Atributo | Valor |
|----------|-------|
| **Nome** | Broad.phase.of.flight |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Fase do voo em que o evento ocorreu |
| **Valores Comuns** | `Landing`, `Takeoff`, `Cruise`, `Approach`, `Climb`, `Descent`, `Taxi`, `Maneuvering`, etc. |
| **Completude** | ~90% |
| **Uso no Star Schema** | Dimensão operacional (dim_flight_phase) |

**Fases Críticas (estatisticamente):**
1. `Landing` - Pouso (fase mais crítica)
2. `Takeoff` - Decolagem
3. `Approach` - Aproximação

---

## 8. Condições do Evento

### 🌤️ Weather.Condition

| Atributo | Valor |
|----------|-------|
| **Nome** | Weather.Condition |
| **Tipo de Dado** | TEXT (Categorical) |
| **Descrição** | Condições meteorológicas durante o evento |
| **Valores Possíveis** | `VMC`, `IMC`, `UNK` |
| **Completude** | ~93% |
| **Uso no Star Schema** | Dimensão meteorológica (dim_weather) |

**Glossário:**
- `VMC` - **Visual Meteorological Conditions** (Condições visuais - céu claro, boa visibilidade)
- `IMC` - **Instrument Meteorological Conditions** (Condições por instrumentos - baixa visibilidade, nuvens)
- `UNK` - **Unknown** (Desconhecido)

**Observação:** ~70% dos eventos ocorrem em VMC (boas condições meteorológicas).

---

## 📊 Resumo de Completude dos Dados

| Campo | Completude | Status | Estratégia ETL |
|-------|-----------|--------|----------------|
| Event.Id | 100% | 🟢 Excelente | Manter |
| Investigation.Type | 100% | 🟢 Excelente | Manter |
| Accident.Number | 100% | 🟢 Excelente | Manter |
| Event.Date | 100% | 🟢 Excelente | Converter para datetime |
| Make | 99.9% | 🟢 Excelente | Manter |
| Model | 99.9% | 🟢 Excelente | Manter |
| Location | 99.9% | 🟢 Excelente | Normalizar |
| Country | 99.7% | 🟢 Excelente | Normalizar |
| Registration.Number | 98% | 🟢 Ótimo | Manter |
| Injury.Severity | 99% | 🟢 Ótimo | Extrair categoria e número |
| Aircraft.damage | 96% | 🟢 Ótimo | Categorizar |
| Weather.Condition | 93% | 🟡 Bom | Preencher "UNK" |
| Number.of.Engines | 93% | 🟡 Bom | Manter ou inferir |
| Engine.Type | 93% | 🟡 Bom | Categorizar |
| Publication.Date | 84% | 🟡 Aceitável | Converter para datetime |
| Airport.Name | 59% | 🟠 Moderado | Aceitar nulos |
| Airport.Code | 56% | 🟠 Moderado | Aceitar nulos |
| Latitude | 39% | 🔴 Baixo | Criar flag has_coordinates |
| Longitude | 39% | 🔴 Baixo | Criar flag has_coordinates |
| Aircraft.Category | 36% | 🔴 Baixo | Inferir de Model ou "Unknown" |

---

## 🔧 Estratégias de Tratamento (ETL)

### ✅ Campos Obrigatórios (sem nulos permitidos)
- `Event.Id` - Já está completo
- `Event.Date` - Já está completo

### 🔄 Conversão de Tipos
- `Event.Date`: object → datetime
- `Publication.Date`: object → datetime
- `Latitude`: object → float
- `Longitude`: object → float
- `Amateur.Built`: object → boolean

### 📝 Preenchimento de Nulos

#### Valores Numéricos (Vítimas)
```python
# Nulos = 0 (ausência de vítimas)
Total.Fatal.Injuries = COALESCE(valor, 0)
Total.Serious.Injuries = COALESCE(valor, 0)
Total.Minor.Injuries = COALESCE(valor, 0)
Total.Uninjured = COALESCE(valor, 0)
```

#### Valores Categóricos
```python
# Nulos = "Unknown"
Aircraft.Category = COALESCE(valor, 'Unknown')
Weather.Condition = COALESCE(valor, 'UNK')
```

### 🏴 Criação de Flags
```python
has_coordinates = (Latitude IS NOT NULL AND Longitude IS NOT NULL)
has_airport_info = (Airport.Code IS NOT NULL OR Airport.Name IS NOT NULL)
has_aircraft_category = (Aircraft.Category IS NOT NULL)
```

---

## 🌟 Modelagem Dimensional (Star Schema)

### Dimensões Propostas

#### 📅 dim_time
- time_key (PK)
- date
- year, month, day
- day_of_week, day_name
- quarter, semester
- is_weekend, is_holiday

#### 🌍 dim_location
- location_key (PK)
- country
- state, city
- airport_code, airport_name
- latitude, longitude
- region, timezone

#### ✈️ dim_aircraft
- aircraft_key (PK)
- make, model
- category
- registration_number
- amateur_built
- num_engines, engine_type

#### 🌤️ dim_weather
- weather_key (PK)
- condition_code (VMC/IMC/UNK)
- condition_description

#### 🚨 dim_severity
- severity_key (PK)
- injury_category (Fatal/Non-Fatal/Incident)
- damage_category (Destroyed/Substantial/Minor)
- report_status

#### 🛫 dim_flight_phase
- phase_key (PK)
- phase_name
- phase_group (Takeoff/Cruise/Landing)

### Tabela Fato

#### 📊 fact_events
- event_key (PK)
- time_key (FK)
- location_key (FK)
- aircraft_key (FK)
- weather_key (FK)
- severity_key (FK)
- phase_key (FK)
- **Medidas:**
  - total_fatal_injuries
  - total_serious_injuries
  - total_minor_injuries
  - total_uninjured
  - total_persons
  - event_count (sempre = 1)

---

## 📚 Glossário de Termos

| Termo | Significado |
|-------|-------------|
| **NTSB** | National Transportation Safety Board - Agência dos EUA responsável pela investigação |
| **VMC** | Visual Meteorological Conditions - Condições meteorológicas visuais |
| **IMC** | Instrument Meteorological Conditions - Condições meteorológicas por instrumentos |
| **FAR** | Federal Aviation Regulations - Regulamentações Federais de Aviação |
| **IATA** | International Air Transport Association - Código de 3 letras para aeroportos |
| **ICAO** | International Civil Aviation Organization - Código de 4 letras para aeroportos |
| **Reciprocating** | Motor a pistão (tipo mais comum em aviação geral) |
| **Homebuilt** | Aeronave construída por amador |
| **General Aviation** | Aviação geral (não comercial) - aviões pequenos, particulares |
| **Air Carrier** | Transporte aéreo comercial regular |
| **Ferry Flight** | Voo de reposicionamento de aeronave sem passageiros |

---

## ⚠️ Observações Importantes

### 1. Qualidade dos Dados
- Dataset tem **completude geral de ~75-80%**
- Campos essenciais (Make, Model, Date) têm **excelente qualidade (>99%)**
- Campos opcionais (Category, Coordinates) têm **muitos nulos (>60%)**

### 2. Viés nos Dados
- **~95% dos eventos são dos Estados Unidos** - dataset americano
- Predominância de **aviação geral** sobre comercial
- Eventos mais antigos têm **menos informações detalhadas**

### 3. Limitações
- **Coordenadas geográficas**: apenas 39% dos eventos
- **Aircraft.Category**: 64% nulos - usar Make/Model como alternativa
- **Air.carrier**: 83% nulos - maioria é aviação privada

### 4. Recomendações
- Usar **Country e Location** para análises geográficas (alta completude)
- Usar **Make e Model** como base da dimensão aeronave
- Criar **flags de qualidade** para facilitar filtragem de análises
- **Não remover registros** com campos nulos - usar estratégias de preenchimento

---

## 📅 Histórico de Versões

| Versão | Data | Autor | Alterações |
|--------|------|-------|------------|
| 1.0 | 2026-01-09 | Grupo 15 | Criação inicial do dicionário de dados |

---


Para dúvidas ou sugestões sobre este dicionário:

- **Grupo:** Grupo 15
- **Disciplina:** Sistemas de Banco de Dados 2
- **Professor:** Thiago Luiz de Souza Gomes

---

**📖 Documento de Referência - Projeto SBD2 - 2025.4**