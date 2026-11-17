# Reto de la concentración "Inteligencia artificial avanzada para la ciencia de datos": datos

Este repositorio reúne la entrega de datos del reto con el CAETEC, que es estimar la condición corporal (BCS) de las vacas a partir de fotos. Aquí están la justificación del uso de herramientas de Big Data, el modelo de almacenamiento en AWS, los notebooks que extraen, limpian y cargan los datos, la validación cruzada k-fold y la medición de tiempos y recursos.

El proceso completo de CRISP-DM y sus reportes por fase están en el repositorio del proyecto, [ai-model-tutsipink](https://github.com/AIPorDefinir/ai-model-tutsipink).

Reporte: [Justificación del uso de herramientas de Big Data](justificacion-bigdata.pdf)

## Entregable

| # | Instrucción | Dónde está |
|---|---|---|
| 1 | Herramientas y tecnologías para trabajar con los datos, y por qué | [Reporte](justificacion-bigdata.pdf) |
| 2 | Modelo de almacenamiento de los datos del reto | [Reporte](justificacion-bigdata.pdf) y la configuración exportada de AWS en [config/aws/](config/aws/) |
| 3 | Extraer, limpiar y cargar el conjunto de datos | [comprension_datos.ipynb](notebooks/comprension_datos.ipynb), [preparacion_datos.ipynb](notebooks/preparacion_datos.ipynb) (incluye la carga a S3) y [carga_resumen.json](resultados/carga_resumen.json) |
| 4 | Scripts de configuración y evidencia de que la carga funciona | [config/](config/), [scripts/pyspark/](scripts/pyspark/), [herramientas/](herramientas/) y la sección de carga del notebook de preparación. No hay muestra de datos, ver [Política de datos](#política-de-datos) |
| 5 | Separación en entrenamiento y prueba con k-fold | Asignación de pliegues en [preparacion_datos.ipynb](notebooks/preparacion_datos.ipynb), ejecución en [validacion_cruzada.ipynb](notebooks/validacion_cruzada.ipynb) y sus resultados en [resultados/](resultados/) |
| 6 | ¿Hace falta un enfoque de Big Data? | [Reporte](justificacion-bigdata.pdf), [medicion_recursos.ipynb](notebooks/medicion_recursos.ipynb) y [tiempos_recursos.csv](resultados/tiempos_recursos.csv) |

## Política de datos

Los datos del CAETEC (fotos, bitácora de los expertos, hoja manuscrita y tablas del dataset) solo se guardan en el bucket privado de S3 del equipo, como lo define la política de datos del equipo. Por eso este repositorio no incluye los archivos de datos ni una muestra de ellos, y el nombre del bucket y las llaves de AWS no aparecen en ningún archivo.

La evidencia de que la carga funciona es la sección de carga del notebook de preparación. Esa sección compara cada archivo que genera el notebook con el que está en el bucket (por tamaño y por su huella MD5) y guarda un resumen con conteos en [resultados/carga_resumen.json](resultados/carga_resumen.json). La primera carga del dataset se hizo a mano desde la consola de S3, y la sección reproduce esa carga y la verifica.

## Flujo de los datos

1. Las fotos de las dos visitas y las hojas de BCS llegaron por carpetas de Google Drive y se subieron al bucket de S3 (región `mx-central-1`).
2. `comprension_datos.ipynb` baja los datos crudos del bucket a Colab y los describe, sin modificarlos.
3. La vista de cada foto se marcó a mano con [marcar_vistas.html](herramientas/marcar_vistas.html), y el `vistas.csv` resultante se subió al bucket.
4. `preparacion_datos.ipynb` baja los datos crudos y `vistas.csv`, quita copias y exclusiones, convierte las fotos a JPG, construye la etiqueta de BCS y los pliegues, arma `dataset_bcs/` y lo compara con el del bucket.
5. `validacion_cruzada.ipynb` lee las tablas del dataset y documenta la ejecución de la validación cruzada con un modelo base.
6. `medicion_recursos.ipynb` mide tiempo, CPU y memoria de cada etapa, y compara la integración con pandas y con PySpark.

## Cómo se corre

Los notebooks corren en Google Colab. Para abrir uno desde GitHub usa Archivo, Abrir cuaderno, GitHub, y pega la liga de este repositorio. En el ícono de la llave de la barra izquierda agrega tres secretos y dales acceso al notebook.

| Secreto | Contenido |
|---|---|
| `AWS_KEY_ID` | ID de la llave de acceso de tu usuario de IAM |
| `AWS_SECRET` | Llave secreta de tu usuario de IAM |
| `S3_BUCKET` | Nombre del bucket del equipo |

El orden es comprensión, preparación, validación cruzada y medición. Los últimos tres descargan al terminar los archivos que van en [resultados/](resultados/).

Los scripts de PySpark también corren fuera de Colab, con Python 3, Java 8, 11 o 17 y los paquetes de [config/requirements.txt](config/requirements.txt):

```bash
cd scripts/pyspark
python3 preparar_entradas.py --dataset <carpeta con los CSV del dataset> --vistas <vistas.csv> --salida entrada
python3 comparar_tiempos.py --entrada entrada --salida salida --referencia <carpeta con los CSV del dataset>
python3 comparar_tiempos.py --entrada entrada --salida salida_cluster --master spark://<ip-del-master>:7077
```

Para exportar la configuración de AWS (solo lectura y sin datos de la cuenta), quien tenga permisos sobre la cuenta corre [exportar_config_aws.py](config/aws/exportar_config_aws.py):

```bash
python3 config/aws/exportar_config_aws.py --bucket <nombre-del-bucket> --perfil <perfil> --salida config/aws/exportado
```

## Estructura

```
justificacion-bigdata.pdf   reporte de la entrega
src/                        fuentes en LaTeX del reporte
notebooks/                  comprensión, preparación, validación cruzada y medición (Colab)
scripts/pyspark/            integración con pandas y con PySpark, y medición de tiempo, CPU y memoria
herramientas/               página para marcar la vista de cada foto
config/                     paquetes y configuración exportada de AWS
resultados/                 archivos que generan los notebooks al correr
```

## Contributors

- A01066843 Facundo Bautista Barbera ([@Facundo-Barbera](https://github.com/Facundo-Barbera))
- A01199004 Oswaldo Isaías Hernández Santes ([@taqueritospro](https://github.com/taqueritospro))
- A01711368 Alfredo Alejandro Soto Herrera ([@AlejandroSH1](https://github.com/AlejandroSH1))
- A01712408 Emiliano Camacho Ponce ([@Emiliano1410](https://github.com/Emiliano1410))
- A01352358 Iker Mejía Hernández ([@ikerMJHDZ09](https://github.com/ikerMJHDZ09))
- A01711783 Jorge Manuel Oyoqui Aguilera ([@JorgeManuelOyoqui](https://github.com/JorgeManuelOyoqui))
