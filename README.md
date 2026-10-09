# dentex-yolo11n-vs-yolo26n

Comparación controlada de **YOLO11n** y **YOLO26n** para detectar **lesiones periapicales** y **dientes impactados** en radiografías panorámicas (conjunto público DENTEX). Incluye cuadernos de preparación, entrenamiento y evaluación, particiones, resultados, pesos finales y un **prototipo de inferencia en Google Colab**.

Artículo asociado: *Sistema basado en deep learning para la detección de patologías dentales en radiografías panorámicas: evaluación comparativa de YOLO11n y YOLO26n* (en revisión).

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23251984.svg)](https://doi.org/10.5281/zenodo.23251984)
[![Abrir prototipo en Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/comcli26/dentex-yolo11n-vs-yolo26n/blob/main/notebooks/04_prototipo_inferencia.ipynb)

> ⚠️ **Prototipo de investigación.** Mide la concordancia algorítmica con las anotaciones del conjunto DENTEX. **No es un dispositivo médico ni una herramienta de diagnóstico clínico.**

## Diseño del estudio

- Conjunto: DENTEX, carpeta `quadrant-enumeration-disease` (705 radiografías panorámicas).
- Clases: lesión periapical (158 instancias) y diente impactado (604 instancias). Las imágenes sin ninguna de las dos clases (379) se usan como negativas.
- Partición única: semilla 42, 70/15/15, estratificada (493 / 106 / 106 imágenes). La misma para ambos modelos.
- Protocolo idéntico: 1024 × 1024 px, AdamW, hasta 150 épocas con paciencia 30, lote 16, aumentos fijados explícitamente, pesos COCO.
- Umbral de confianza elegido en **validación** (0,46 para YOLO11n y 0,17 para YOLO26n) y fijado en prueba.
- Intervalos de confianza del 95 % por bootstrap de imágenes (1 000 remuestreos) y diferencia pareada entre modelos.
- Entorno: Ultralytics 8.4.118, Python 3.13, PyTorch 2.11, GPU Tesla T4 (Google Colab).

## Resultados principales (partición de prueba, semilla 42)

| Métrica | YOLO11n | YOLO26n |
|---|---|---|
| mAP@50 [IC 95 %] | 0,581 [0,494–0,689] | 0,565 [0,484–0,659] |
| mAP@50–95 [IC 95 %] | 0,367 [0,307–0,443] | 0,374 [0,313–0,446] |
| Precisión | 0,841 | 0,760 |
| Recall | 0,617 | 0,633 |
| F1 | 0,712 | 0,691 |
| Parámetros (fusionado) | 2 582 542 | 2 375 226 |
| GFLOPs a 1024 px (fusionado) | 16,56 | 14,09 |
| Tamaño de `best.pt` | 5,51 MB | 5,42 MB |
| Tiempo total por imagen (T4, lote 1) | 14,36 ms | 14,35 ms |

Ninguna diferencia global en mAP, recall ni F1 es concluyente (el IC de la diferencia incluye el cero). La única diferencia cuyo IC excluye el cero es la precisión, favorable a YOLO11n, y depende del umbral elegido. Ambos modelos detectan mejor los dientes impactados que las lesiones periapicales. Todas las tablas están en `results/tables/`.

## Estructura del repositorio

```text
├── README.md · LICENSE · CITATION.cff · .zenodo.json
├── requirements.txt · environment_colab.txt
├── notebooks/
│   ├── 01_preparacion_dataset_DENTEX.ipynb   # conversión COCO→YOLO, partición, verificaciones
│   ├── 02a_entrenamiento_YOLO11n.ipynb
│   ├── 02b_entrenamiento_YOLO26n.ipynb
│   ├── 03_evaluacion_conjunta.ipynb          # métricas con IC, matriz de confusión, tiempos, complejidad
│   └── 04_prototipo_inferencia.ipynb         # prototipo del sistema
├── data/                                     # data.yaml, listas de partición, etiquetas derivadas, metadatos (ver data/README_data.md)
├── models/                                   # pesos finales (.pt)
└── results/
    ├── training/                             # curvas, results.csv, evaluaciones de cada entrenamiento
    ├── tables/                               # tablas CSV
    ├── figures/                              # figuras del artículo
    └── evidence/                             # evidencia_comparacion.json (hashes del protocolo, del conjunto y de los pesos)
```

## Cómo reproducir

Los cuadernos asumen rutas de Google Drive: ajusta la variable `ROOT` en la primera celda de cada uno.

1. Descargar `training_data.zip` de DENTEX (https://doi.org/10.5281/zenodo.7812323) y subirlo a Drive. Detalles en `data/README_data.md`.
2. Ejecutar `notebooks/01_preparacion_dataset_DENTEX.ipynb`.
3. Ejecutar `02a_...` y `02b_...` (pueden correr en paralelo en dos sesiones; ≈ 33 y ≈ 23 min en una T4).
4. Ejecutar `03_evaluacion_conjunta.ipynb`.

## Probar el prototipo (sin entrenar nada)

1. Abrir `notebooks/04_prototipo_inferencia.ipynb` con el botón de Colab de arriba.
2. Activar GPU (Entorno de ejecución → Cambiar tipo de entorno de ejecución → T4). También funciona en CPU, con más tiempo de respuesta.
3. Ejecutar todas las celdas. El cuaderno descarga los pesos de `models/`, te pide cargar una radiografía panorámica (PNG o JPG), ejecuta el modelo elegido y muestra la radiografía con las cajas, la clase y la confianza de cada detección.

Ejemplo de salida:

```text
Patología detectada: diente impactado
Confianza: 75.7 %
Modelo: YOLO26n
```

Colores: rojo = lesión periapical; verde = diente impactado; amarillo discontinuo = anotación de referencia (solo para imágenes de prueba de DENTEX).

## Verificación de que ambos modelos usan la misma partición y los mismos pesos

- `data/metadata/split_evidence.json` y `results/evidence/evidencia_comparacion.json`: SHA-256 de `data.yaml`, de las listas de partición, del protocolo y de los pesos.
- Los pesos se pueden comprobar con:

```bash
cd models
sha256sum -c SHA256SUMS.txt
```

## Datos y licencias

- Imágenes y anotaciones originales: DENTEX (Hamamci et al., 2023; Er, 2023), **CC BY 4.0**. Las radiografías **no se redistribuyen**.
- Etiquetas YOLO derivadas y listas de partición: CC BY 4.0, con atribución a DENTEX.
- Código y pesos: **AGPL-3.0**, coherente con la licencia de Ultralytics.

## Limitaciones

Una sola semilla por modelo, partición de prueba pequeña (106 imágenes; 27 lesiones periapicales), sin identificadores de paciente en DENTEX (no se puede descartar fuga entre particiones) y sin validación clínica externa.

## Cómo citar

Ver `CITATION.cff` o:

> Muñoz Vélez, G. E., Jimbo Ortiz, J. G., Rivas Asanza, W. B., Tusa Jumbo, E. A., & Celleri Pacheco, J. K. (2026). *dentex-yolo11n-vs-yolo26n* (Version 1.0.1) [Computer software]. Zenodo. https://doi.org/10.5281/zenodo.XXXXXXX
