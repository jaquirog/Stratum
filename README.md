# STRATUM — System Intelligence Engine

> Motor de análisis estático que entiende tu proyecto completo —código, base de datos, datos y flujos de negocio— en una sola pasada, sin enviar nada a la nube.

STRATUM analiza un proyecto de software de principio a fin y genera un dashboard y un reporte con hallazgos accionables: calidad de código, riesgos de seguridad, modelo de datos, pipelines ETL, infraestructura cloud y —su función estrella— la **reconstrucción automática de los flujos de negocio** a partir del propio código.

Pensado para desarrolladores, analistas de datos y consultores que necesitan entender o auditar un proyecto rápido. **STRATUM no reemplaza a la IA: la potencia**, entregándole contexto estructurado de un repositorio real.

## Características

**Análisis de código**
- AST de Python: complejidad ciclomática, imports y funciones sin usar, problemas detectados.
- Heurística multi-lenguaje: JavaScript/TypeScript, Java, C#, PHP, R.
- Detector de framework dominante (Streamlit, FastAPI, Django, Flask, React, Next.js, Spring…) con umbrales calibrados por framework.

**Base de datos y datos**
- Parser SQL multi-dialecto (PostgreSQL/Supabase, MySQL, SQL Server, SQLite): tablas, vistas, índices, FKs y políticas RLS.
- Code-DB Linker: cruza las tablas con el código y detecta tablas huérfanas.
- DWH Auditor: detecta esquemas estrella, dimensiones/hechos y SCD 1/2/3.
- Data Profiler: perfila CSV/JSON, infiere tipos, detecta PII y outliers.

**Procesos, calidad y seguridad**
- FlowAnalyzer: reconstruye automáticamente los flujos de negocio (estados y transiciones entre módulos), los dibuja en Mermaid y detecta flujos rotos y bypasses.
- Detección ETL/ELT (Airflow, Prefect, Dagster, dbt).
- Cloud Auditor (Terraform, Docker, Kubernetes) con escaneo de credenciales expuestas.
- Security Auditor: secretos hardcodeados, inyección SQL, `eval`/`exec`, deserialización insegura.
- Query Intel: N+1, `SELECT *`, queries sin límite.
- KPI Engine, Viz Mapper y Excel Intelligence.
- STRATUM Score (0–100) ponderado por severidad + Quick Fix Engine con estimación de esfuerzo.

**Salida**
- Dashboard HTML local e interactivo.
- Reporte HTML autónomo, listo para imprimir.

## Por qué es diferente

- **Cero dependencias** en el núcleo (solo la librería estándar de Python); `openpyxl` es opcional para Excel.
- **100% local y privado**: corre en `127.0.0.1`; sin llamadas de red en el núcleo (auditable).
- **Un solo archivo**: `stratum.py`. Fácil de revisar, distribuir y ejecutar.
- **Multi-lenguaje**: Python, JS/TS, Java, C#, PHP, R y SQL.
- **Integridad verificable**: STRATUM calcula su propio hash **SHA-256** para que cualquiera pueda comprobar que el archivo no fue alterado.

## Uso

```bash
# Analizar un proyecto
python3 stratum.py --ruta /ruta/a/tu/proyecto

# Ver el hash SHA-256 del propio script (verificación de integridad)
python3 stratum.py --hash

# Exponer el dashboard en la red local (opt-in; avisa antes de activarse)
python3 stratum.py --ruta /ruta/a/tu/proyecto --red
```

El dashboard queda disponible en `http://127.0.0.1:7474`. Al terminar, STRATUM deja un reporte `stratum_reporte_AAAA-MM-DD.html` en la raíz del proyecto analizado.

## Lenguajes soportados

Python · JavaScript / TypeScript · Java · C# · PHP · R · SQL · VBA / Excel

## Privacidad y seguridad

STRATUM no envía tu código a ningún servidor: todo el análisis ocurre en tu máquina. Los archivos de licencia, auditoría y caché viven en `~/.stratum/` y nunca contienen datos personales.

## Autor

**Javier Alfonso Quiroga Sarmiento** — Desarrollador Full-Stack & Analista de Datos/BI
GitHub: [@jaquirog](https://github.com/jaquirog)

## Licencia

Pendiente de definir. Para un proyecto de uso libre se recomienda una licencia **MIT** (añade un archivo `LICENSE`).
