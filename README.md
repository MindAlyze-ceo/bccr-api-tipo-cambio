# BCCR API Tipo de Cambio

Proyecto en Python para consultar indicadores económicos del Banco Central de Costa Rica mediante el API del Sistema de Divulgación de Datos Económicos (SDDE).

El objetivo principal es sustituir el consumo anterior mediante Web Service XML por una consulta REST API en formato JSON, permitiendo guardar la información consultada en una base SQLite.

## Objetivo

Consultar indicadores económicos del BCCR por medio de su API oficial, transformar las series diarias y guardar los resultados en dos tablas:

- `bccr_indicadores_diario`: conserva la serie diaria completa.
- `bccr_indicadores_mensual`: conserva el último dato disponible de cada mes.

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
