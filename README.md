# Traducción automática neuronal: inglés → español

Proyecto experimental para comparar cuatro enfoques de traducción automática neuronal (NMT) EN→ES: modelos recurrentes entrenados desde cero, un Transformer entrenado desde cero y ajuste fino de un modelo preentrenado. La organización propuesta separa los componentes compartidos de las arquitecturas para que las comparaciones sean reproducibles y fáciles de ampliar.

## Arquitectura experimental

| ID | Modelo | Entrenamiento | Pregunta que responde |
|---|---|---|---|
| `rnn-small` | Encoder–decoder BiLSTM sin atención, capacidad pequeña | Desde cero | ¿Qué rendimiento ofrece un baseline secuencial sencillo? |
| `rnn-attention` | Encoder–decoder BiLSTM con atención aditiva, capacidad mayor | Desde cero | ¿Qué aporta la atención y cómo se comporta un modelo recurrente de mayor capacidad? |
| `transformer` | Encoder–decoder Transformer | Desde cero | ¿Cómo se compara la autoatención sin recurrencia con el sesgo secuencial? |
| `marian-finetune` | Helsinki-NLP/opus-mt-en-es (MarianMT) | Ajuste fino | ¿Qué ganancia aporta el transfer learning frente al entrenamiento desde cero? |

Para atribuir diferencias a la arquitectura y no al tamaño, conviene igualar el número de parámetros de `rnn-attention` y `transformer` (con una tolerancia predefinida, por ejemplo ±5 %) y usar el mismo corpus, particiones, presupuesto de entrenamiento y criterio de parada. `rnn-small` sirve como baseline de capacidad reducida; no debe interpretarse como una comparación controlada de arquitectura con los modelos grandes. La comparación entre `rnn-small` y `rnn-attention`, junto con sus curvas de entrenamiento y validación, permite estudiar señales de sobreajuste, aunque una conclusión causal sobre capacidad requiere controlar la arquitectura. Para aislar mejor ese efecto, se recomienda ejecutar variantes pequeña/grande de una misma arquitectura como ablation adicional.

## Organización recomendada

```text
.
├── configs/
│   ├── data.yaml                 # Corpus, idiomas y semilla de partición
│   ├── training.yaml             # Batch size, épocas, optimizador y early stopping
│   └── models/                   # Hiperparámetros por modelo y presupuesto
├── data/                         # Datos descargados/procesados; ignorados por Git
├── docs/
│   └── experimental_protocol.md  # Decisiones, limitaciones y resultados
├── notebooks/                    # Análisis y visualización; no lógica de entrenamiento
├── outputs/                      # Checkpoints, logs y predicciones; ignorados por Git
├── src/nmt/
│   ├── data/                     # Carga, limpieza, partición y tokenización
│   ├── models/                   # Interfaces y las cuatro arquitecturas
│   ├── training/                 # Bucle común, checkpoints y early stopping
│   ├── evaluation/               # Métricas, inferencia y análisis de errores
│   └── cli.py                    # Entrenar, evaluar y traducir
└── tests/                        # Pruebas unitarias y pruebas de humo
```

La capa de datos debe proporcionar una interfaz común para corpus EN–ES, eliminar pares vacíos o duplicados y fijar las particiones `train`, `validation` y `test` antes de entrenar. Ajustar el vocabulario/tokenizador solo con `train`; reutilizar el tokenizador del checkpoint para MarianMT. No usar `test` para seleccionar hiperparámetros ni para early stopping. Versionar configuraciones y metadatos, no los corpus, checkpoints ni salidas grandes.

Las cuatro implementaciones deberían compartir el contrato de entrada/salida, máscara de padding, manejo de BOS/EOS, pérdida con padding ignorado y rutina de generación. La configuración de cada ejecución debe registrar semilla, revisión del corpus y del checkpoint, vocabulario/tokenizador, número de parámetros, hiperparámetros, dispositivo, duración y mejor época. Ejecutar varias semillas cuando los recursos lo permitan e informar media y dispersión.

## Protocolo de evaluación

1. Definir el corpus y publicar la fuente, licencia, filtrado y tamaños de las particiones.
2. Entrenar los tres modelos desde cero con el mismo conjunto de entrenamiento y una política de selección basada solo en validación. Ajustar MarianMT sobre el mismo conjunto de entrenamiento y validación.
3. Guardar para cada experimento su configuración, checkpoint seleccionado, curva de pérdida y predicciones del conjunto de prueba.
4. Calcular BLEU con SacreBLEU (incluyendo la firma/configuración), chrF y, si está disponible, una métrica semántica como COMET. Informar el método de decodificación y la longitud máxima.
5. Comparar las curvas de entrenamiento y validación para discutir sobreajuste; comparar BLEU/chrF en test para calidad. Añadir ejemplos de errores cualitativos y limitaciones.

No comparar puntuaciones obtenidas con particiones, tokenización o decodificación diferentes. Una sola semilla o una diferencia pequeña en BLEU no constituye evidencia concluyente.

## Flujo de trabajo sugerido

```text
configuración → preparación de datos → entrenamiento de cada modelo
              → selección en validation → evaluación única en test
              → tabla de métricas + análisis de errores
```

Mantén la carga de datos, entrenamiento y evaluación en módulos reutilizables; los notebooks deben consumir sus resultados en lugar de implementar otro pipeline. Añade comandos reproducibles de instalación, pruebas, entrenamiento y evaluación cuando se incorpore el código ejecutable. El entrenamiento de los cuatro modelos y la descarga de corpus/checkpoints requieren recursos y datos externos; este repositorio inicial no los incluye.
