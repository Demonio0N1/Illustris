# Análisis Cinemático de Galaxias en IllustrisTNG (TNG100-1)

Pipeline para la selección, análisis cinemático y clasificación segun curvas de Luz de galaxias de la simulación cosmológica **IllustrisTNG** (TNG100-1, snapshot 99, 85, 67, 50 y 33).

## Descripción general

Este proyecto implementa un flujo de trabajo de cuatro etapas para estudiar la relación entre la historia de fusiones (*mergers*) y las distorsiones cinemáticas en galaxias masivas con gas. Se aplica la clasificación de van Eymeren et al. (2011) a las curvas de rotación ajustadas con MCMC.

## Notebooks

### `1)Filtro.ipynb` — Selección de galaxias
Filtra la muestra de subhalos de TNG100-1 aplicando criterios físicos y cinemáticos:
- **Aislamiento**: ningún vecino con M★ ≥ 10 % dentro de 100 kpc
- **Masa estelar**: 10.5 < log(M★/M☉) < 11.5 (dentro de 2 R½)
- **Fracción de gas**: M_gas ≥ 20 % de M★
- **Inclinación**: 10° < i < 80°
- **Asimetría cinemática**: A < 0.6 y PCR > 0.4 para estrellas y gas

Produce un `DataFrame` con parámetros de posición, velocidad, ángulo de posición e inclinación de cada galaxia seleccionada.

### `2)Mapas de velocidad.ipynb` — Mapas 2D de velocidad del gas
Para cada subhalo de la muestra:
- Construye mapas 2D de velocidad en la línea de visión (LOS = Z) del gas usando estadística binned
- Aplica un filtro mínimo de 9 partículas por píxel
- Determina el ángulo de posición cinemático (φ₀) maximizando el gradiente de velocidad
- Exporta los píxeles válidos a archivos `.csv` con columnas `r_kpc`, `phi_rad`, `Vobs_kms`

### `3)mcmc.ipynb` — Ajuste MCMC y clasificación cinemática
- Deproyecta las velocidades observadas usando la geometría (i, φ₀) de cada galaxia
- Ajusta independientemente el lado positivo (recesión) y negativo (aproximación) con el modelo:

  **V(r) = Vc · tanh(r / Rt) + s_out · r**

  usando `emcee` (32 walkers, 1500 pasos, 500 de burn-in).
- Clasifica las galaxias en 5 tipos según van Eymeren et al. (2011):
  - **Tipo 1**: curvas simétricas
  - **Tipo 2**: desplazamiento global constante
  - **Tipo 3**: distorsión local externa
  - **Tipo 4**: distorsión local interna
  - **Tipo 5**: cruce entre curvas (inversión cinemática)

### `4)Mergers.ipynb` — Historia de fusiones y propiedades de masa
- Lee las clasificaciones cinemáticas y traza la historia de fusiones vía SubLink (Snap 71 → 99, últimos ~5 Gyr)
- Distingue **major mergers** (razón de masa ≥ 1:4) y **minor mergers** (razón < 1:10)
- Analiza la distribución de masas (estelar, gas, materia oscura, agujero negro) por tipo cinemático
- Genera histogramas y scatter plots comparativos entre tipos

## Dependencias

```
numpy
pandas
scipy
matplotlib
emcee
illustris_python
```

La simulación TNG100-1 debe estar accesible en el `basePath` configurado en cada notebook (`/home/tnguser/sims.TNG/TNG100-1/output/`).

## Flujo de trabajo

```
Filtro.ipynb
    └── subhalo_ids seleccionados
         └── Mapas de velocidad.ipynb
                  └── archivos *.csv por galaxia
                           └── mcmc.ipynb
                                    └── cls99.txt (clasificaciones)
                                             └── Mergers.ipynb
```

## Referencia

- van Eymeren et al. (2011) — clasificación de asimetrías en curvas de rotación
- [IllustrisTNG](https://www.tng-project.org/) — simulación cosmológica TNG100-1
