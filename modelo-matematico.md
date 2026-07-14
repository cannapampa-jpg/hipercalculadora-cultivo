# Modelo matemático — Hipercalculadora de cultivo

Solo fórmulas, en orden de ejecución. Sin interfaz, sin componentes, sin nombres de UI. Este documento es la referencia para programar la lógica y para armar casos de prueba antes de tocar React.

---

## 1. Necesidad diaria (mg de cannabinoide objetivo)

**Modo A — dato directo:**
```
mg_día = valor cargado por el usuario
```

**Modo B — desde gotas y concentración:**
```
ml_día = gotas_día ÷ gotas_por_ml        (gotas_por_ml default 20, editable)
mg_día = ml_día × concentración_mg_ml
```

## 2. Necesidad anual de cannabinoides

```
mg_año = mg_día × días_tratamiento        (días_tratamiento default 365)
```

## 3. Eficiencia total del proceso (flor → producto)

```
eficiencia_total = eficiencia_método × factor_descarboxilación
```
- `eficiencia_método`: 0,70 (aceite artesanal) / 0,80 (alcohol) / 0,70 (otro / no lo sé)
- `factor_descarboxilación`: 0,98 (fijo, prácticamente despreciable)

## 4. Flor seca necesaria

```
gramos_secos_necesarios_año = mg_año ÷ (potencia_flor_% × 10 × eficiencia_total)
gramos_necesarios_por_ciclo = gramos_secos_necesarios_año ÷ ciclos_por_año
```
- `potencia_flor_%`: selector (5/8/10/12/15/18/20/personalizado)

---

## 5. Capacidad de producción

### 5a. Indoor y Outdoor-Suelo (cálculo directo)

```
rendimiento_base = curva_A_B(litros_maceta)                    [Indoor]
                  o curva_C(altura_final)                        [Outdoor-Suelo]

rendimiento_esperado_planta = rendimiento_base × índice_experiencia
producción_potencial = rendimiento_esperado_planta × cantidad_plantas
producción_disponible = producción_potencial × (1 − margen_seguridad)
```
- `índice_experiencia`: 0,6 (novicio) / 0,8 (medio) / 1,0 (experto)
- `margen_seguridad`: 0,30 (conservador) / 0,15 (normal) / 0 (sin margen)
- `cantidad_plantas` (Indoor): ver 5c — se deriva del espacio, no la tipea el usuario

### 5b. Outdoor-Maceta (cálculo inverso)

```
gramos_necesarios_por_planta = gramos_necesarios_por_ciclo ÷ cantidad_máxima_plantas
maceta_recomendada = mín. litraje en curva_A_B tal que
                      curva_A_B(litraje) × índice_experiencia ≥ gramos_necesarios_por_planta
semanas_vegetativo = curva_D(maceta_recomendada)
```
(fecha de siembra: ver sección 7)

### 5c. Superficie y cantidad de plantas — Indoor

```
superficie_necesaria_m2 = (litros_maceta × cantidad_plantas) ÷ 100
cantidad_máxima_plantas = (superficie_disponible_m2 × 100) ÷ litros_maceta
```
Regla de densidad de sustrato: máximo 100 L de sustrato por m².

---

## 6. Fecha de disparo de floración natural (fórmula astronómica)

```
declinación_solar(N) = 23,45 × sin(360/365 × (284 + N))          [N = día del año, 1-365]
horas_luz(latitud, N) = (24/π) × arccos(−tan(latitud) × tan(declinación_solar(N)))
```
- `latitud`: valor fijo según provincia (tabla de 24 valores, capital provincial)
- Buscar el primer `N` posterior al solsticio de verano (≈21 dic, N≈355) donde `horas_luz(latitud, N) ≤ 13,1` → esa fecha es el **disparo de floración**.

## 7. Fecha de siembra (Outdoor)

```
ventana_real = fecha_disparo_floración − fecha_fin_riesgo_helada(macrorregión)

si (semanas_vegetativo × 7) > ventana_real:
    → resultado: no alcanza el tiempo disponible
    → sugerencia: arrancar en interior y trasplantar después de la helada,
      o aceptar maceta más chica dentro del tiempo real disponible

si no:
    fecha_siembra = fecha_disparo_floración − semanas_vegetativo
```
- `fecha_fin_riesgo_helada`: tabla fija de 4 valores (NOA/NEA y Centro = principios de octubre, Cuyo = mediados-fines de octubre, Patagonia = noviembre-diciembre)

---

## 8. Comparación y semáforo

```
necesidad_anual = gramos_secos_necesarios_año
capacidad_anual = producción_disponible × ciclos_por_año         [Indoor / Outdoor-Suelo]
                   o producción_disponible (ya anualizada)        [Outdoor-Maceta, si aplica]

ratio = capacidad_anual ÷ necesidad_anual

si ratio ≥ 1,20  → 🟢 Sí alcanza
si 0,90 ≤ ratio < 1,20 → 🟡 Justo
si ratio < 0,90 → 🔴 No alcanza (sugerir ajuste)
```

## 9. Indicador de confianza y rango de salida

```
confianza = "alta" si todos los parámetros usados son valores estándar
            (potencia no personalizada, rendimiento dentro de rango habitual, combinación común)
confianza = "aproximada" en cualquier otro caso

ancho_rango = 0,10 si confianza = "alta"
ancho_rango = 0,25 si confianza = "aproximada"

rango_producción = [capacidad_anual × (1 − ancho_rango), capacidad_anual × (1 + ancho_rango)]
```

Salida final: se muestra `rango_producción` (no un número único), junto con el semáforo y el indicador de confianza.

---

## 10. Superficie total estimada (para informe)

```
superficie_m2 = (litros_maceta × cantidad_plantas) ÷ 100         [Indoor y Outdoor-Maceta]
              = valor directo cargado por el usuario               [Outdoor-Suelo, si aplica]
```

---

## Casos de prueba sugeridos (para validar antes de programar la interfaz)

1. Indoor, mg/día directo, potencia estándar, maceta 20L, 4 plantas, experiencia media, margen normal, 2 ciclos/año.
2. Outdoor-Suelo, dosis por gotas, altura media, provincia Buenos Aires, experiencia experta.
3. Outdoor-Maceta, cantidad máxima de plantas = 3, provincia Salta, margen conservador — validar que la fecha de siembra caiga dentro de la ventana real (o dispare el aviso de "no alcanza").
4. Caso límite: outdoor-maceta con litraje recomendado que exige más semanas de vegetativo que la ventana real disponible → debe activar el aviso, no una fecha imposible.
5. Potencia "personalizada" → debe bajar automáticamente a confianza "aproximada" y ensanchar el rango de salida.

---

## Árboles de decisión de la app (referencia completa, por área)

### 1. Selector de modalidad (raíz)
```
¿Cómo vas a cultivar?
├── INDOOR
└── OUTDOOR
     ├── SUELO
     └── MACETA
```

### 2. Árbol INDOOR — cálculo directo, sin ramas de recomendación
```
cantidad_por_espacio = (superficie_disponible × 100) ÷ litros_maceta
cantidad_final = mín(cantidad_por_espacio, cantidad_máxima_declarada_por_usuario)
producción = rendimiento(maceta) × índice_experiencia × cantidad_final × (1 − margen) × ciclos
```
Sin ramas condicionales de ajuste. El semáforo informa, no recomienda.

### 3. Árbol OUTDOOR-SUELO — cálculo directo, sin ramas de recomendación
```
rendimiento = curva_altura(baja/media/alta) × índice_experiencia
producción = rendimiento × cantidad_plantas × (1 − margen) × 1 ciclo
```
Mismo caso: directo, sin recomendación si el semáforo da rojo.

### 4. Árbol OUTDOOR-MACETA — único con árbol de decisión completo por restricciones
```
RAÍZ: objetivo_por_planta = (necesidad ÷ (1−margen)) ÷ cantidad_máxima_plantas
      maceta_ideal = mín. litraje que cumple ese objetivo

RAMA 0 — ¿Ni la maceta más grande razonable (100 L) alcanza el objetivo por planta?
  SÍ → restricción: RENDIMIENTO/PLANTAS → muestra plantas necesarias con esa maceta
  NO → sigue a RAMA 1

RAMA 1 — ¿La maceta ideal entra en el tiempo disponible (ventana real)?
  SÍ → RAMA A: recomienda maceta ideal + fecha de siembra
  NO → sigue a RAMA 2

RAMA 2 — busca maceta_B = la más grande que sí entra en el tiempo
  ¿Con maceta_B y cantidad máxima de plantas alcanza igual?
    SÍ → RAMA B: recomienda maceta_B (limitada por tiempo, no por elección)
    NO → RAMA C: muestra 2 caminos a la vez —
           (1) subir a X plantas con maceta_B
           (2) arrancar en interior + trasplantar después de la helada (usa maceta_ideal)
```

### 5. Árbol de confianza — aplica después de cualquiera de los anteriores
```
¿potencia cargada como "personalizado"? → SÍ → confianza = aproximada
¿maceta recomendada fuera de [5,60] L (outdoor-maceta)? → SÍ → confianza = aproximada
si ninguna → confianza = alta
ancho_rango = alta → ±10% / aproximada → ±25%
```

### 6. Árbol del semáforo final — último paso, siempre corre
```
ratio = capacidad_anual ÷ necesidad_anual
ratio ≥ 1,20        → 🟢 Sí alcanza
0,90 ≤ ratio < 1,20  → 🟡 Justo
ratio < 0,90         → 🔴 No alcanza
```

**Asimetría resuelta — motor de recomendaciones:** en vez de escribir árboles separados por modalidad, cada una declara qué variables puede modificar y en qué orden; un motor único prueba una por una (recalculando todo el resultado en cada intento, no incrementando a ciegas) hasta encontrar la primera que saca el resultado del rojo:

- **Indoor** — variables en orden: plantas (hasta el límite físico del espacio) → tamaño de maceta (recalculando cuántas entran) → ciclos por año.
- **Outdoor-Suelo** — variables en orden: plantas → altura final (desarrollo esperado).
- **Outdoor-Maceta** — mantiene su árbol específico (Rama 0-C), no usa el motor genérico porque la restricción de tiempo (fotoperíodo/heladas) no es una "variable a subir de a un escalón" sino un límite duro que cambia toda la lógica.

El motor devuelve la primera modificación mínima de una sola variable que resuelve el problema; si ninguna alcanza, informa que hace falta combinar ajustes o revisar dosis/potencia con el RT.
