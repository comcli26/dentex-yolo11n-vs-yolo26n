# Datos: cómo obtener y preparar DENTEX

Este repositorio **no redistribuye las radiografías**. Aquí se explica cómo obtenerlas y qué parte se usó.

## Conjunto de datos original

- **DENTEX Challenge 2023** (Hamamci et al., 2023; Er, 2023)
- Zenodo: https://doi.org/10.5281/zenodo.7812323
- Licencia: **CC BY 4.0** (se puede reutilizar con atribución)
- Archivo usado: `training_data.zip` (≈ 10,9 GB)
- MD5 del zip: `<COPIAR DESDE LA FICHA DE ZENODO>` (se comprueba con `md5sum training_data.zip` en Linux o `certutil -hashfile training_data.zip MD5` en Windows)

## Qué carpeta se usa y por qué (importante)

El zip contiene **cuatro carpetas** y **cada una numera sus archivos por separado**: `train_0.png` existe en las cuatro, pero son radiografías distintas.

| Carpeta | Imágenes | Se usa |
|---|---|---|
| `quadrant` | 693 | No |
| `quadrant_enumeration` | 634 | No |
| `quadrant-enumeration-disease` | 705 | **Sí** |
| `unlabelled` | 1571 | No |

Para que las cajas queden sobre la anatomía correcta, cada JSON debe emparejarse **solo con las imágenes de su propia carpeta**. El cuaderno `01_preparacion_dataset_DENTEX.ipynb` lo hace y aborta si más del 1 % de las imágenes tiene un tamaño distinto al declarado en el JSON.

## Qué se extrajo

- JSON: `quadrant-enumeration-disease/train_quadrant_enumeration_disease.json` (campo `categories_3`, nivel de diagnóstico)
- Categorías retenidas:

| Categoría DENTEX | Clase YOLO | Instancias | Imágenes |
|---|---|---|---|
| Periapical Lesion (id 2) | 0 · `periapical_lesion` | 158 | 116 |
| Impacted (id 0) | 1 · `impacted_tooth` | 604 | 254 |

- Categorías descartadas: Caries y Deep Caries. Las imágenes que solo las tienen se conservan como **negativas** (379 imágenes).
- Total: 705 imágenes (326 con al menos una clase objetivo, 44 con ambas, 379 negativas).

## Partición (semilla 42, 70/15/15, estratificada)

| Partición | Imágenes | Negativas | Lesión periapical | Diente impactado |
|---|---|---|---|---|
| train | 493 | 265 | 107 | 428 |
| val | 106 | 57 | 24 | 83 |
| test | 106 | 57 | 27 | 93 |

Las listas exactas están en `data/splits/` y sus SHA-256 en `data/metadata/split_evidence.json`.

## Contenido de esta carpeta

| Ruta | Descripción |
|---|---|
| `data.yaml` | Configuración del conjunto para Ultralytics (la ruta `path:` corresponde a Colab) |
| `splits/train.txt`, `val.txt`, `test.txt` | Nombres de archivo de cada partición |
| `labels_yolo/{train,val,test}/*.txt` | Etiquetas derivadas en formato YOLO (clase, centro y tamaño normalizados). Una por imagen; vacía = imagen negativa |
| `metadata/` | Estadísticas por imagen, resumen de clases, evidencia de la partición y verificación visual de las cajas |

## Cómo reconstruir el conjunto

1. Descargar `training_data.zip` de Zenodo y subirlo a Google Drive.
2. Ejecutar `notebooks/01_preparacion_dataset_DENTEX.ipynb` (ajustar `ZIP_DRIVE_PATH` en la primera celda). Genera `DENTEX-YOLO-2clases.zip` con `images/`, `labels/`, `splits/` y `data.yaml`.
3. Comprobar que los SHA-256 coinciden con `metadata/split_evidence.json`. Los cuadernos de entrenamiento lo verifican automáticamente y se detienen si no coinciden.

## Atribución

Las etiquetas YOLO y las listas de partición son obras derivadas de DENTEX y se comparten bajo **CC BY 4.0**:

> Hamamci, I. E., et al. (2023). DENTEX: Dental enumeration and tooth pathosis detection benchmark for panoramic X-ray. arXiv. https://doi.org/10.48550/arXiv.2305.19112
> Er, S. (2023). DENTEX CHALLENGE 2023 [Data set]. Zenodo. https://doi.org/10.5281/zenodo.7812323
