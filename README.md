# Laboratorio 1 – Detección de Phishing (URLs)
**Curso:** CC3094 – Security Data Science (UVG)  
**Tema:** Clasificación de URLs (legitimate vs phishing) usando features basadas en la URL

## Objetivo
Construir un clasificador que detecte URLs de phishing **usando únicamente información de la URL** (sin analizar HTML/WHOIS).  
Para eso se generan features numéricas, se separa el dataset en train/val/test y se entrenan **dos modelos** para comparar resultados.

---

## Dataset
Archivo: `dataset_pishing.csv`  
Columnas: `url`, `status`

- Total: 11,430 registros
- Clases: 5,715 `legitimate` y 5,715 `phishing` (**50/50**, dataset balanceado)
- Se encontró 1 URL duplicada (se eliminó antes de entrenar)

---

## ¿Qué se hizo?
### 1) Ingeniería de características (15+)
Se generaron **33 features** a partir de la URL, por ejemplo:
- Longitudes: `url_length`, `host_length`, `path_length`, `query_length`
- Conteos de símbolos: puntos, guiones, slashes, `@`, `&`, `=`, etc.
- Ratios: proporción de dígitos y caracteres especiales
- Estructura: subdominios, si el host parece IP
- Palabras clave: “login”, “verify”, “secure”, “update”, etc.
- Entropías: **Shannon** y **entropía relativa**

### 2) Preprocesamiento
- `status` a binario: `phishing = 1`, `legitimate = 0`
- Eliminación de duplicados por URL
- Split estratificado **55/15/30**:
  - Train: 6,285
  - Val: 1,715
  - Test: 3,429

> Nota: La distribución base para entropía relativa se calculó usando **solo URLs legítimas del train**, para evitar mezclar información del val/test.

### 3) Selección simple de features
Se revisó varianza y correlación:
- No hubo columnas de varianza casi 0 (`low-variance cols: []`)
- No aparecieron pares casi duplicados por correlación (> 0.95)
Por eso se mantuvieron las 33 features.

---

## Modelos entrenados y resultados
Se entrenaron 2 modelos:
1) **Logistic Regression**
2) **Random Forest**

### Resultados (Validación)
**Logistic Regression**
- Precision: 0.8086
- Recall: 0.7048
- AUC: 0.8715
- CM:
  - [[715, 143],
     [253, 604]]

**Random Forest**
- Precision: 0.8660
- Recall: 0.8518
- AUC: 0.9348
- CM:
  - [[745, 113],
     [127, 730]]

### Resultados (Prueba)
**Logistic Regression**
- Precision: 0.8301
- Recall: 0.7526
- AUC: 0.8895
- CM:
  - [[1451, 264],
     [424, 1290]]

**Random Forest**
- Precision: 0.8713
- Recall: 0.8611
- AUC: 0.9399
- CM:
  - [[1497, 218],
     [238, 1476]]

**Conclusión:** El mejor modelo fue **Random Forest**, porque obtuvo mejor AUC y recall en validación y prueba.

---

## Base rate (50,000 correos; 15% phishing)
Con Random Forest (test):
- FPR ≈ 0.1271
- Recall (TPR) ≈ 0.8611

Estimación esperada:
- TP ≈ 6458.6
- FN ≈ 1041.4
- FP ≈ 5402.3
- TN ≈ 37097.7
- Alertas totales ≈ 11860.9

Interpretación rápida: detecta bastante phishing, pero aún genera varias alertas falsas. Para bajar FP se puede ajustar el umbral (no usar 0.5 fijo) o usar un proceso en 2 pasos para revisar casos dudosos.

---

## Archivos del repo
- `LAB01_Phishing_URLs_Colab.ipynb` : notebook principal (Colab)
- `dataset_pishing.csv` : dataset original
- `train_split.csv`, `val_split.csv`, `test_split.csv` : splits exportados
- `README.md` : este archivo

---

## Cómo ejecutar (Google Colab)
1) Abrir el notebook en Colab.
2) Subir `dataset_pishing.csv` al panel de archivos de Colab (o montarlo desde Drive).
3) Verificar que `DATA_PATH` apunte al archivo (`/content/dataset_pishing.csv`).
4) Ejecutar celdas en orden.
5) Al finalizar se generan: `train_split.csv`, `val_split.csv`, `test_split.csv`.

---

## Dependencias
- Python 3
- pandas, numpy
- scikit-learn
- matplotlib

## Prompt Utilizado 

Necesito que me ayudes como tutor con mi laboratorio de detección de phishing usando URLs. Quiero que me expliques qué pasos seguir para explorar el dataset, crear features (incluyendo entropía) y hacer el split 55/15/30 de forma estratificada. También guíame para entrenar dos modelos sencillos y entender qué métricas debo reportar (matriz de confusión, precision, recall y AUC). Ayúdame a redactar las respuestas de los incisos con un lenguaje claro y natural, sin sonar demasiado técnico.
