# Instrucciones para ejecutar

1. Asegúrate de que el archivo `.ipynb` y la carpeta `data/` estén en la misma ubicación, para que Jupyter pueda acceder a la información y entrenar/validar el modelo.

2. El código entrena un modelo en **fold1 → fold2** y otro en **fold2 → fold1**, mostrando métricas de ambos experimentos y el promedio final de validación cruzada.

---

# Modelo seleccionado

El modelo contiene los siguientes componentes:

- **ResNet18**: red neuronal convolucional de 18 capas preentrenada con *ImageNet*.  
  - Se reemplaza la última capa totalmente conectada (`fc`) por una nueva capa lineal con 7 salidas (una por clase).  
  - Se entrena en **dos fases**:  
    1. **Head training**: solo la capa final, manteniendo el backbone congelado.  
    2. **Fine-tuning**: todo el modelo se entrena con un *learning rate* menor.

Este enfoque combina la potencia de las CNN preentrenadas con la simplicidad de entrenar solo las capas necesarias en cada fase, mejorando desempeño y evitando sobreajuste.

---

# Técnicas de preprocesamiento

- **Redimensionamiento y recorte**  
  - Train: `Resize(256)` + `RandomResizedCrop(224)`  
  - Test: `Resize(256)` + `CenterCrop(224)`  
- **Aumentos (solo en entrenamiento)**:  
  - Flips horizontales y verticales.  
  - Rotaciones aleatorias (±10°).  
  - *Color jitter* para pequeñas variaciones de brillo, contraste, saturación y tono.  
- **Normalización**: valores de píxeles reescalados con las medias y desviaciones estándar de *ImageNet*:  
  `(0.485, 0.456, 0.406)` y `(0.229, 0.224, 0.225)`.  
- **Balanceo de clases**: uso de `WeightedRandomSampler` y pérdida `CrossEntropyLoss` con pesos por clase.

---

# Hiperparámetros utilizados

- **Semilla aleatoria**: `42`  
- **Tamaño de imagen**: `224 × 224`  
- **Batch size**: `32`  
- **Épocas**:  
  - Fase 1 (solo cabeza): `4`  
  - Fase 2 (fine-tuning): `8`  
- **Learning rate**:  
  - Head: `1e-3`  
  - Fine-tuning: `1e-4`  
- **Sampler**: balanceado según número de imágenes por clase.

---

# Métricas de evaluación

El modelo se evaluó con validación cruzada de **2 folds** (entrenando en un fold y validando en el otro, y viceversa).  
Las métricas obtenidas fueron las siguientes:

**Fold A (train=fold1 / test=fold2)**  
- Accuracy: `0.6696`  
- Macro-F1: `0.6374`  
- Loss: `1.0424`  

**Fold B (train=fold2 / test=fold1)**  
- Accuracy: `0.6358`  
- Macro-F1: `0.5981`  
- Loss: `1.0302`  

**Promedio (2-fold CV)**  
- Accuracy: **0.6527**  
- Macro-F1: **0.6178**

> El modelo logra un desempeño sólido diferenciando tejido normal y algunos subtipos de SCC, aunque aún muestra confusión entre grados de adenocarcinoma y carcinoma escamoso debido a la similitud morfológica y al desbalance de clases.

---

# Artefactos generados

Tras la ejecución, se guardan en la carpeta `outputs/`:

- Métricas en formato `.json`.  
- Matrices de confusión (`confusion_matrix.png`).  
- Mejor modelo entrenado (`best.pt`).  
- Resumen de validación cruzada (`cv_summary.json`).
