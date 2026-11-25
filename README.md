# Seq2Seq Neural Machine Translation

Proyecto de traducción automática neuronal con una organización modular para separar el manejo de datos, los modelos, el entrenamiento y la evaluación. Esta estructura sirve como base para desarrollar el proyecto; todavía no incluye implementaciones de modelos.

## Estructura

```text
.
├── configs/             # Configuraciones del proyecto y experimentos
├── data/
│   ├── raw/             # Datos originales
│   └── processed/       # Datos preparados para los experimentos
├── notebooks/           # Exploración y experimentos
├── outputs/
│   ├── checkpoints/     # Pesos y checkpoints generados
│   ├── metrics/         # Métricas y resultados de evaluación
│   └── predictions/     # Traducciones generadas
├── src/
│   ├── data/            # Carga y preprocesamiento de datos
│   ├── models/          # Componentes y arquitecturas de modelos
│   ├── training/        # Lógica de entrenamiento
│   ├── evaluation/      # Métricas y evaluación
│   ├── inference/       # Generación de traducciones
│   └── utils/           # Utilidades compartidas
└── tests/               # Pruebas del proyecto
```

Los datos y resultados generados se mantienen separados del código fuente.
