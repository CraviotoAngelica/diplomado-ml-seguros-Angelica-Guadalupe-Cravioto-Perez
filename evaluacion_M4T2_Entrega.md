# Evaluación — Módulo 4 · Tema 2: GLM con Python

**Alumno:** Angelica Guadalupe
**Variable asignada:** `uso`  (uso)
**Fecha de entrega:** 13 de Septiembre del 2026

> **Instrucciones.** Este archivo evalúa las tres sesiones del tema. Las tablas ya vienen
> calculadas; tu trabajo es **responder las preguntas de interpretación** en el espacio
> "**Tu respuesta:**". Se evalúa la interpretación, no el código. Máx. 4–6 líneas por respuesta.
> Todas tus preguntas usan **tu variable asignada** (`uso`). Guarda y sube este archivo a tu repositorio.

---

## Parte 1 · Sesión 1 — Modelo de Frecuencia

**Diagnóstico del supuesto de Poisson (modelo completo):**

| métrica | valor |
| --- | --- |
| φ de Pearson | 1.1664 |
| Cameron-Trivedi α | 0.0744 |
| z | 15.80 |
| p-value | 3.7e-56 |

**Rating factors de frecuencia para `uso`** (base × RF reproduce la tasa empírica; diferencia máx = 1.7e-05):

| nivel | RF_frec | IC_inf | IC_sup | p | tasa_emp |
| --- | --- | --- | --- | --- | --- |
| Particular (ref) | 1 | 1 | 1 | 0 | 0.1393 |
| Trabajo | 0.9887 | 0.9272 | 1.0542 | 0.7281 | 0.1377 |

**P1.** ¿Se cumple la equidispersión? Justifica con φ **y** con Cameron-Trivedi, y di qué familia usarías.


*Respuesta:*


No se cumple la equidispersión, ya que en un modelo Poisson se espera que la media y la varianza sean aproximadamente iguales. El φ de Pearson es 1.1664, indicando una sobredispersión leve. Esto se confirma con Cameron-Trivedi (p-value = 3.7e-56), por lo que se rechaza la equidispersión. Al ser una sobredispersión leve (φ < 1.5), utilizaría un modelo QuasiPoisson.


**P2.** Interpreta los rating factors de tu variable: nivel más alto y más bajo, traducidos a % de
recargo/descuento. ¿Algún IC cruza 1 o tiene p > 0.05? ¿Qué harías con ese nivel?


*Respuesta:*


Particular es la categoría de referencia, con un RF de 1, por lo que representa un ajuste de 0%. Trabajo tiene el RF más bajo, de 0.9887, equivalente a un descuento de aproximadamente 1.13% respecto a Particular. Su IC [0.9272, 1.0542] cruza el 1 y además tiene p-value = 0.7281 > 0.05, por lo que el efecto no es estadísticamente significativo. Por ello, consideraría agrupar Trabajo con Particular.


**P3.** ¿Por qué el GLM one-way reproduce exactamente la tasa empírica, y qué aporta el GLM que una
tabla empírica no puede dar?


*Respuesta:*


El GLM one-way reproduce la tasa empírica porque, en el modelo Poisson con enlace log, las ecuaciones de score hacen que la suma de los valores predichos coincida con la suma de los observados por cada nivel. En particular, `exp(η)=Σn/Σe`, que corresponde a la tasa empírica. La ventaja del GLM es que permite combinar varias variables de forma multiplicativa y obtener errores estándar, intervalos de confianza y pruebas de significancia, algo que una tabla empírica por sí sola no proporciona.


---

## Parte 2 · Sesión 2 — Severidad y Selección de Modelos

**Comparación de modelos de frecuencia:**

| modelo | AIC | BIC | pseudoR2_McF |
| --- | --- | --- | --- |
| Poisson | 125,081.7 | 125,261.7 | 0.0198 |
| Binomial Negativa | 124,925.7 | 125,105.8 | 0.021 |

**Rating factors de severidad (Gamma) para `uso`:**

| nivel | RF_sev | severidad_emp |
| --- | --- | --- |
| Particular (ref) | 1 | 1,310 |
| Trabajo | 0.9868 | 1,293 |

**P4.** ¿Por qué se usa **Gamma** para severidad y no una regresión lineal sobre log(Y)? (menciona la
propiedad del CV y por qué Lognormal no es GLM).


*Respuesta:*


Gamma es adecuada para modelar la severidad porque trabaja con valores positivos y supone un coeficiente de variación constante, es decir, que la variabilidad relativa de los siniestros se mantiene aproximadamente estable. Además, el GLM Gamma modela directamente la severidad esperada E[Y]. En cambio, una regresión sobre log(Y) modela E[log Y], no directamente E[Y], por lo que al regresar a la escala original se requiere una corrección por sesgo. La Lognormal no pertenece a la familia exponencial natural de los GLM tradicionales.


**P5.** Según la tabla de comparación, ¿qué modelo elegirías? Justifica con AIC/BIC. ¿Por qué el pseudo R²
es tan bajo y eso NO significa que el modelo sea malo?


*Respuesta:*


Elegiría la Binomial Negativa porque presenta un AIC y BIC menores que Poisson: 124,925.7 vs. 125,081.7 y 125,105.8 vs. 125,261.7, respectivamente. Como en estos criterios un valor menor indica un mejor balance entre ajuste y complejidad, ambos favorecen a la Binomial Negativa. El pseudo R² de 0.02 es bajo, pero esto es común en seguros por la variabilidad propia de los siniestros. Por ello, es más importante comparar los modelos y revisar su capacidad de discriminación.


**P6.** Compara tus rating factors de frecuencia (Parte 1) con los de severidad para `uso`. ¿Apuntan en
la misma dirección? ¿Qué implica eso para separar Frecuencia × Severidad?


*Respuesta:*


Los rating factors de frecuencia y severidad apuntan en la misma dirección para el uso Trabajo. En frecuencia, el RF es 0.9887, que representa una disminución de 1.13%, y en severidad es 0.9868, una disminución de 1.32%, respecto a Particular. Esto indica que Trabajo presenta tanto una menor frecuencia como una menor severidad. Por ello, es útil modelar por separado Frecuencia × Severidad para identificar cómo cada variable afecta la prima pura.


---

## Parte 3 · Sesión 3 — Validación y Tarifa

**Validación out-of-sample del modelo de frecuencia:**

| metrica | valor | ideal |
| --- | --- | --- |
| Gini (test) | 0.2315 | > 0.30 aceptable |
| Ratio pred/obs (test) | 1.0249 | ≈ 1.00 |

**Prima pura por nivel de `uso`** (Frecuencia × Severidad, con su factor de tarifa):

| nivel | prima_pura_modelo | factor_tarifa |
| --- | --- | --- |
| Particular (ref) | 182.53 | 1.0011 |
| Trabajo | 178.57 | 0.9794 |

**P7.** Interpreta las métricas de validación: ¿el modelo está bien calibrado (ratio pred/obs)? ¿discrimina
bien el riesgo (Gini)? ¿Qué mide cada una?


*Respuesta:*


El ratio pred/obs es 1.0249, cercano a 1, por lo que el modelo está bien calibrado en promedio, ya que las predicciones son similares a los valores observados. Por otro lado, el Gini es 0.2315, menor al 0.30 tomado como referencia, por lo que la capacidad de discriminación es limitada. El ratio pred/obs evalúa la calibración del modelo, mientras que el Gini mide qué tan bien el modelo diferencia y ordena los riesgos de menor a mayor.


**P8.** Lee la tabla de tarifa: ¿qué nivel de tu variable paga la prima pura más alta y cuál la más baja?
Traduce el factor de tarifa a un recargo/descuento sobre la prima promedio.


*Respuesta:*


El nivel Particular presenta la prima pura más alta, con $182.53, mientras que Trabajo presenta la más baja, con $178.57. El factor de tarifa de Particular es 1.0011, que equivale a un recargo de 0.11% sobre la prima promedio. Para Trabajo, el factor de tarifa de 0.9794 representa un descuento de 2.06% sobre la prima promedio. Por lo tanto, Trabajo tendría una tarifa ligeramente menor que Particular.


**P9. (Conclusión de nota técnica).** En 3–4 líneas, redacta cómo `uso` afecta la tarifa, integrando
frecuencia, severidad y prima pura, en estilo defendible ante la CNSF.


*Respuesta:*


El uso Trabajo presenta una frecuencia 1.13% menor y una severidad 1.32% menor respecto a Particular, resultando en una prima pura de $178.57 frente a $182.53. El factor de tarifa para Trabajo es 0.9794, equivalente a un descuento de 2.06% respecto a la prima promedio. Sin embargo, el efecto de frecuencia no es estadísticamente significativo (RF = 0.9887; IC: [0.9272, 1.0542]). Por lo tanto, la diferenciación tarifaria por uso debe sustentarse en evidencia estadística y validarse antes de su aplicación.



---
*Evaluación generada automáticamente · Diplomado ML en Seguros · FC UNAM · Módulo 4 · Tema 2*
