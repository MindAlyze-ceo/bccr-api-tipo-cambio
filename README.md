# BCCR API Tipo de Cambio

Proyecto en Python para consultar indicadores económicos del Banco Central de Costa Rica mediante el API del Sistema de Divulgación de Datos Económicos (SDDE).

El objetivo principal es sustituir el consumo anterior mediante Web Service XML por una consulta REST API en formato JSON, permitiendo consultar indicadores económicos por código, transformar la información y guardar los resultados en una base SQLite.

---

## Objetivo del proyecto

Este proyecto permite:

- Validar la suscripción del usuario ante el API del BCCR.
- Consultar indicadores económicos mediante código.
- Solicitar fecha inicial y fecha final.
- Convertir la respuesta JSON del BCCR en una tabla diaria.
- Guardar la serie diaria completa en SQLite.
- Transformar la serie diaria en una tabla mensual.
- Guardar el último dato disponible de cada mes.
- Mantener trazabilidad mediante usuario y fecha de registro.
- Evitar duplicados mediante llaves primarias en SQLite.

---

## Estructura del proyecto

```text
bccr-api-tipo-cambio/
│
├── data/
│   └── bccr_indicadores.db
│
├── notebooks/
│   └── 01_api_bccr_tipo_cambio.ipynb
│
├── src/
│
├── .gitignore
├── README.md
└── requirements.txt
```

Nota: la base de datos SQLite no se sube a GitHub porque está excluida en `.gitignore`.

---

## Requisitos

El proyecto utiliza Python y las siguientes librerías:

```text
pandas
requests
ipykernel
```

Para instalar las dependencias:

```bash
pip install -r requirements.txt
```

El archivo `requirements.txt` contiene:

```text
pandas
requests
ipykernel
```

---

## Seguridad y credenciales

El token del BCCR no debe escribirse directamente en el notebook ni subirse a GitHub.

El notebook utiliza:

```python
import getpass

token = getpass.getpass("Ingrese el token del BCCR: ")
```

De esta forma, el token se ingresa en tiempo de ejecución y no queda visible en el repositorio.

El correo utilizado para validar la suscripción es:

```python
correo = "mmorales@cajadeande.fi.cr"
```

---

## Archivo `.gitignore`

El archivo `.gitignore` evita subir archivos innecesarios o sensibles a GitHub.

Contenido recomendado:

```gitignore
# Entornos virtuales
.venv/
venv/
env/

# Python
__pycache__/
*.pyc

# Jupyter
.ipynb_checkpoints/

# Variables sensibles
.env

# Bases de datos locales
*.db
*.sqlite
*.sqlite3

# Archivos temporales
*.log
*.tmp

# Sistema operativo
.DS_Store
Thumbs.db
```

---

## Notebook principal

El notebook principal se encuentra en:

```text
notebooks/01_api_bccr_tipo_cambio.ipynb
```

El flujo del notebook es el siguiente:

1. Importar librerías.
2. Definir rutas del proyecto.
3. Solicitar credenciales del API BCCR.
4. Validar la suscripción.
5. Solicitar código del indicador económico.
6. Solicitar fecha inicial y fecha final.
7. Consultar el API del BCCR.
8. Convertir la respuesta JSON en un DataFrame diario.
9. Guardar la tabla diaria en SQLite.
10. Transformar la serie diaria a corte mensual.
11. Guardar la tabla mensual en SQLite.
12. Validar los datos guardados.

---

## API utilizada

La ruta base del API es:

```text
https://apim.bccr.fi.cr/SDDE/api/Bccr.GE.SDDE.Publico.Indicadores.API
```

Endpoint para validar suscripción:

```text
POST /Usuario/ValideSuscripcion
```

Endpoint para consultar series de indicadores económicos:

```text
GET /indicadoresEconomicos/{codigo}/series
```

Parámetros requeridos para consultar series:

```text
fechaInicio
fechaFin
idioma
```

Formato de fechas requerido:

```text
yyyy/mm/dd
```

Ejemplo:

```text
2026/01/01
```

---

## Ejemplos de indicadores

Algunos códigos utilizados para tipo de cambio son:

| Indicador | Código |
|---|---:|
| Tipo de cambio compra | 317 |
| Tipo de cambio venta | 318 |

---

## Tablas generadas

El proyecto genera dos tablas principales en SQLite:

```text
bccr_indicadores_diario
bccr_indicadores_mensual
```

---

## Tabla diaria

Nombre sugerido:

```text
bccr_indicadores_diario
```

Esta tabla conserva la serie diaria completa consultada desde el API del BCCR.

Estructura:

| Campo | Descripción |
|---|---|
| codigo_indicador | Código del indicador económico del BCCR |
| nombre_indicador | Nombre del indicador |
| fecha | Fecha diaria del dato |
| valor | Valor del indicador |
| MES_CORTE | Mes de referencia del dato |
| USUARIO_REGISTRO | Usuario que ejecutó el proceso |
| FECHA_REGISTRO | Fecha y hora de carga |

Llave primaria:

```text
codigo_indicador + fecha
```

Esto permite guardar varios indicadores sin duplicar registros diarios.

---

## Tabla mensual

Nombre sugerido:

```text
bccr_indicadores_mensual
```

Esta tabla conserva un único dato por mes, tomando el último dato disponible dentro de cada mes.

Estructura:

| Campo | Descripción |
|---|---|
| codigo_indicador | Código del indicador económico del BCCR |
| nombre_indicador | Nombre del indicador |
| fecha | Última fecha disponible del mes |
| valor | Valor del último dato disponible del mes |
| MES_CORTE | Mes de referencia |
| USUARIO_REGISTRO | Usuario que ejecutó el proceso |
| FECHA_REGISTRO | Fecha y hora de carga |

Llave primaria:

```text
codigo_indicador + MES_CORTE
```

Esto permite guardar compra, venta u otros indicadores en la misma tabla mensual sin conflictos.

---

## Control de duplicados

El proyecto utiliza `ON CONFLICT DO UPDATE` en SQLite.

Esto significa que si el proceso se ejecuta varias veces:

- No se duplican registros.
- Si el dato ya existe, se actualiza.
- Si el BCCR publica una corrección, el registro queda actualizado.
- Se actualiza la trazabilidad con `USUARIO_REGISTRO` y `FECHA_REGISTRO`.

---

## Funciones principales del notebook

### 1. Validar suscripción

```python
validar_suscripcion_bccr(correo, token)
```

Valida que el correo y el token tengan una suscripción válida ante el API del BCCR.

---

### 2. Consultar serie de indicador

```python
consultar_serie_indicador_bccr(
    codigo_indicador,
    fecha_inicio,
    fecha_fin,
    token,
    idioma="ES"
)
```

Consulta la serie económica del indicador indicado.

---

### 3. Convertir JSON a DataFrame diario

```python
convertir_json_bccr_a_dataframe(respuesta_json)
```

Convierte la respuesta JSON del BCCR en una tabla diaria estructurada.

---

### 4. Guardar tabla diaria en SQLite

```python
guardar_diario_en_sqlite(
    df_diario,
    db_path,
    nombre_tabla="bccr_indicadores_diario"
)
```

Guarda la serie diaria en SQLite.

---

### 5. Convertir diario a mensual

```python
convertir_diario_a_mensual(df_diario)
```

Agrupa la serie diaria por mes y conserva el último dato disponible de cada mes.

---

### 6. Guardar tabla mensual en SQLite

```python
guardar_mensual_en_sqlite(
    df_mensual,
    db_path,
    nombre_tabla="bccr_indicadores_mensual"
)
```

Guarda la serie mensual en SQLite.

---

### 7. Ejecutar proceso completo

```python
ejecutar_proceso_indicador_bccr(
    codigo_indicador,
    fecha_inicio,
    fecha_fin,
    token,
    db_path,
    nombre_tabla_diaria="bccr_indicadores_diario",
    nombre_tabla_mensual="bccr_indicadores_mensual",
    idioma="ES"
)
```

Ejecuta el proceso completo:

1. Consulta el indicador en el API del BCCR.
2. Convierte la respuesta a DataFrame diario.
3. Guarda la tabla diaria en SQLite.
4. Convierte la información diaria a mensual.
5. Guarda la tabla mensual en SQLite.
6. Retorna ambos DataFrames.

---

## Ejemplo de uso

Ejemplo para consultar el tipo de cambio compra:

```python
codigo_indicador = "317"
fecha_inicio = "2020/01/01"
fecha_fin = "2026/12/31"

df_diario_resultado, df_mensual_resultado = ejecutar_proceso_indicador_bccr(
    codigo_indicador=codigo_indicador,
    fecha_inicio=fecha_inicio,
    fecha_fin=fecha_fin,
    token=token,
    db_path=DB_PATH,
    nombre_tabla_diaria="bccr_indicadores_diario",
    nombre_tabla_mensual="bccr_indicadores_mensual",
    idioma="ES"
)

df_diario_resultado.tail()
df_mensual_resultado.tail()
```

---

## Consultar datos guardados

### Consultar tabla diaria

```python
with sqlite3.connect(DB_PATH) as conn:
    df_sql_diario = pd.read_sql_query(
        """
        SELECT *
        FROM bccr_indicadores_diario
        ORDER BY codigo_indicador, fecha
        """,
        conn,
        parse_dates=["fecha", "FECHA_REGISTRO"]
    )

df_sql_diario.tail()
```

---

### Consultar tabla mensual

```python
with sqlite3.connect(DB_PATH) as conn:
    df_sql_mensual = pd.read_sql_query(
        """
        SELECT *
        FROM bccr_indicadores_mensual
        ORDER BY codigo_indicador, MES_CORTE
        """,
        conn,
        parse_dates=["fecha", "MES_CORTE", "FECHA_REGISTRO"]
    )

df_sql_mensual.tail()
```

---

### Resumen por indicador

```python
with sqlite3.connect(DB_PATH) as conn:
    resumen_diario = pd.read_sql_query(
        """
        SELECT 
            codigo_indicador,
            nombre_indicador,
            MIN(fecha) AS fecha_minima,
            MAX(fecha) AS fecha_maxima,
            COUNT(*) AS cantidad_registros
        FROM bccr_indicadores_diario
        GROUP BY codigo_indicador, nombre_indicador
        ORDER BY codigo_indicador
        """,
        conn
    )

resumen_diario
```

---

## Uso con Git

### Inicializar repositorio

```bash
git init
```

### Revisar estado

```bash
git status
```

### Agregar archivos

```bash
git add .gitignore requirements.txt README.md notebooks/01_api_bccr_tipo_cambio.ipynb
```

### Crear commit

```bash
git commit -m "feat: crear notebook para consumir API BCCR y guardar indicadores en SQLite"
```

### Revisar historial

```bash
git log --oneline
```

---

## Subir a GitHub

Crear primero un repositorio en GitHub con el nombre:

```text
bccr-api-tipo-cambio
```

Luego conectar el repositorio local con GitHub:

```bash
git remote add origin https://github.com/TU_USUARIO/bccr-api-tipo-cambio.git
```

Cambiar el nombre de la rama principal a `main`:

```bash
git branch -M main
```

Subir el proyecto:

```bash
git push -u origin main
```

Para futuros cambios:

```bash
git add .
git commit -m "fix: ajustar proceso de consulta y guardado de indicadores BCCR"
git push
```

---

## Consideraciones

- No subir el token del BCCR a GitHub.
- No subir la base SQLite si contiene información operativa o trazabilidad interna.
- Validar que `.gitignore` excluya `.db`, `.sqlite`, `.env` y checkpoints de Jupyter.
- Usar siempre `getpass` para ingresar el token.
- Revisar la respuesta del API antes de transformar los datos.
- Conservar la tabla diaria como fuente original.
- Usar la tabla mensual como tabla derivada para análisis de cortes.

---

## Estado del proyecto

Proyecto en desarrollo para automatizar la consulta de indicadores económicos del BCCR y su almacenamiento en SQLite para procesos analíticos internos.
