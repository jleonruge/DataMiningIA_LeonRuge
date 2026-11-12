# DataMiningIA_[jleonruge]

Proyecto final de Minería de Datos con IA — *De la pregunta al insight*
Universidad Ean ·

---

## 1. Objetivo y pregunta central

**Objetivo.** Aplicar un ciclo completo de minería de datos sobre un histórico de transacciones de venta de
vehículos para identificar qué explica el valor del ticket y cómo se agrupan las operaciones, usando
herramientas de IA como asistente en cada etapa del proceso.

**Pregunta central.**
> ¿Qué factores explican el valor del ticket (precio de venta) en las transacciones de vehículos,
> y cómo se agrupan las transacciones en perfiles diferenciados?

**Hipótesis planteadas**

| # | Hipótesis | Resultado |
|---|---|---|
| H1 | A mayor ingreso anual del cliente, mayor precio del vehículo | ❌ Rechazada (r = 0,008) |
| H2 | El precio lo determinan los atributos del producto, no el cliente ni el canal | ⚠️ Aceptada parcialmente (52 % de la importancia del modelo) |
| H3 | El volumen de ventas es estacional | ✅ Aceptada (Q4 = 34 % de las ventas) |
| H4 | Existen segmentos de transacciones diferenciables | ✅ Aceptada (k = 3, silueta = 0,41) |

---

## 2. Fuente de los datos y proceso de limpieza

**Fuente.** `Car_data.csv` — histórico de 23.906 transacciones de venta de vehículos (ene-2022 a dic-2023),
con 16 variables: identificador de venta, fecha, datos del cliente (nombre, género, ingreso anual),
del concesionario (nombre, número, región) y del vehículo (marca, modelo, motor, transmisión, color,
carrocería, precio).
[Agrega aquí el enlace de origen del dataset, p. ej. Kaggle.]

**Problemas de calidad detectados en el perfilado**

| Hallazgo | Tratamiento |
|---|---|
| Encabezado `'Dealer_No '` con espacio final | Limpieza de nombres de columna con `str.strip()` |
| `Engine` con doble codificación UTF-8 (`DoubleÂ\xa0Overhead Camshaft`) | Normalización de texto (eliminación de `Â`, `\xa0` y espacios múltiples) |
| `Annual Income` = 13.500 en 22,1 % de los registros (valor centinela de relleno) | Se conserva el dato y se marca con la bandera `ingreso_dudoso`; se excluye del clustering |
| `Phone` y `Customer Name` (datos personales sin valor analítico) | Columnas eliminadas |
| `Date` almacenada como texto | Conversión a `datetime` y derivación de `anio`, `mes`, `trimestre` |
| 1 valor nulo en `Customer Name`; 0 filas duplicadas | Resuelto al eliminar la columna |
| 23.906 filas vs. máximo de 500 exigido | **Muestreo aleatorio estratificado** por carrocería y banda de precio (semilla = 42), verificando que la muestra conserve las proporciones del original |

**Dataset limpio entregado:** `Car_data_limpio_500.csv` (500 filas × 18 columnas, nombres en snake_case,
tipos corregidos y variables derivadas).

---

## 3. Técnicas de minería de datos aplicadas

| Etapa | Técnica | Herramienta |
|---|---|---|
| Exploración | Estadística descriptiva, correlaciones, análisis temporal | `pandas`, `seaborn`, `matplotlib` |
| Clasificación | Árbol de decisión y Random Forest sobre la banda de precio (Bajo/Medio/Alto) | `scikit-learn` |
| Clustering | K-Means sobre precio e ingreso (log), con método del codo y coeficiente de silueta | `scikit-learn` |
| Reglas de asociación | Apriori sobre atributos del vehículo (soporte ≥ 3 %, lift ≥ 1,15) | `mlxtend` |

**Rol de la IA en el proceso**

- **Definición del problema:** refinamiento de la pregunta de investigación y formulación de las cuatro hipótesis contrastables.
- **Preparación:** detección del mojibake, del valor centinela en el ingreso y del encabezado con espacio; diseño del muestreo estratificado en lugar de un corte de las primeras 500 filas.
- **Análisis:** selección de las técnicas según el tipo de variable objetivo, elección de hiperparámetros conservadores para n = 500 y advertencia sobre el sesgo de la importancia por impureza en variables continuas.
- **Visualización e interpretación:** propuesta de los gráficos por pregunta de negocio y redacción de los insights accionables.

---

## 4. Resultados obtenidos

**Clasificación (banda de precio, 3 clases)**

| Modelo | Exactitud |
|---|---|
| Línea base (azar) | 0,333 |
| Árbol de decisión (profundidad 4) | 0,376 |
| Random Forest (300 árboles, profundidad 8) | 0,384 |
| Random Forest sobre el dataset completo (23.906 filas) | 0,632 |

Importancia acumulada por variable original: marca 27,5 %, ingreso anual 15,2 %, región 14,9 %,
mes 11,9 %, carrocería 10,3 %. Agrupando: **producto 52 % · cliente 21 % · canal y tiempo 27 %**.

**Clustering (K-Means, k = 3, silueta = 0,41)**

| Segmento | Transacciones | Precio medio | Ingreso medio | Perfil típico |
|---|---|---|---|---|
| 0 — Ticket alto | 69 | USD 53.641 | USD 1.101.799 | Pontiac · Sedán · Scottsdale |
| 1 — Ticket bajo / ingreso alto | 130 | USD 21.088 | USD 1.519.105 | Dodge · Hatchback · Middletown |
| 2 — Ticket bajo / ingreso medio | 190 | USD 23.047 | USD 633.347 | Dodge · SUV · Austin |

**Reglas de asociación.** La regla de mayor lift asocia la marca Mitsubishi con la carrocería Hatchback
(soporte 3,2 %, confianza 51,6 %, lift 2,02), es decir, esa combinación aparece el doble de lo esperado
si ambos atributos fueran independientes.

---

## 5. Conclusiones

1. **El ingreso declarado del cliente no explica el precio del vehículo** (r ≈ 0). Además, uno de cada cinco
   registros comparte un mismo valor de relleno, así que la variable no es apta para segmentación comercial
   sin antes corregir su captura en el punto de venta.
2. **El valor del ticket se explica sobre todo por el producto**: la marca es la variable individual más
   informativa y los atributos del vehículo concentran ~52 % de la importancia del modelo, frente a 21 % del
   cliente. La palanca de valor está en el mix de inventario, no en la segmentación demográfica.
3. **La demanda es estacional**: septiembre, noviembre y diciembre concentran los picos y el cuarto trimestre
   reúne el 34 % de las ventas de la muestra → conviene anticipar inventario y fuerza comercial desde agosto.
4. **La banda de precio es predecible por encima del azar, pero con margen estrecho** (0,384 vs. 0,333) con
   500 filas. El mismo modelo alcanza 0,632 con el dataset completo: el tamaño de muestra, y no la elección
   del algoritmo, es la principal restricción del ejercicio.
5. **Los segmentos hallados se ordenan por precio, no por ingreso**, lo que refuerza la conclusión 1: el
   comportamiento de compra observado es independiente de la capacidad económica declarada.

---

## 6. Limitaciones

- Muestra de 500 filas (2,1 % del dataset original) por requisito del proyecto → alta varianza en las métricas
  e intervalos de confianza amplios.
- `ingreso_anual` contiene un valor centinela en 22,1 % de los registros; se aisló con una bandera, pero el
  sesgo residual no desaparece.
- La variable `modelo` (154 categorías) se excluyó de la clasificación para evitar sobreajuste con n = 500,
  aun siendo la más asociada al precio.
- El dataset no incluye costos, descuentos ni antigüedad del vehículo: se analiza precio de lista, no rentabilidad.
- No se aplicó validación cruzada ni búsqueda de hiperparámetros: con 375 filas de entrenamiento el resultado
  sería inestable.
- Varias variables parecen generadas de forma sintética (independencia casi total entre cliente y precio), lo
  que limita la validez externa de las conclusiones.

## 7. Recomendaciones futuras

- Repetir el ejercicio con el dataset completo, validación cruzada estratificada y `GridSearchCV` para estimar
  el techo real de predicción.
- Incorporar costo de adquisición y descuentos para pasar de *precio* a *margen*.
- Corregir la captura del ingreso del cliente y volver a probar H1 con datos confiables.
- Probar `permutation_importance` en lugar de la importancia por impureza, para evitar el sesgo hacia variables
  continuas.
- Modelar la estacionalidad con una serie temporal (por ejemplo SARIMA o Prophet) sobre el histórico completo.

---

## 8. Estructura del repositorio

```
DataMiningIA_[TuNombre]/
├── README.md                        # Este documento
├── DataMiningIA_Proyecto.ipynb      # Notebook con todo el flujo (ejecutable en Google Colab)
├── datos/
│   ├── Car_data.csv                 # Dataset original
│   └── Car_data_limpio_500.csv      # Dataset limpio entregable (500 filas)
├── salidas/                         # Gráficos generados por el notebook
│   ├── 01_eda_precio.png
│   ├── 02_eda_tiempo_corr.png
│   ├── 03_clasificacion.png
│   ├── 04_k_optimo.png
│   └── 05_clusters.png
└── video/
    └── enlace_video.md              # Enlace al video explicativo
```

## 9. Cómo reproducirlo

1. Abre `DataMiningIA_Proyecto.ipynb` en [Google Colab](https://colab.research.google.com).
2. Ejecuta la primera celda y sube `Car_data.csv` cuando aparezca el botón.
3. Corre todas las celdas (`Entorno → Ejecutar todas`). La semilla está fijada en 42, así que los resultados
   son reproducibles.
4. La última celda descarga un `.zip` con el dataset limpio y los gráficos.

## 10. Video explicativo

📹 [Enlace al video (2–3 minutos)]

Contenido: flujo de trabajo, cómo asistió la IA en cada etapa y hallazgos principales.

---

**Autor:** [Jose Leonardo Cuello] · [rugelis3@hotmail.com] · [https://github.com/jleonruge/DataMiningIA_LeonRuge]
