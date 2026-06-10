# Impacto del Preprocesamiento de Imágenes en el Desempeño de una CNN para Clasificación Botánica

Este proyecto analiza y evalúa el impacto de aplicar una técnica avanzada de preprocesamiento de imágenes —basada en la proyección HCL (Máximo Contraste), umbralización de Otsu y refinamiento morfológico— sobre el rendimiento de una Red Neuronal Convolucional (CNN) entrenada para clasificar 5 tipos de flores del dataset [Flowers Recognition](https://www.kaggle.com/datasets/alxmamaev/flowers-recognition) (daisy, dandelion, rose, sunflower, tulip).

---

## Propuesta de Preprocesamiento

El objetivo de nuestra propuesta es **aislar el objeto de interés (la flor) y eliminar el ruido de fondo (hierba, tierra, hojas u otras flores)**, permitiendo que la CNN concentre su aprendizaje exclusivamente en las características morfológicas, texturas y colores del espécimen y no en patrones contextuales del fondo.

El flujo de preprocesamiento aplicado a cada imagen consiste en:

1. **Filtro de Mediana (`cv2.medianBlur` de 3x3):** Suavizado inicial para eliminar ruido de tipo "sal y pimienta" sin difuminar los bordes de la flor.
2. **Proyección HCL a Escala de Grises de Máximo Contraste (HCM):** 
   - Proyecta la imagen RGB mediante una combinación lineal optimizada $I = w_1 R + w_2 G + w_3 B$.
   - Los pesos óptimos se encuentran utilizando el algoritmo de Nelder-Mead (`scipy.optimize.minimize`) para maximizar la desviación estándar de la imagen en grises dividida por su rango (máximo contraste dinámico).
3. **Umbralización Adaptativa de Otsu con Offset ($p = -0.2$):** 
   - Binariza la proyección HCM encontrando automáticamente el umbral óptimo a partir del histograma bimodal.
   - Se aplica un desplazamiento (offset) de $-0.2$ para asegurar que se conserven los bordes finos de los pétalos.
4. **Relleno de Huecos (`fillhole` mediante Flood-Fill):** Rellena los espacios internos vacíos dentro de la máscara binarizada.
5. **Limpieza Morfológica Avanzada (`skimage.morphology`):**
   - Eliminación de objetos aislados pequeños en el fondo con un umbral de área de 200 píxeles (`remove_small_objects`).
   - Relleno de huecos oscuros internos restantes con un umbral de área de 1000 píxeles (`remove_small_holes`).
6. **Enmascaramiento Bitwise (`cv2.bitwise_and`):** Aplica la máscara binaria final sobre la imagen RGB original, resultando en la flor a color sobre un fondo negro absoluto.
7. **Redimensionamiento (`cv2.resize`):** Ajusta las imágenes al tamaño requerido por la red convolucional ($150 \times 150$ píxeles).

---

## Evidencia Visual

A continuación se presenta una comparativa de la imagen original frente a la procesada:

| Imagen Original (RGB) | Máscara de Segmentación | Flor Segmentada (Fondo Negro) |
| :---: | :---: | :---: |
| ![Original](ruta/a/tu/imagen_original.jpg) | ![Máscara](ruta/a/tu/mascara.jpg) | ![Procesada](ruta/a/tu/imagen_procesada.jpg) |

*(Nota: Asegúrate de reemplazar las rutas anteriores con las imágenes reales de ejemplo en tu repositorio).*

---

## Estructura del Repositorio

* `codigo_procesamiento.py`: Script de Python con el pipeline de preprocesamiento completo para aplicar a todo el dataset.
* `cnn/proyecto_final_imágenes_rgb.py`: Notebook/Script con la arquitectura e instrucciones de entrenamiento de la CNN, modificado únicamente con la ruta del dataset procesado.
* `README.md`: Este archivo explicativo de la metodología y resultados.

---

## Instrucciones de Ejecución

### 1. Preprocesamiento de Datos
Para procesar las imágenes originales del dataset, ejecuta el script `codigo_procesamiento.py`. 
* Asegúrate de tener montada tu unidad de Google Drive (si usas Colab) o ajustar las variables de ruta local:
  ```python
  ORIGINAL_DATASET_DIR = "/ruta/a/flowers"
  PROCESSED_DATASET_DIR = "/ruta/a/flores_procesadas"
  ```
* Ejecuta el script:
  ```bash
  python codigo_procesamiento.py
  ```

### 2. Entrenamiento de la CNN
Una vez generadas las imágenes preprocesadas, ejecuta la CNN desde el script en la carpeta `cnn/`.
* Comprueba que la variable `DATASET_DIR` apunte a tu carpeta de imágenes procesadas:
  ```python
  DATASET_DIR = "/content/drive/MyDrive/proyecto_mineria/flores3"
  ```
* Corre el código para entrenar y evaluar la red.

---

## Resumen de Resultados

| Métrica (Validation) | Modelo Original (Sin Preprocesar) | Modelo Propuesto (Segmentado) | Diferencia |
| :--- | :---: | :---: | :---: |
| **Accuracy** | **77%** | **76%** | **-1%** |
| **Precision (Macro Avg)** | **79%** | **76%** | **-3%** |
| **Recall (Macro Avg)** | **77%** | **77%** | **0%** |
| **F1-Score (Macro Avg)** | **76%** | **76%** | **0%** |

### Conclusiones Principales:
* **Girasoles (Sunflowers):** Fue la clase que más se vio beneficiada por el preprocesamiento, aumentando su F1-score de **0.76 a 0.80** y su precisión de **0.62 a 0.71**, ya que aislar la flor eliminó la confusión visual generada por fondos silvestres similares a los dientes de león.
* **Pérdida de Contexto:** Aunque aislar la flor ayuda a enfocar la red, en clases estructuralmente y cromáticamente parecidas como **Rosas** y **Tulipanes**, la eliminación de hojas y tallos (que sirven como contexto de apoyo para la red) causó ligeras confusiones, resultando en una baja marginal del 1% en el accuracy global (77% vs 76%).
