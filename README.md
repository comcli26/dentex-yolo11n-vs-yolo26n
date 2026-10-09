# dentex-yolo11n-vs-yolo26n

Comparación controlada de **YOLO11n** y **YOLO26n** para detectar **lesiones periapicales** y
**dientes impactados** en radiografías panorámicas (conjunto DENTEX).
Artículo asociado: <título + enlace/DOI cuando exista>.

> ⚠️ Prototipo de investigación. Mide concordancia algorítmica con las anotaciones del conjunto;
> **no es un dispositivo ni una herramienta de diagnóstico clínico.**

[![DOI](https://zenodo.org/badge/DOI/<DOI_ZENODO>.svg)](https://doi.org/<DOI_ZENODO>)

## Resultados principales (partición de prueba, semilla 42)
<pegar Tabla 5 resumida: mAP@50, mAP@50–95, F1, parámetros, GFLOPs, ms/imagen>

## Contenido
<árbol de carpetas resumido>

## Cómo reproducir
1. Descargar `training_data.zip` de DENTEX: https://doi.org/10.5281/zenodo.7812323 (md5 en `data/README_data.md`).
2. Ejecutar `notebooks/01_...` (usa solo la carpeta `quadrant-enumeration-disease`).
3. Ejecutar `02a_...` y `02b_...` (pueden correr en paralelo; GPU Tesla T4).
4. Ejecutar `03_evaluacion_conjunta`.
5. Probar el prototipo con `04_prototipo_inferencia`.
Los cuadernos asumen rutas de Google Drive; ajustar la variable `ROOT` de la primera celda.

## Verificación de que ambos modelos usan la misma partición
`data/metadata/split_evidence.json` y `results/evidence/evidencia_comparacion.json` (SHA-256).

## Datos y licencias
- Imágenes y anotaciones originales: DENTEX (Hamamci et al., 2023; Er, 2023), CC BY 4.0. **No se redistribuyen las radiografías.**
- Etiquetas YOLO derivadas y listas de partición: CC BY 4.0 (atribución a DENTEX).
- Código y pesos: AGPL-3.0 (coherente con Ultralytics).

## Cómo citar
Ver `CITATION.cff`.
