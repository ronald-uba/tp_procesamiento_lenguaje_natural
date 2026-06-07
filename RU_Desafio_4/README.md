**Tabla 1 - Resumen de casos y resultados**
<img width="1723" height="333" alt="image" src="https://github.com/user-attachments/assets/16b7ad4c-dc93-4f28-a0a5-90665020b468" />


Siguiendo las consignas planteadas para el desafío se analizan 10 modelos alternativos al caso base visto en clase (Tabla 1).

Se toman como métricas de evaluación [val_accuracy], [val_loss] y [valoracion_traduccion]. Para la valoración de la traducción se analiza cualitativamente la traducción de 6 nuevas frases agregadas al notebook, de acuerdo con estos criterios:

a) totalmente incorrecta = 0 puntos

b) parcialmente correcta = 1 punto

c) correcta o muy cercana = 2 puntos



De detallan a continuación los modelos evaluados y sus resultados:

**01_caso_base:** modelo traductor visto en clase, para tomarlo como baseline. Tomado como referencia, tiene val_accuracy de 0.7851y val_loss de 1.4461. La calidad de traducción es baja (3/12).

**02_mas_datos_30k:** se amplia la cantidad de oraciones del corpus a 30k. El modelo muestra una importante mejora en las 3 métricas de evaluación. Val_accuracy aumenta 5% y val_loss se reduce 35%, entanto que la calidad de la traducción mejora a 9/12. Esto muestra que el modelo se benefició la mayor cantidad de ejemplos, logrando traducciones más coherentes y completas.

**03_mas_datos_50k:** se amplia la cantidad de oraciones del corpus a 50k con la expectativa de mayor mejora pero el resultado es el contrario. Val_accuracy y val_loss mejoraron en forma extrema (0,9976 y 0,0245, respectivamente). Sin embargo, la evaluación cualitativa es 0/12, con traducciones repetitivas y semánticamente incorrectas. Podría deberse a que un mayor volumen de datos requiera un vocabulario ampliado. Se decide descartar esta configuración, a pesar de sus métricas aparentemente superiores.

**04_mas_vocabulario_14k:** sobre la base de [02_mas_datos_30k] se amplia MAX_VOCAB_SIZE a 14k. Aumentar el vocabulario no mejoró el desempeño. Las 3 metricas de evaluación muestran un leve deterioro. Se infiere que un vocabulario más extenso aumenta innecesariamente la complejidad en la salida del decoder.

**05_max_input_len_30:** sobre la base de [02_mas_datos_30k] se amplia la longitud de maxíma de input y output a 30 y 33 palabras. El modelo muestra otra importante mejora en las 3 métricas de evaluación. Val_accuracy aumenta 7% y val_loss se reduce 33% sobre el mejor modelo anterior, mientras que la calidad de la traducción mejora a 10/12. Inferimos que al permitir secuencias más largas, el modelo puede generar traducciones de mejor calidad. Este es el mejor modelos entre los evaluados (post descarte de [03_mas_datos_50k])

**06_n_unitas_512:** sobre la base de [05_max_input_30] se amplia la cantidad de unidades recurrentes de las LSTM de 256 a 512. Los resultados son similares a los del mejor modelo anterior en las 3 métricas de evaluación. Agregar más unidades aumenta el costo computacional, pero no produce una mejora relevante e incrementó el costo computacional. Inferimos que con estos datos el cuello de botella no es la cantidad de unidades recurrentes.

**07_n_unitas_128:** sobre la base de [05_max_input_30] se reduce la cantidad de unidades recurrentes de las LSTM de 256 a 128 para evaluar el impacto. En este caso sí aparecen cuellos de botella. Val accuracy empeora 1% y val_los un 10%, en tanto que la calidad de la traducción se reduce a 7/12. Se evidencia que una arquitectura más chica pierde calidad de traducción.

**08_embeddings_espanol:** a la base de [05_max_input_30] se incorporan en el decoder embeddings preentrenados en español, sin entrenarlos. Se usan vectores de dimension 300 por ser los disponibles. Esto incrementa marcadamente el tiempo de entrenamiento (además del tiempo extra de carga). Las 3 métricas de evaluación empeoraran vs en mejor modelo anterior, que entrena los embeddings del decoder desde cero.

**09_embeddings_espanol_entrenados:** a la base de [05_max_input_30] se incorporan en el decoder embeddings preentrenados en español, pero esta vez sí se entrenan. Esto mejora las métricas respecto de dejarlos fijos, pero todavía con peor desempeño que el mejor modelo anterior [05_max_input_30]. Esto indica que adaptar los embeddings ayuda un poco, pero no alcanza -en este caso- para superar al embedding aprendido desde cero.

**10_beam_search:** sobre la base de [05_max_input_30] se cambia la estrategia de generación de greedy a beam search. Esto no cambia el entrenamiento sino la inferencia. La calidad de las traducciones empeora vs la versión del modelo con estrategia greedy.

**11_datos_50k_vocab_14k:** en los casos anteriores se sigue una estrategia greedy en la que sólo se incorporan ajustes de una varible que por sí sola mejore el modelo. Dado el desempeño anómalo de aumentar a 50k las oraciones, se incorpora un aumento de vocabulario máximo a 14k. El impacto combinado de ambos cambios muestra muy buen desempeño. Si bien no supera en val_accuracy y val_loss a [05_max_input_30] (ocupa el segundo lugar) lo equipara en cuanto a la calidad de las traducciones.

# Conclusiones generales

Los resultados muestran que las mejoras más importantes provienen del aumento del conjunto de entrenamiento de 10k a 30k oraciones y del incremento de la longitud máxima de las secuencias. En cambio, aumentar el vocabulario, incrementar las unidades LSTM a 512, incorporar embeddings preentrenados en español o utilizar beam search no produce mejoras claras. La mejor configuración es la de 30k oraciones, vocabulario de 8k, longitudes máximas de 30 y 33 tokens, 256 unidades LSTM y greedy decoding, alcanzando la mejor valoración cualitativa de traducción.

El aumento aislado a 50k oraciones lleva al modelo a su peor desempeño, a pesar de que val_accuracy y val_loss son en ese caso especialemente buenos. Esto mustra que estás métricas pueden no correlacionar siempre con la calidad de la traducción. La evaluación cualitativa de las traducciones es un complemento clave en la comparación.

La combinación de 50k oraciones y un vocabulario máximo de 14k palabras tienen el segundo mejor desempeño, con igual calidad de traducción que el mejor modelo. Esto muestra que la anomalía en los resultados de [03_mas_datos_50k] se relaciona con un vocabulario insuficiente para el nuevo volumen de datos. Esto muestra claramente que la estrategia greedy de ajuste variables puede ser limitada a la hora de elegir la mejora combinación de ajustes.
