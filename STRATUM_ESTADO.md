# STRATUM_ESTADO.md
# Documento de continuidad entre sesiones de Claude y Gemini
# Actualizado: Sesión 5 completada — Bloques 17-21 + fix framework-aware thresholds
# Pegar este documento al inicio de cada nueva sesión

---

## 🎯 IDENTIDAD DEL PROYECTO

**Producto:** STRATUM System Intelligence Engine v1.0.0
**Autor:** Javier Alfonso Quiroga Sarmiento
**Marca:** Estratega de Negocios: Arquitecto y Ejecutor de Ecosistemas Tecnológicos | Data Analytics | Financial BI | ERP Solutions
**Manifiesto:** STRATUM no reemplaza la IA. La potencia. Somos aliados de la IA, no su competencia.
**Filosofía:** STRATUM es el traductor entre un proyecto real y Claude/Gemini

---

## 📋 DECISIONES TOMADAS — NO REABRIR

| Decisión | Valor definido |
|---|---|
| Archivo único | `stratum.py` — todo en un solo archivo |
| Servidor | Solo `127.0.0.1` por defecto, `--red` para red local |
| Dependencias | CERO para núcleo. openpyxl+oletools opcional Excel. prophet+sklearn opcional ML |
| Período gratuito | 12 meses desde `LAUNCH_DATE = "2025-12-01"` |
| Early adopters | 50% descuento permanente para quienes instalen durante período free |
| Precios región | Colombia (COP), Latinoamérica (USD bajo), USA/Europa (USD completo) |
| Protección código | PyArmor + Nuitka (post v1.0, no bloquea desarrollo) |
| Puerto default | 7474 |
| Modo red | `--red` flag, avisa al usuario antes de activar |
| Licencia archivo | `.stratum_license.dat` en `~/.stratum/` con checksum SHA-256 |
| Log auditoría | `.stratum_audit.log` en `~/.stratum/` |
| Caché de análisis | `~/.stratum/cache/` — opcional, acelera re-análisis |
| Excel | FREE siempre para todos + donación voluntaria |
| Modelo negocio | Mes 1-12 gratis → Mes 13+ freemium con precios PPP |
| Lenguajes | Python(profundo), JS/TS, Java, C#, PHP, R, SQL, VBA/Excel |
| Estándar código | Cada bloque con título, cada función documentada, sin código huérfano |
| Demo LinkedIn | Capturas mes 1, video mes 2, demo pública mes 3 |
| Rol Gemini | Asesor de decisiones, NO generador de código |
| Rol Claude | Generador de código, arquitectura, pruebas |
| Persistencia engines | Memoria + caché opcional en `~/.stratum/cache/` |
| Paralelismo | Secuencial en v1.0 (predecible, sin race conditions) |
| Umbral CC Streamlit | Dos poblaciones calibradas con QS Ingeniería (~13K LOC): vista_*/modulo_*/render* → CC≤25 (UI overhead); lógica/utilidades → CC≤10 (McCabe). CC>25 = CRÍTICO siempre. Longitud CRÍTICO >400L. |
| Numeración bloques | Bloque ORQ (sin número) = orquestador. Bloques 0-28 = funcionalidades |
| Distribución v1.0 | GitHub Releases + sitio propio + PyPI + Winget. Microsoft Store en v1.2+ |

---

## ✅ BLOQUES COMPLETADOS — SESIÓN 1

### BLOQUE 0 — Encabezado y manifiesto ✅
- Banner ASCII con nombre, autor, marca
- Manifiesto de filosofía
- Instrucciones de uso CLI

### BLOQUE 1 — Configuración global ✅
- `STRATUM_VERSION`, `STRATUM_AUTOR`, `STRATUM_MARCA`
- `LAUNCH_DATE`, `FREE_PERIOD_DAYS`, `WARNING_DAYS_BEFORE`
- `PRECIOS` dict con Colombia/Latinoamérica/USA-Europa
- `LIMITES_TIER` por tier (trial/free/starter/professional/enterprise/early_free)
- `CARPETAS_IGNORADAS`, `EXTENSIONES_VALIDAS` por categoría
- `MAX_MB_POR_ARCHIVO = 5`, `MAX_MB_TOTAL_CONTEXTO = 10`

### BLOQUE 2 — Seguridad y autoverificación ✅
Clase `SeguridadStratum`:
- `configurar_logging()` → log de auditoría local en `~/.stratum/`
- `calcular_hash_propio()` → SHA-256 del script para distribución
- `verificar_integridad(hash_conocido)` → compara con hash oficial
- `obtener_machine_id()` → ID único de máquina para licenciamiento
- `validar_host(host, modo_red)` → garantiza localhost salvo --red

### BLOQUE 3 — Sistema de licencias ✅
Clase `SistemaLicencias`:
- Carga/crea licencia con checksum SHA-256 anti-manipulación
- Detecta early adopters (instalados durante período gratuito)
- Detecta región del usuario (Colombia/Latinoamérica/Internacional)
- Calcula días desde lanzamiento, retorna estado actual + precios
- Registra historial de últimas 100 sesiones

### BLOQUE 4 — Scanner Universal ✅
Clase `ScannerUniversal`:
- Recorre proyecto, filtra carpetas/archivos ignorados
- Categoriza: codigo/datos/web/docs/excel/scripts/cloud
- Detecta lenguaje por extensión
- Lectura segura con truncado por tamaño
- Genera árbol visual ASCII
- Filtros por categoría y por lenguaje

### BLOQUE ORQ — Orquestador (renombrado en Sesión 2) ✅
Clase `StratumApp`:
- Coordina todos los módulos
- Banner con tier, estado, región, SHA-256
- Servidor HTTP local con cabeceras de seguridad
- Dashboard HTML expandido con tarjetas de los nuevos engines

---

## ✅ BLOQUES COMPLETADOS — SESIÓN 2

### BLOQUE 5 — AST Engine Python ✅
Clase `ASTEnginePython`:
- `analizar()` → punto de entrada, dos pasadas (parseo + cross-archivo)
- `_analizar_archivo(ruta)` → parsea con `ast.parse`, captura `SyntaxError`
- `_extraer_imports(tree)` → imports + from..imports + alias
- `_extraer_clases(tree)` → clases top-level + métodos + decoradores
- `_extraer_funciones_top_level(tree)` → solo funciones nivel módulo
- `_inspeccionar_funcion(node)` → firma completa + complejidad + llamadas
- `_calcular_complejidad(node)` → estilo radon: If/For/While/IfExp/BoolOp/comprensiones/Match
- `_extraer_referencias(tree)` → Names + Attributes para detección de no-usados
- `_detectar_no_usados_global()` → pasada 2: imports/funciones nunca usados + complejidad >10
- Output: `{archivo: {imports, clases, funciones, problemas, complejidad_total}}`
- Cero dependencias externas — solo `ast` stdlib

### BLOQUE 6 — Heuristic Engine multi-lenguaje ✅
Clase `HeuristicEngine`:
- `_limpiar_codigo(texto, lang)` → enmascara strings + elimina comentarios ANTES de regex
- `_parsear_js_ts()` → import/require/export, class/interface, function/arrow
- `_parsear_java()` → import, class/interface/enum, métodos, anotaciones
- `_parsear_csharp()` → using, namespace, class/struct/record, métodos
- `_parsear_php()` → use/require, class/interface/trait, function
- `_parsear_r()` → library/source, funciones `nombre <- function()`
- `_extraer_bloque_balanceado(texto, inicio)` → encuentra `{...}` respetando anidación
- `_extraer_metodos_*()` → métodos por lenguaje sobre cuerpo de clase ya extraído
- Solo declaraciones de nivel superior y nivel 1 de anidamiento
- Output con la misma forma que ASTEnginePython para consistencia

### BLOQUE 7 — SQL Parser multi-dialecto ✅
Clase `SQLParser`:
- `_limpiar_sql(texto)` → quita `--` y `/* */` antes de cualquier extracción
- `_detectar_dialecto(texto)` → infiere postgresql/supabase/mysql/sqlserver/sqlite
- `_extraer_tablas(texto)` → CREATE TABLE + columnas + tipos + nullability
- `_parsear_columnas_tabla(cuerpo)` → distingue columnas de constraints, FK inline + constraint
- `_extraer_vistas/indices/funciones/triggers/secuencias()` → DDL completo
- `_extraer_policies(texto)` → CREATE POLICY de Supabase/PostgreSQL
- `_extraer_fks_alter(texto)` → ALTER TABLE ADD CONSTRAINT FOREIGN KEY
- `_extraer_dml(texto)` → cuenta SELECT/INSERT/UPDATE/DELETE por tabla
- Identificadores entrecomillados: `"x"` (PG), `` `x` `` (MySQL), `[x]` (MSSQL)
- `_extraer_parens_balanceados()` y `_split_top_level()` para parsing robusto
- Output: `{tablas, vistas, funciones, fks, policies, dml}` + catálogo global

### BLOQUE 8 — Code-DB Linker ✅
Clase `CodeDBLinker`:
- Cruza catálogo de tablas (del SQL Parser) con código fuente
- NUNCA busca nombres pelados (evita falsos positivos con palabras como `users`, `data`)
- Patrones SQL crudos: `FROM/JOIN/INTO/UPDATE/DELETE FROM` con identificadores entrecomillados
- Patrones Python: `__tablename__`, SQLAlchemy `Table()`, Django `db_table`, Supabase `from_()`, psycopg `execute()`
- Patrones JS/TS: Supabase `.from()`, Knex `knex()`, Sequelize `define()`, Prisma `prisma.X.`
- Patrones Java: `@Table(name=...)`, `@Entity`
- Patrones C#: EF Core `[Table(...)]`, `DbSet<X>`
- Patrones PHP: Eloquent `$table`, `DB::table()`
- Patrones R: `dbReadTable()`, `tbl()`
- `_es_nombre_tabla_valido()` filtra palabras reservadas (where, group, order, etc.)
- Output: `tabla_a_archivos`, `archivo_a_tablas`, `tablas_huerfanas_db`, `tablas_huerfanas_codigo`

### Actualizaciones del Orquestador ✅
- Nuevos atributos: `resultado_ast`, `resultado_heur`, `resultado_sql`, `resultado_link`
- Caché en `~/.stratum/cache/` (decisión Sesión 2)
- `ejecutar_analisis()` orquesta los 4 engines en orden, con prints de progreso
- 4 nuevos métodos `_html_seccion_*()` para tarjetas del dashboard

### Actualizaciones del Dashboard HTML ✅
- Badge: "v1.0.0 · Sesión 2 · Módulos 0–8 activos"
- Sección AST Python: archivos, clases, funciones, complejidad, problemas, top 5 más complejos
- Sección Heuristic: tabla por lenguaje (archivos/clases/funciones/imports)
- Sección SQL: tablas/vistas/funciones/FKs/RLS + lista de tablas detectadas
- Sección Linker: tablas definidas/usadas/huérfanas + top 10 más referenciadas

---

## 🔧 PRUEBA EXITOSA SESIÓN 2

```bash
# Hash actualizado
python3 stratum.py --hash
# SHA-256: e36f24f38af47d677dea41afe0958e48f833cb8de92eeb296d4beea35a6de114

# Análisis sobre proyecto de prueba con .py + .sql + .js + .java
python3 stratum.py --ruta /tmp/proyecto_test
# AST: 1 clase, 2 funciones, 11 complejidad, 4 problemas detectados
# Heuristic: JS (1 clase, 3 funciones), Java (1 clase, 1 método)
# SQL: 3 tablas, 1 vista, 2 FKs, 1 política RLS
# Linker: 3 definidas, 3 usadas, 1 huérfana correctamente identificada
# Sin warnings, sin errores
```

---

## ✅ BLOQUES COMPLETADOS — SESIÓN 3

### BLOQUE 9 — ETL/ELT Detector ✅
Clase `ETLDetector`:
- `analizar()` → detecta archivos Python/SQL/YAML con patrones ETL
- `_analizar_python_etl()` → Airflow (`@dag`, `DAG(`, `@task`), Prefect (`@flow`, `@task`), Dagster (`@asset`, `@job`), pandas/polars transformaciones
- `_analizar_sql_etl()` → dbt `ref()`/`source()`, CTEs (3+ = pipeline complejo)
- `_analizar_yaml_etl()` → dbt `schema.yml` (models/sources/columns)
- Agrupa pipelines con nombre, framework, n_tareas y dependencias
- Output: `{n_pipelines, n_archivos_etl, frameworks_detectados, pipelines[]}`

### BLOQUE 10 — Cloud Auditor ✅
Clase `CloudAuditor`:
- `_analizar_terraform()` → parsea bloques `resource`, infiere proveedor (aws/gcp/azure/cloudflare/supabase/digitalocean/hetzner)
- `_analizar_dockerfile()` → FROM, EXPOSE, ENV, USER, COPY/ADD (alertas no-root)
- `_analizar_generic_cloud()` → CloudFormation (`AWSTemplateFormatVersion`), Kubernetes (`apiVersion`)
- `_escanear_credenciales()` → 8 patrones regex: AWS Access Key ID, AWS Secret Key, GitHub Token, Private Key Block, Generic API Key, Password Hardcoded, Connection String DB, Slack/Discord Token
- `_inferir_proveedor()` → mapea tipo de recurso Terraform a proveedor
- Output: `{n_recursos_cloud, proveedores[], credenciales_expuestas[], recursos[], alertas_seguridad[]}`

### BLOQUE 11 — DWH Auditor ✅
Clase `DWHAuditor`:
- Recibe catálogo SQL del SQLParser (tablas + columnas)
- Clasifica tablas: `dim_*/dimension_*`, `fact_*/fct_*`, `stg_*/staging_*`, `mart_*`, `raw_*`
- `_analizar_dimension()` → detecta SK (surrogate key), NK (natural key), indicadores SCD
- `_analizar_fact()` → detecta métricas (cantidad/monto/total/precio), FKs, grain de fecha
- `_detectar_scd()` → SCD1 (updated_at/modified_at), SCD2 (valid_from/valid_to/is_current/dw_end), SCD3 (prev_/previous_/_anterior)
- `_detectar_tipo_schema()` → `star_schema` si ≥2 dims + ≥1 fact, `dimensional_simple` si solo dims
- `_calcular_score()` → 0-100 puntos: dims sin SK (-15), facts sin FKs (-20), SCD sin tipo claro (-10)
- Output: `{n_dims, n_facts, n_staging, tipo_schema, score_calidad, problemas[], dims[], facts[]}`

### BLOQUE 12 — Data Profiler ✅
Clase `DataProfiler`:
- Solo .csv y .json (archivos .parquet requieren pandas/polars, no incluidos en core)
- Skippea archivos > 50 MB
- `_perfilar_csv()` → usa stdlib `csv.DictReader`, límite `MAX_FILAS_ANALISIS = 10_000`
- `_perfilar_json()` → solo arrays de objetos JSON (descarta escalares/nested complejo)
- `_perfilar_columna()` → por columna: total, nulos, únicos, tipo inferido, PII, estadísticas, SQL sugerido
- `_inferir_tipo()` → umbral 60%: bool → int → float → date → email → text
- `_detectar_pii()` → mapeo de nombre de columna a categoría PII: email, telefono, cedula, nombre, direccion, ip, fecha_nac, salario + regex de validación
- `_estadisticas_numericas()` → min/max/mean/IQR + detección de outliers (método IQR)
- `_sugerir_tipo_sql()` → VARCHAR(n), BOOLEAN, BIGINT, DECIMAL(18,4), DATE, TEXT, etc.
- Output: `{n_archivos, n_archivos_con_pii, alertas_pii[], perfiles[]}`

### Actualizaciones del Orquestador ✅
- Nuevos atributos: `resultado_etl`, `resultado_cloud`, `resultado_dwh`, `resultado_profiler`
- `ejecutar_analisis()` llama los 4 nuevos engines en orden después de Block 8
- DWHAuditor recibe catálogo del SQLParser; DataProfiler solo procesa .csv y .json

### Actualizaciones del Dashboard HTML ✅
- Badge actualizado: "v1.0.0 · Sesión 3 · Módulos 0–12 activos"
- `_html_seccion_etl()` → tarjeta con pipeline count por framework, tabla de pipelines detectados
- `_html_seccion_cloud()` → inventario por proveedor + tabla de alertas de seguridad (rojo si hay credenciales expuestas)
- `_html_seccion_dwh()` → tarjetas dim/fact/staging, tipo de schema, score de calidad con barra de progreso, lista de problemas
- `_html_seccion_profiler()` → resumen de archivos perfilados, alertas PII en rojo, tabla de columnas por archivo

### Correcciones de sintaxis aplicadas ✅
- Python 3.10 no permite backslashes ni comillas del mismo tipo dentro de `{...}` en f-strings
- Solución: precomputar bloques HTML condicionales como variables antes del f-string (`sec_probs`, `sec_pii`, `rows_alert`)

---

## 🔧 PRUEBA EXITOSA SESIÓN 3

```bash
# Verificación sintaxis — OK
python3 -c "import ast; ast.parse(open('stratum.py').read()); print('SINTAXIS OK')"
# → SINTAXIS OK

# Hash actualizado Sesión 3
# SHA-256: 47df561c52c63d9f09d97dbb4f58dfff6872f3ee6b3eed065f670a7a69eda2fc

# Líneas totales del archivo
wc -l stratum.py
# → 4493 líneas
```

---

## ✅ BLOQUES COMPLETADOS — SESIÓN 4

**CAMBIO DE PLAN EJECUTADO:**
- ML Detector+Generator eliminado del core (requería sklearn/prophet)
- Reemplazado por Bloque 16 FlowAnalyzer (Business Process Reconstructor)
- Pattern Intelligence System documentado → entra completo en Sesión 5-6

### Fix Bloque 12 — PII Detector (bug crítico) ✅
- Causa raíz: detectaba PII solo por nombre de columna sin validar contenido
- Pipeline de 3 pasos implementado:
  1. Filtrar columnas residuales `Unnamed:*` y vacías
  2. Heurística nombre + dtype (solo candidatos, no retorna todavía)
  3. Validar con regex sobre muestra real de valores (umbral 70%)
- `PRECIO UNITARIO` ya no clasifica como cédula
- `NOMBRE` de materiales ya no clasifica como PII personal
- Requiere patrón de 2 palabras capitalizadas para "nombre_persona"

### Bloque Transversal — Framework Primary Detector ✅
Clase `FrameworkDetector`:
- Detecta framework dominante: Streamlit, FastAPI, Flask, Django, Dash, Gradio, Airflow, React, NextJS, Vue, Angular, Express, Spring, ASP.NET
- Lee manifiestos de versión: `pom.xml`, `package.json`, `tsconfig.json`, `pyproject.toml`, `.python-version`, `.csproj`, `composer.json`
- Ajusta umbrales por framework: función >500 líneas en Streamlit = CRÍTICO; 0 clases en Streamlit = ✅ correcto
- Output: `{framework, lenguaje_dominante, version_info, umbrales, nota}`

### Bloque 13 — KPI Engine ✅
Clase `KPIEngine`:
- Categorías: financiero / operativo / temporal / calidad
- Detecta agregaciones: `.groupby`, `.agg`, `.sum/.count/.mean`, SQL `SUM/AVG/GROUP BY`
- Detecta `st.metric()`, `go.Indicator`, `px.bar/line/scatter`
- JS: detecta `.reduce/.filter/.map` como señales de cómputo de métricas
- Output: `{n_archivos_con_kpi, total_kpis_detectados, total_agregaciones, categorias, top_archivos_kpi}`

### Bloque 14 — Viz Mapper ✅
Clase `VizMapper`:
- 8 librerías soportadas: matplotlib, plotly, seaborn, altair, bokeh, streamlit, Chart.js, D3.js
- Detecta tipo de gráfico: estático / interactivo / estadístico / declarativo / web
- Output: `{total_visualizaciones, n_archivos_con_viz, librerias_usadas, por_libreria, top_archivos}`

### Bloque 15 — Query Intel ✅
Clase `QueryIntel`:
- Detecta N+1: query dentro de bucle for/while/forEach (severidad CRÍTICO)
- Detecta SELECT * en SQL
- Detecta `.all()/.execute()` sin `.limit()` (severidad ALTO)
- Detecta subqueries correlacionadas SQL (severidad ALTO)
- Soporta Python ORM, JS/TS (Prisma, Sequelize), SQL puro
- Output: `{hallazgos[], resumen: {total, por_severidad, por_tipo}}`

### Bloque 16 — FlowAnalyzer (Business Process Reconstructor) ✅
Clase `FlowAnalyzer` — joya de la corona:
- **Shared State Registry**: clave `(valor_estado)` → `{writers[], readers[]}`; clave framework-agnóstica
- **14 patrones de escritura**: dict `{"estado": "Valor"}`, asignación `.estado = "Valor"`, `.update({...})`, SQL `SET estado=`, Django `.update(estado=)`, Supabase
- **6 patrones de lectura**: `.eq("estado", "Valor")`, `.filter(estado=)`, SQL `WHERE estado=`, `.in_()`, comparación `== "Valor"`, Prisma/Sequelize
- **Detección de flujos**: reconstrucción automática de cadenas de transición ModA→ValX→ModB→ValY→ModC
- **Detección de GAPS**: estados escritos que ningún módulo externo consume (flujo roto)
- **Detección de BYPASSES estructurales**: rutas de código que omiten estado intermedio
- **Mermaid.js**: genera sintaxis flowchart LR automáticamente; gaps en rojo (#fc8181)
- Funciona en Python, JS/TS, Java, C#, PHP, R, SQL — framework agnóstico
- Output: `{registry, flujos[], gaps[], bypasses[], mermaid, resumen}`

### Actualizaciones del Orquestador ✅
- 5 nuevos atributos: `resultado_framework`, `resultado_kpi`, `resultado_viz`, `resultado_query`, `resultado_flow`
- `ejecutar_analisis()` llama los 5 engines después del Bloque 12
- Mermaid.js cargado via CDN `cdn.jsdelivr.net` en el HTML del dashboard

### Actualizaciones del Dashboard HTML ✅
- Badge: "Sesión 4 · Módulos 0–16 activos"
- `_html_seccion_framework()` → framework detectado + umbrales ajustados
- `_html_seccion_kpi()` → categorías de KPI + top archivos por densidad
- `_html_seccion_viz()` → inventario de visualizaciones por librería
- `_html_seccion_query()` → hallazgos de query con severidad coloreada
- `_html_seccion_flow()` → flujos reconstruidos + gaps en rojo + grafo Mermaid

---

## 🔧 PRUEBA EXITOSA SESIÓN 4

### ⚠️ HOTFIX CRÍTICO — Motores leen archivos desde disco

**Causa raíz:** El Scanner Universal NO almacena `contenido` en el registro de archivos.
Los 6 motores de Sesión 4 usaban `arch.get("contenido", "") or ""` → siempre vacío.

**Motores corregidos:**
- `FrameworkDetector.detectar()` → ahora usa `open(arch["ruta_abs"], "r", ...)`
- `FrameworkDetector._detectar_versiones()` → ídem para manifiestos
- `KPIEngine.analizar()` → ídem
- `VizMapper.analizar()` → ídem
- `QueryIntel.analizar()` → ídem
- `FlowAnalyzer.analizar()` → ídem

**Patrón correcto (igual que HeuristicEngine):**
```python
ruta_abs = arch.get("ruta_abs", "")
if not ruta_abs:
    continue
try:
    with open(ruta_abs, "r", encoding="utf-8", errors="replace") as _f:
        contenido = _f.read()
except Exception:
    continue
```

**REGLA PARA SESIONES FUTURAS:** Todo motor que necesite leer contenido de archivos
DEBE abrir desde `arch["ruta_abs"]`. El Scanner solo entrega metadatos, no contenido.

---

```bash
# Verificación sintaxis — OK
python3 -c "import ast; ast.parse(open('stratum.py').read()); print('SINTAXIS OK')"
# → SINTAXIS OK

# Hash actualizado Sesión 5
# SHA-256: 559b80efba01a6a6c61175f60460d446d1c397f545afd4c00949b0b2336178b7

# Líneas totales del archivo
wc -l stratum.py
# → 7782 líneas
```

---


---

## ✅ BLOQUES COMPLETADOS — SESIÓN 5

### FIX PRIORITARIO — Framework-Aware Thresholds ✅
- `ASTEnginePython` ahora acepta `umbrales_fw` del `FrameworkDetector`
- Para Streamlit (y frameworks funcionales): umbral ciclomático = 20 (antes 10)
- Check de longitud de función: >500 líneas = CRÍTICO, >250 = ALTO (Streamlit)
- Check de longitud de función: >300 líneas = CRÍTICO, >150 = ALTO (generic)
- FrameworkDetector se ejecuta PRIMERO en el orquestador y pasa umbrales al AST
- Dashboard: Complejidad total ahora muestra promedio por función y umbral usado
- **Regla permanente**: umbral ciclomático alto en frameworks funcionales (Streamlit/FastAPI/Flask)

### BLOQUE 17 — Excel Intelligence ✅
- Clase `ExcelIntelligence`: audita .xlsx, .xls, .xlsm, .csv, .tsv, .ods
- Con openpyxl: fórmulas volátiles, referencias externas, hojas ocultas, macros VBA
- Sin openpyxl: análisis básico + nota de instalación
- CSV: detección de encoding incorrecto (mojibake), separador, archivos grandes
- Sección HTML en dashboard con detalle por archivo

### BLOQUE 18 — STRATUM Score ✅
- Clase `StratumScore`: score 0–100 ponderado por severidad
- Pesos: CRÍTICO=8, ALTO=4, MEDIO=2, BAJO=1
- Badges: EXCELENTE (≥90), BUENO (≥70), ATENCIÓN (≥50), DEFICIENTE (≥30), CRÍTICO (<30)
- Consolida findings de TODOS los bloques (AST, SQL, ETL, Cloud, DWH, Query, Flow, Excel)
- Top 10 findings priorizados por severidad
- Sección HTML prominente en el dashboard (card 2rem, color por badge)

### BLOQUE 19 — Quick Fix Engine ✅
- Clase `QuickFixEngine`: recetas de fix para cada tipo de finding
- Estimación de esfuerzo: MINUTOS / HORAS / DÍAS
- Snippets de código para: funcion_muy_larga, complejidad_alta, n_plus_1, select_star, etc.
- Prioriza por ratio impacto/esfuerzo (CRÍTICO+MINUTOS primero)
- Quick Wins: lista de fixes que toman minutos
- Sección HTML con tabla de todas las sugerencias y quick wins destacados

### BLOQUE 20 — Pattern Intelligence Capa 1 (Coverage Score) ✅
- Clase `CoverageScoreEngine`: ratio estructuras detectadas / señales brutas
- Umbral: < 60% → advertencia de cobertura insuficiente
- Soporta Python, JS, TS, Java, C#, PHP, R
- Señales brutas: keywords `def`, `class`, `function`, `=>`, etc.
- Sección HTML con ratio global y archivos con cobertura baja

### BLOQUE 21 — Pattern Intelligence Capa 2 (Language Version Detection) ✅
- Clase `LanguageVersionDetector`: lee manifiestos del proyecto
- Soporta: pyproject.toml, .python-version, package.json, tsconfig.json,
  pom.xml, build.gradle, composer.json, .csproj, global.json, Pipfile, .nvmrc
- Cruza con versiones soportadas y emite advertencias para versiones nuevas
- Sección HTML con tabla de versiones detectadas y advertencias de cobertura

---

## ✅ BLOQUES COMPLETADOS — SESIÓN 6

### FIXES CRÍTICOS ✅

#### Fix #23 — PEN_MAX Dinámico (Score=0 en proyectos grandes)
- **Causa raíz:** `_PEN_MAX = 200` fijo → 527 CRÍTICOs × 8 = 4216 >> 200 → score=0
- **Solución:** PEN_MAX dinámico: `max(200, (total_funciones + total_metodos) × 8)`
- **Semántica:** score=0 cuando CADA función tiene al menos 1 hallazgo CRÍTICO
- **Expuesto en output:** `pen_max` visible en resultado para debugging
- Ejemplo QS Ingeniería: 400 units → pen_max=3200; score calibrado meaningful

#### Fix #24 — Mermaid.js CDN + Fallback
- **Causa raíz:** `<script src="cdn.jsdelivr.net">` + `startOnLoad` fallaba silenciosamente en archivos locales
- **Solución:** ES module async import + fallback a `<pre>` si CDN no responde (2s timeout)
- Mensaje de aviso: "Diagrama requiere internet para renderizar"

#### Fix #25 — Texto Truncado en Tablas
- **Causa raíz:** slices duros `[:70]`, `[:40]`, `[:35]`, `[:30]` en `rows_top` y `rows_qw`/`rows_todas`
- **Solución:** CSS classes `.detail-cell` (max-width:320px, word-break) y `.file-cell` (word-break:break-all)
- Texto completo accesible, CSS se encarga del wrapping

### BLOQUE 22 — Streamlit Cache Detector ✅
Clase `StreamlitCacheDetector`:
- Solo activo cuando `FrameworkDetector` detecta Streamlit como framework dominante
- Detecta funciones con nombre "costoso" (cargar/load/fetch/query/compute) sin `@st.cache_data`
- Detecta `@st.experimental_memo`, `@st.experimental_singleton`, `@st.cache` (deprecados desde 1.18)
- Detecta acceso a `st.session_state["key"]` sin `.get()` ni verificación `"key" in st.session_state`
- Detecta iteración `for x in st.session_state.items()` sin copia (RuntimeError)
- Detecta conexiones DB (`psycopg2.connect`, `create_engine`, `MongoClient`, etc.) fuera de `@st.cache_resource`
- Detecta `@st.cache_data` con argumentos mutables (`[]` o `{}`)
- Output: `{hallazgos[], resumen: {sin_cache_n, deprecated_n, session_state_n}}`
- Conectado a StratumScore findings con severidad propia

### BLOQUE 23 — Security Auditor ✅
Clase `SecurityAuditor`:
- Sin dependencias externas — análisis por patrones regex
- **Secretos hardcodeados:** password, api_key, token, AWS AKIA*, JWT (3 segmentos base64), DATABASE_URL con credenciales, SECRET_KEY real
- **Inyección SQL:** f-strings con SQL, `.format()` sobre queries, `%` sobre queries, `cursor.execute(query + user_input)`
- **Código inseguro:** `eval()` (CRÍTICO), `exec()`, `subprocess(shell=True)` (CRÍTICO), `os.system()`, `os.popen()`, `__import__()`
- **Deserialización insegura:** `pickle.loads()` (CRÍTICO), `marshal.loads()` (CRÍTICO), `yaml.load()` sin Loader, `jsonpickle.decode()`
- **Config expuesta:** `DEBUG=True`, `ALLOWED_HOSTS=["*"]`, `CORS_ORIGIN_ALLOW_ALL=True`
- Filtros anti-falso-positivo: ignora líneas con test_, _test, example, placeholder, todo:, fixme:
- Escaneado: todos los .py + .cfg + .ini + .env + .yaml + .yml + .toml + .json
- Settings especiales: DEBUG y CORS solo en archivos settings.py/config.py/secrets.py/etc.
- Hallazgos alimentan StratumScore con severidad propia

### BLOQUE 28 — Report Generator ✅
Clase `ReportGenerator`:
- Genera `stratum_reporte_YYYY-MM-DD.html` en la raíz del proyecto analizado
- HTML standalone con CSS print-friendly (`@media print`) — sin CDN
- **Portada:** nombre proyecto, fecha, score/badge con color
- **Resumen ejecutivo:** 6 tarjetas (Score, Framework, Archivos, Funciones, Seguridad, Cobertura) + barras de severidad
- **Top 10 hallazgos** con severidad coloreada
- **Recomendaciones** (Quick Fix top 5) con estimación de esfuerzo
- **Estado de módulos** (17 módulos con Activo/No aplica + conteo hallazgos)
- Se ejecuta automáticamente al final de `ejecutar_analisis()`; error no bloquea
- Output: `{ruta_reporte, resumen: dict métricas}`

### Actualizaciones del Orquestador ✅
- `resultado_security`: hallazgos SecurityAuditor → alimenta StratumScore
- `resultado_st_cache`: hallazgos StreamlitCacheDetector → alimenta StratumScore (solo si Streamlit)
- `resultado_reporte`: ruta HTML + métricas del ReportGenerator
- StratumScore recibe `"security"` y `"st_cache"` en su dict de resultados
- `_recolectar_findings()` extrae hallazgos de SecurityAuditor y StreamlitCacheDetector

### Actualizaciones del Dashboard HTML ✅
- `_html_seccion_security()`: nivel de riesgo, alertas inmediatas (secretos/SQL injection), tabla top 15
- `_html_seccion_st_cache()`: solo visible si proyecto Streamlit; contadores sin @cache/deprecated/session_state
- CSS agregado: `.detail-cell`, `.file-cell` para texto wrap sin truncado
- Mermaid: ES module async con fallback a `<pre>` formateado
- Badge: "v1.0.0 · Sesión 6 · Módulos 0–23 activos"

---

## 🔧 VERIFICACIÓN SESIÓN 6

```bash
# Sintaxis OK
python3 -m py_compile stratum.py
# → Sin errores

# SHA-256 Sesión 6
# ff778261ef0c51f74e37a6f778c302f612654e053cdb7163f80934f6fc7d2f9a

# Líneas totales
wc -l stratum.py
# → 8867 líneas

# Self-test SecurityAuditor
python3 -c "
import importlib.util, tempfile, os
spec = importlib.util.spec_from_file_location('s', 'stratum.py')
m = importlib.util.module_from_spec(spec); spec.loader.exec_module(m)
code = 'api_key = \"sk-live-abc123xyz\"\npickle.loads(data)\neval(x)\n'
with tempfile.NamedTemporaryFile(mode='w', suffix='.py', delete=False) as f:
    f.write(code); tmp=f.name
r = m.SecurityAuditor([{'ruta_abs':tmp,'ruta_rel':'t.py','extension':'.py'}]).analizar()
os.unlink(tmp)
print(r['resumen'])
# → {'total_hallazgos': 3, 'nivel_riesgo': 'ALTO', 'tiene_secretos': True, ...}
"

# PEN_MAX dinámico
python3 -c "
import importlib.util
spec = importlib.util.spec_from_file_location('s', 'stratum.py')
m = importlib.util.module_from_spec(spec); spec.loader.exec_module(m)
r = m.StratumScore({'ast':{'archivos':{},'resumen':{'total_funciones':300,'total_metodos':100,'total_clases':0}}}).calcular()
print(f'pen_max={r[\"pen_max\"]}')  # → 3200 (400 × 8)
"
```

---

## ⛔ STOP — HANDOFF A OPUS 4.7

**SESIÓN 6 COMPLETADA.** A partir de aquí usar **Claude Opus 4.7** para continuar el desarrollo.

**Instrucción para Sesión 7 (Opus 4.7):**
```
Continuamos STRATUM System Intelligence Engine v1.0.0.
Sesión 7. Lee STRATUM_ESTADO.md primero.
Bloques pendientes: 24 (Dependency Scanner), 25 (Performance Profiler),
26 (Documentation Score), 27 (Test Coverage Detector),
Pattern Intelligence Capa 3 (patrones externos), UI D3.js.
```

**Razón del handoff:** Opus 4.7 tiene mayor ventana de contexto y capacidad
de razonamiento para las sesiones finales de v1.0.0 con bloques de mayor complejidad.

---

## 🚧 BLOQUES PENDIENTES — SESIÓN 7+

**PENDIENTES para Opus 4.7:**
- Bloque 24: Dependency Scanner (requirements.txt, package.json, CVEs conocidos)
- Bloque 25: Performance Profiler (N+1 avanzado, índices faltantes, queries sin límite)
- Bloque 26: Documentation Score (docstrings, README, cobertura de comentarios)
- Bloque 27: Test Coverage Detector (archivos test_, pytest, unittest, coverage.py)
- Pattern Intelligence Capa 3: patrones externos ~/.stratum/patterns/ + --update-patterns
- UI completa D3.js (reemplaza _generar_html_basico con dashboard interactivo)

---

## 🚧 BLOQUES PENDIENTES — SESIÓN 6 (ORIGINAL — TODOS COMPLETADOS)

**INSTRUCCIÓN PARA SESIÓN 6:**
Pegar este documento + el código actual de `stratum.py`
Luego escribir: "Continuamos STRATUM. Construimos Sesión 6: Bloques 22-28"

**⚠️ STOP EN SESIÓN 6:** Al terminar Sesión 6, hacer pausa para handoff a Opus 4.7

**COMPLETADOS EN SESIÓN 6:**
- ✅ Fix Score=0 (PEN_MAX dinámico)
- ✅ Fix Mermaid raw text (ES module + fallback)
- ✅ Fix texto truncado (CSS detail-cell/file-cell)
- ✅ Bloque 22: Streamlit Cache Detector
- ✅ Bloque 23: Security Auditor
- ✅ Bloque 28: Report Generator

## 🚧 BLOQUES PENDIENTES — SESIÓN 5

**INSTRUCCIÓN PARA SESIÓN 5:**
Pegar este documento + el código actual de `stratum.py`
Luego escribir: "Continuamos STRATUM. Construimos Sesión 5: Bloques 17-21"

**PENDIENTES TRASLADADOS:**
- Pattern Intelligence System Capa 1 (Coverage Score en HeuristicEngine) → Sesión 5
- Pattern Intelligence System Capa 2 (Language Version Detection en Scanner) → Sesión 5
- Detector de caching faltante Streamlit (extensión Bloque 9) → Sesión 5

---

## 🏗️ DECISIONES ARQUITECTÓNICAS — SESIÓN 3 (análisis, no código)

### Decisión 1 — FlowAnalyzer / Business Process Reconstructor ✅ APROBADO
**Concepto:** La "joya de la corona" de STRATUM. Reconstruye automáticamente flujos de negocio desde el código sin que nadie explique qué es el flujo. Ninguna herramienta del mercado hace esto.

**Algoritmo (4 pasos):**
1. **Descubrimiento de estados:** escanear todo el código buscando strings literales asignados a campos de tipo `estado/status/step/etapa/phase/stage`. Construir inventario por entidad y tabla.
2. **Mapeo productor/consumidor:** para cada valor de estado, identificar qué módulo lo ESCRIBE (.update, .insert, asignación =) y qué módulo lo LEE (.eq, .filter, WHERE, if == valor). Clave del registro: `(nombre_tabla, nombre_campo)` — framework-agnóstico.
3. **Construcción del grafo dirigido:** una transición = módulo A produce estado S → módulo B consume estado S y produce estado S2. Encadenar automáticamente → flujo reconstruido. Nombrar el flujo automáticamente desde el nombre de la tabla.
4. **Diagnóstico de consistencia:** verificar que cada módulo consumidor consulte la tabla correcta, detectar estados huérfanos (escritos, nunca leídos), estados fantasma (leídos, nunca escritos), y bypasses estructurales (ruta de código que omite un estado intermedio sin validarlo).

**Output esperado:**
```
Flujo detectado: REQUISICIÓN — 5 estados — 4 módulos — flujo lineal
  ✓ ingeniero.py → [Pendiente Jefe Compras] → jefe_compras.py
  ✓ jefe_compras.py → [Pendiente Asistente] → asistente_compras.py
  ✓ asistente_compras.py → [Pendiente Gerente] → gerencia.py
  ✓ gerencia.py → [Aprobado] → genera orden_compra
  ⚠ PROBLEMA: estado "Pendiente Validación Jurídica" escrito en os_ajustes
    por asistente_compras.py — ningún módulo lo consume. FLUJO ROTO.
```

**Generalidad:** funciona para Python (Supabase, SQLAlchemy, Django ORM, raw SQL), JS/TS (Prisma, Sequelize, Supabase JS), Java (JPA, JDBC), C# (Entity Framework), PHP (Eloquent), R (dbWriteTable). El Shared State Registry usa claves `(tabla, campo)` independientes del ORM.

**Visualización:** Mermaid.js (no D3.js para este componente). STRATUM genera el texto Mermaid y el browser lo renderiza via CDN. Ejemplo: `A[ingeniero.py] -->|Pendiente Jefe Compras| B[jefe_compras.py]`. D3.js queda para analytics cuantitativos del dashboard general.

**Bypass detection:** Solo bypasses ESTRUCTURALES (rutas de código que pueden saltarse un estado). No bypasses de runtime (requeriría logs de BD). Cubre el 90% de los casos que importan.

**Ubicación en roadmap:** Sesión 4 (reemplaza ML Detector). Sesión 6 añade visualización D3.js del grafo como pantalla principal del dashboard.

---

### Decisión 2 — Pattern Intelligence System ✅ APROBADO
**Problema:** Si Java 25 lanza nueva sintaxis, los regex del HeuristicEngine no la detectan. Necesitamos que STRATUM sepa cuándo está fallando y pueda actualizarse.

**Solución en 3 capas:**

**Capa 1 — Detector de Cobertura (Sesión 4, mejora Bloque 6):**
El HeuristicEngine calcula antes de reportar: estructuras detectadas / señales estructurales brutas (keywords `class`, `function`, `def`, etc.). Si ratio < 60% → advertencia: `⚠️ Cobertura de análisis: 43% — posible sintaxis no reconocida o versión de lenguaje no soportada`. Nunca reporta datos incorrectos como correctos.

**Capa 2 — Detección de versión del lenguaje (Sesión 4, mejora Bloque 4 Scanner):**
Scanner lee archivos de configuración del proyecto:
- Java: `pom.xml` (<java.version>), `build.gradle`, `.java-version`
- TypeScript/JS: `tsconfig.json` (target), `package.json` (engines), `.nvmrc`
- C#: `.csproj` (TargetFramework), `global.json`
- PHP: `composer.json` (require php)
- Python: `pyproject.toml`, `.python-version`, `Pipfile`

Resultado: STRATUM aplica patrones versionados. Ejemplo: proyecto Java 21 usa `_PATRONES_JAVA_21` que incluye records, sealed classes, pattern matching en switch. Si detecta versión sin patrones propios, usa los más recientes y activa advertencia de cobertura.

**Capa 3 — Patrones externos + --update-patterns (Sesión 6, distribución):**
- `~/.stratum/patterns/java.json` tiene prioridad sobre patrones internos si existe
- Usuarios enterprise pueden agregar patrones para frameworks/DSLs propios
- Flag `--update-patterns`: única acción de red en STRATUM, completamente opt-in, descarga JSONs desde repositorio oficial a `~/.stratum/patterns/`
- NO viola principio de cero dependencias ni cero red por defecto

---

### Decisión 3 — Feedback de producción real (QS Ingeniería, ~13K LOC Streamlit)
Hallazgos de auditoría real que afectan bloques existentes:

**Bloque 12 PII — BUG CRÍTICO a corregir en Sesión 4:**
- Causa: detecta PII solo por nombre de columna, sin validar contenido → 80% falsos positivos
- Fix aprobado: pipeline de 3 pasos — (1) filtrar columnas `Unnamed:*`, (2) heurística por nombre+dtype, (3) validar con regex sobre muestra real de valores con umbral 70%. Solo si ≥70% de la muestra cumple el patrón se flaggea como PII.
- Columnas `NOMBRE` de productos/materiales NO se flaggean. Solo `nombre_cliente` con valores que matcheen patrón de nombre de persona.

**Detector de Framework Primario — nuevo paso 0 del orquestador (Sesión 4):**
- Antes de evaluar métricas, detectar framework dominante: Streamlit, FastAPI, Flask, Django, Express, Spring, genérico
- Ajusta interpretación: "0 clases" en Streamlit = ✓ (paradigma funcional correcto). "0 clases" en Django = ⚠️
- Streamlit tiene umbrales propios: función >500 líneas = CRÍTICO (Streamlit la re-ejecuta entera en cada click)
- Aplica a todos los lenguajes que STRATUM soporta, no solo Python

**Sistema de severidad ponderada (Sesión 5, STRATUM Score):**
- Reemplaza conteo plano de "90 problemas"
- 4 niveles: CRÍTICO / ALTO / MEDIO / BAJO con iconos diferenciados
- Cada finding incluye: diagnóstico, impacto estimado, acción sugerida, esfuerzo estimado

**Detector de caching faltante Streamlit (extensión Bloque 9, Sesión 4):**
- IO calls sin @st.cache_data = performance crítica en Streamlit
- Ratio IO/funciones_cacheadas > 5:1 = ALTO. Cero cache con >5 IO = CRÍTICO

---

### Decisión 4 — Roles del equipo de IA (definitivo)
- **Claude:** arquitecto y ejecutor. Decide qué entra, dónde y cómo. Genera TODO el código.
- **Gemini:** asesor externo. Aporta perspectiva, valida decisiones. NO genera código ni define arquitectura.
- **Javier:** product manager y dueño del producto. Prioriza, aprueba, hace pruebas reales.

---

## 🗓️ CRONOGRAMA COMPLETO (revisado en Sesión 3)

| Sesión | Bloques | Estado |
|---|---|---|
| Sesión 1 | Bloques 0-4: Núcleo, Seguridad, Licencias, Scanner | ✅ COMPLETO |
| Sesión 2 | Bloques 5-8: AST Python, Heuristic, SQL Parser, Code-DB Linker | ✅ COMPLETO |
| Sesión 3 | Bloques 9-12: ETL/ELT, Cloud Migration, DWH Auditor, Data Profiler | ✅ COMPLETO |
| Sesión 4 | Bloques 13-16: KPI Engine, Viz Mapper, Query Intel, FlowAnalyzer + mejoras transversales | ✅ COMPLETO |
| Sesión 5 | Bloques 17-21: Excel Intelligence, STRATUM Score, Quick Fix, Pattern Intelligence | ✅ Completa |
| Sesión 6 | Bloques 22-27: Reportes, UI D3.js + Mermaid, Claude Exporter, Pattern JSON, .bat/.sh, MSIX | ⏳ Pendiente |

---

## 📐 ESTÁNDAR DE CÓDIGO OBLIGATORIO

Todo código nuevo debe seguir este patrón sin excepción:

```python
# ════════════════════════════════════════════════════════════════════════════
# BLOQUE N — NOMBRE DEL BLOQUE EN MAYÚSCULAS
# Descripción clara de qué hace este bloque y por qué existe
# ════════════════════════════════════════════════════════════════════════════
class NombreClase:
    """
    Descripción de la clase.
    Responsabilidades:
      - Responsabilidad 1
      - Responsabilidad 2
    """
    # ─── Descripción del grupo de métodos ───────────────────────────────────
    def nombre_metodo(self, param: tipo) -> tipo_retorno:
        """
        Qué hace este método.
        Parámetros: descripción de params
        Retorna: descripción del retorno
        """
        # ─── Comentario interno del bloque lógico ───────────────────────────
        ...
```

---

## 🔐 ARQUITECTURA DE SEGURIDAD (NO CAMBIAR)

- Servidor: `127.0.0.1` por defecto, `0.0.0.0` solo con `--red` explícito
- Licencia: `~/.stratum/.stratum_license.dat` con checksum SHA-256
- Caché: `~/.stratum/cache/` (no contiene PII, solo análisis estructurales)
- Log: `~/.stratum/.stratum_audit.log`
- Zero imports de red en el código fuente (auditable)
- machine_id: hash de características del sistema, nunca datos personales
- Checksum licencia: SHA-256 con machine_id como salt

---

## 💰 MODELO DE NEGOCIO (NO CAMBIAR)

```
LAUNCH_DATE = "2025-12-01"  # Cambiar al día real de lanzamiento
FREE_PERIOD_DAYS = 365      # 12 meses gratis

Meses 1-10:  Completamente gratis, sin avisos
Meses 10-12: Gratis + banner "PRO llegará pronto"
Mes 13+:     Nuevos usuarios: freemium con límites
             Early adopters: 50% descuento PERMANENTE
             Excel Assistant: gratis SIEMPRE para todos

Precios Colombia:  Starter $35k/mes, Pro $89k/mes, Enterprise $199k/mes
Precios LATAM:     Starter $12/mes, Pro $29/mes, Enterprise $69/mes
Precios USA/EU:    Starter $29/mes, Pro $79/mes, Enterprise $199/mes
```

---

## 📦 ESTRATEGIA DE DISTRIBUCIÓN (decidida en Sesión 2)

**Canal 1 (mes 1, lanzamiento):** GitHub Releases + sitio web propio
- `.exe` empaquetado con Nuitka, SHA-256 publicado oficialmente
- Stripe internacional + Wompi/Mercado Pago para Colombia
- Cero comisión de plataforma, control total del licenciamiento

**Canal 2 (mes 2-3):** PyPI (`pip install stratum`)
- Para data engineers / devs Python que ya tienen Python instalado
- Refuerza posicionamiento técnico en LinkedIn

**Canal 3 (mes 3-4):** Winget + Chocolatey
- `winget install stratum` para usuarios Windows técnicos
- Trivial publicar, alta credibilidad

**Canal 4 (mes 6+):** Microsoft Store con MSIX
- Empaquetar el `.exe` con MSIX Packaging Tool
- Declarar `broadFileSystemAccess` (requiere aprobación manual)
- Canal de descubrimiento masivo, no canal principal
- Incluir `build_msix.ps1` + GitHub Action en Sesión 6

---

## 🤖 ROL DE CADA IA

**Claude (esta conversación) — cuenta principal de desarrollo:**
- Genera TODO el código de STRATUM
- Toma decisiones de arquitectura
- Ejecuta pruebas y verifica funcionamiento
- Actualiza este documento al final de cada sesión

**Claude (segunda cuenta — backup/segunda opinión):**
- Puede continuar si la principal se queda sin contexto
- Pega este .md + stratum.py para retomar

**Gemini (asesor externo):**
- Recibe este STRATUM_ESTADO.md para contexto
- Valida decisiones antes de implementar
- Segunda opinión en decisiones críticas
- NO genera código — solo asesora

---

## 🔑 HASH SHA-256 OFICIAL (publicar a clientes)

**Versión actual (Sesión 5 final — dual score + calibración):**
```
559b80efba01a6a6c61175f60460d446d1c397f545afd4c00949b0b2336178b7
```

**Versión anterior (Sesión 4 + hotfix contenido):**
```
1119f80db147c16fbd356f09fc9b0ecea04bfc049864c765d09c1ae846bf948a
```

**Versión anterior (Sesión 4 inicial):**
```
68ed8b16664929da5e67c2ba5ed0e05b0c62fff29233812796fb14a98594a3c0
```

**Versión anterior (Sesión 3):**
```
47df561c52c63d9f09d97dbb4f58dfff6872f3ee6b3eed065f670a7a69eda2fc
```

**Versión anterior (Sesión 2):**
```
e36f24f38af47d677dea41afe0958e48f833cb8de92eeb296d4beea35a6de114
```

Verificación por el cliente:
```bash
python3 stratum.py --hash
# Debe coincidir con el hash de arriba
```

---

## ⚠️ INSTRUCCIÓN ESPECIAL — SESIÓN 6

En la Sesión 6 se debe hacer un **STOP** para que Javier pueda pasar a un chat diferente con **Claude Opus 4.7** para las tareas finales de empaquetado, MSIX y distribución.

---

*Documento generado automáticamente al finalizar Sesión 4*
*Próxima acción: Copiar `stratum.py` + este `.md` al inicio de Sesión 5*
