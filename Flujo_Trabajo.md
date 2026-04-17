# Flujo de Trabajo: Segmentación de Jugadores FIFA19 (Diplomado Ciencias de Datos - Gen 33)

## 1. Objetivo del Proyecto
Identificar segmentos de jugadores de campo con características técnicas y físicas similares, excluyendo métricas compuestas, para facilitar la construcción de equipos ficticios balanceados basados en habilidades puras.

## 2. Datos de Entrada
- `FIFA19-DS.csv`: 17k+ registros, 89 columnas.
- `FIFA19-MD.xlsx`: Diccionario de variables (hoja "Descripción").

## 3. Análisis Exploratorio y Selección de Variables

### 3.1. Hallazgos clave del EDA (basado en muestra y diccionario)
- **Height**: string "5.7" (pies.pulgadas) → requiere conversión a cm.
- **Weight**: número entero (libras) → requiere conversión a kg.
- **Work Rate**: numérico (4-8), no categórico como indica el diccionario.
- **Variables GK**: presentes en jugadores de campo con valores residuales bajos.
- **Overall, Potential, Special**: métricas compuestas que no deben usarse en clustering.

### 3.2. Variables seleccionadas para clustering (jugadores de campo)

| Categoría | Variables incluidas | Justificación |
|-----------|---------------------|---------------|
| Demográfico | Age | Edad influye en atributos físicos y desarrollo. |
| Habilidad especial | Weak Foot, Skill Moves, Work Rate | Versatilidad y compromiso táctico. |
| Físico | Height_cm, Weight_kg | Dimensiones corporales convertidas. |
| Habilidades técnicas | Crossing, Finishing, HeadingAccuracy, ShortPassing, Volleys, Dribbling, Curve, FKAccuracy, LongPassing, BallControl, ShotPower, LongShots, Penalties, Marking, StandingTackle, SlidingTackle, Interceptions | Atributos ofensivos y defensivos básicos. |
| Atributos físico-mentales | Acceleration, SprintSpeed, Agility, Reactions, Balance, Jumping, Stamina, Strength, Aggression, Positioning, Vision, Composure | Velocidad, resistencia, inteligencia de juego. |

### 3.3. Variables excluidas del clustering

| Variable | Motivo de exclusión |
|----------|----------------------|
| Overall, Potential, Special | Métricas compuestas; sesgarían la segmentación. |
| Value, Wage, International Reputation | Reflejan mercado, no habilidades de juego. |
| Posiciones específicas (LS, ST, CAM, etc.) | Redundantes con habilidades base. |
| Position | Categórica; se usará solo en perfilamiento. |
| Preferred Foot, Body Type | Categóricas; no numéricas. |
| Variables de portero (GK*) | Se excluyen porteros del clustering de campo. |

## 4. Preprocesamiento de Datos

### 4.1. Filtrado
- Excluir registros con `Position == 'GK'` (análisis separado para porteros si se requiere).

### 4.2. Conversión de unidades
- **Height**: función que parsea "5.7" → (pies*30.48) + (pulgadas*2.54) = cm.
- **Weight**: multiplicar por 0.453592 para obtener kg.

### 4.3. Manejo de valores faltantes
- Imputar con mediana por grupo posicional (usando `Position`) para variables con <5% de nulos.

### 4.4. Estandarización
- Aplicar `StandardScaler` a todas las variables seleccionadas para que tengan media 0 y desviación 1.

## 5. Reducción de Dimensionalidad: PCA

### 5.1. Justificación de PCA sobre otras técnicas
- **t-SNE**: no preserva distancias globales; inadecuado como entrada para KMeans.
- **MDS**: costo computacional prohibitivo (matriz 17k x 17k).
- **PCA**: eficiente, determinista, elimina multicolinealidad y permite interpretar componentes como factores de rendimiento.

### 5.2. Implementación
- Retener número de componentes que expliquen ≥ 85% de la varianza acumulada.
- Guardar scores en `X_pca`.

## 6. Determinación del Número Óptimo de Clusters (k)

### 6.1. Métricas a evaluar
- **Método del codo**: inercia intra-cluster para k=2..15.
- **Coeficiente de Silueta**: promedio para cada k; buscar máximo.

### 6.2. Criterio de selección
- k que maximice silueta y/o muestre codo claro, dentro del rango 5–8 esperado.

## 7. Algoritmo de Agrupamiento: KMeans

### 7.1. Justificación de KMeans
- Tras PCA, los datos presentan clusters aproximadamente esféricos.
- Rápido, escalable y fácil interpretación de centroides.
- Reproducible con semilla fija (`random_state=333`) y `n_init=10`.

### 7.2. Evaluación de calidad del clustering final
- Coeficiente de silueta por cluster y global (>0.3 aceptable).
- Verificación de tamaño de clusters (ninguno <1% sin justificación).

## 8. Perfilamiento de Clusters

### 8.1. Método
- Calcular media de cada variable original estandarizada por cluster.
- Restar media global para obtener z-score del cluster.
- Identificar 5-7 habilidades con mayor z-score positivo (fortalezas) y negativo (debilidades).

### 8.2. Visualizaciones para informe
- Radar chart comparativo de habilidades clave.
- Scatter plot de dos primeros componentes PCA coloreado por cluster.
- Boxplot de Overall y Potential por cluster (para perfilamiento adicional).

### 8.3. Interpretación de perfiles
Cada cluster representará un "rol táctico latente" (ej. "Defensa central dominante", "Extremo explosivo", "Creador de juego").

## 9. Entregables

### 9.1. Presentación ejecutiva (PDF ≤5 páginas)
Estructura sugerida:
1. Portada y objetivo.
2. EDA y selección de variables (mapa de calor de correlaciones).
3. Reducción de dimensionalidad (scree plot) y elección de k (gráficos codo/silueta).
4. Perfilamiento de clusters (radar chart + tabla resumen).
5. Conclusiones y aplicación para equipos balanceados.

### 9.2. Código (Python)
- Reproducible (incluir `random_seed=333` y `np.random.seed(333)`).
- Modular y comentado.
- Incluir funciones de limpieza, PCA, clustering, evaluación y gráficos.

