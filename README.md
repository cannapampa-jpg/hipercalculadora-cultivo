# Hipercalculadora de cultivo — Cannabis medicinal

Herramienta de planificación para autocultivo medicinal. Estima si un cultivo puede cubrir la necesidad de consumo del paciente: cuántas plantas, qué maceta, y desde cuándo sembrar.

## Cómo funciona

Compara **necesidad anual** (dosis del paciente × frecuencia × eficiencia de extracción) contra **capacidad de producción** (rendimiento por planta × cantidad de plantas × ciclos). Si no alcanza, sugiere ajustes concretos (más plantas, maceta más grande, sumar ciclos, o arrancar en interior y trasplantar).

## Stack

- HTML + CSS + JS vanilla (archivo único, sin dependencias)
- Deploy: Netlify (sitio estático)
- Subdominio: `cultivo.matiasbucat.com.ar`

## Modelo de cálculo

El modelo matemático completo está documentado en `modelo-matematico.md`.

## Integración

Enlaza con el [Formulador de aceite](https://formulador.matiasbucat.com.ar) pasando la cantidad de flor disponible por parámetro de URL.

## Autor

Matías Bucat — Ing. Agrónomo (UBA) · RT REPROCANN  
[matiasbucat.com.ar](https://matiasbucat.com.ar)
