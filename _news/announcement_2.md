---
layout: post
title: "Semantic Entropy: Un enfoque probabilístico para detectar confabulaciones en LLMs"
date: 2025-10-30 10:00:00-0400
inline: false
related_posts: false
---

La investigación reciente de Farquhar, Kossen, Kuhn y Gal (Universidad de Oxford, 2024) representa un avance significativo en la detección de alucinaciones en modelos de lenguaje de gran escala.

#### Contribución principal

El paper propone utilizar **entropía semántica** como métrica para cuantificar la incertidumbre epistémica del modelo. A diferencia de enfoques previos basados en entropía léxica, este método evalúa la variabilidad del *significado* entre múltiples generaciones, no solo la distribución de tokens.

#### Relevancia metodológica

La técnica presenta tres ventajas fundamentales:

1. **Invarianza semántica**: Agrupa respuestas equivalentes en significado mediante modelos de bi-direccionalidad
2. **Detección temprana**: Identifica confabulaciones antes de la generación completa
3. **Aplicabilidad general**: Funciona sin reentrenamiento del modelo base

#### Próxima publicación

Desarrollaré un análisis técnico completo incluyendo:

- Formalización matemática accesible del método
- Comparativa con técnicas de calibración alternativas  
- Implementación reproducible con evaluación empírica

Publicación prevista: Primera semana de noviembre 2025.
 
