# Tarea 06: SageMaker Processing Job — BYOC con scikit-learn
## Diana Arroyo / Luis Cuadros

## Descripción del proyecto
En esta tarea se implementó un SageMaker Processing Job usando el patrón Bring Your Own Container (BYOC) para ejecutar una lógica de preprocesamiento de datos con `scikit-learn`, `pandas` y `numpy`.

El objetivo fue construir un flujo reproducible y desacoplado del entorno local, donde SageMaker administra la transferencia de archivos entre Amazon S3 y el contenedor, mientras que el script de procesamiento trabaja únicamente con rutas locales dentro de `/opt/ml/processing/`.

El procesamiento realizado transforma el archivo crudo `sales_train.csv` en un dataset agregado a nivel mensual, generando como salida el archivo `monthly_sales.csv`, el cual posteriormente queda almacenado en S3.

---

## Estructura del repositorio
```text
Tarea_6/
├── data/
│   └── raw/
│       └── sales_train.csv
├── processing/
│   ├── container/
│   │   └── Dockerfile
│   └── prep.py
├── sm_processing_byoc.ipynb
├── README.md
└── requirements.txt
```
---

Componentes principales
1. processing/prep.py

Este script contiene la lógica de transformación de datos.
Su función principal es:

leer el archivo crudo desde /opt/ml/processing/input/sales_train.csv

transformar las ventas a nivel mensual

generar el archivo de salida en /opt/ml/processing/output/monthly_sales.csv

El script no interactúa directamente con S3, ya que SageMaker se encarga de mover los archivos mediante ProcessingInput y ProcessingOutput.

2. processing/container/Dockerfile

Se construyó una imagen Docker mínima basada en python:3.11-slim, con las dependencias necesarias para ejecutar el script de procesamiento:

pandas

numpy

scikit-learn

El contenedor se mantuvo simple, sin configuración de clústeres ni archivos adicionales de serving/training, ya que esta tarea corresponde exclusivamente a un Processing Job.

Contenido base del contenedor:

imagen base: python:3.11-slim

instalación de dependencias con pip

ENTRYPOINT ["python3"]

3. sm_processing_byoc.ipynb

Este notebook implementa el flujo completo del Processing Job:

Configuración de la sesión de SageMaker

Obtención de región, rol y bucket por defecto

Definición de rutas locales del proyecto

Carga del archivo crudo a S3

Construcción de la imagen Docker

Publicación de la imagen en Amazon ECR

Creación y ejecución del ScriptProcessor

Escritura del output en S3

Lectura e inspección del archivo transformado

Flujo de procesamiento

El flujo implementado fue el siguiente:

S3 (datos crudos)
   ↓
/opt/ml/processing/input/
   ↓
prep.py
   ↓
/opt/ml/processing/output/
   ↓
S3 (datos procesados)

SageMaker administra automáticamente la transferencia del archivo de entrada y del archivo de salida entre S3 y el contenedor.

Ejecución del Processing Job

El Processing Job se ejecutó mediante ScriptProcessor, usando:

imagen publicada en Amazon ECR

una instancia ml.m5.large

entrada desde S3

salida hacia S3

La ejecución fue exitosa y el script generó correctamente el archivo:

monthly_sales.csv
Resultado obtenido

El archivo de salida generado fue:

s3://sagemaker-us-east-1-995371347105/tarea-6-processing-byoc/output/monthly_sales.csv

Al inspeccionar el resultado en el notebook, se validó que:

el archivo fue creado correctamente

las columnas de salida son:

date_block_num

shop_id

item_id

item_cnt_month

la dimensión del dataset resultante es:

(1609124, 4)
Evidencia de ejecución

Se generaron evidencias de los siguientes puntos:

Processing Job con status Completed en SageMaker

Imagen publicada en Amazon ECR

Archivo de salida en S3

Inspección del output en el notebook con df_out.head() y df_out.shape

Dependencias

Las principales dependencias utilizadas fueron:

Python 3.11

boto3

sagemaker

pandas

numpy

scikit-learn

Conclusión

Con esta implementación se construyó exitosamente un pipeline de preprocesamiento reproducible usando Amazon SageMaker Processing con un contenedor propio. El enfoque BYOC permitió controlar las dependencias del entorno y desacoplar la lógica de transformación del ambiente local, cumpliendo con el objetivo de generar un flujo escalable y listo para integrarse en etapas posteriores de entrenamiento.
