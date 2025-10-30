---
layout: post
title: "¿Por qué los LLMs alocinan? Detectando confabulaciones mediante entropía semántica"
date: 2025-10-29 14:00:00-0400
description: "Análisis del método propuesto por investigadores de Oxford para detectar cuándo los modelos de lenguaje están inventando información, publicado en Nature 2024."
tags: LLMs alucinaciones entropía-semántica machine-learning NLP
categories: investigación
thumbnail: assets/img/hallucination-thumbnail.png
giscus_comments: true
related_posts: true
toc:
  sidebar: left
---

Le pregunté a ChatGPT quién inventó el teléfono. Me respondió con total seguridad: "Alexander Graham Bell, en 1876". Reformulé la pregunta manteniendo la misma intención. Esta vez me dijo: "Antonio Meucci fue el verdadero inventor, Bell solo patentó el dispositivo". Ambas respuestas suenan convincentes, están bien estructuradas y carecen de cualquier indicador de duda. ¿Cuál es correcta? Esta capacidad de los modelos de lenguaje para generar información falsa con aparente certeza es uno de los problemas más críticos de la inteligencia artificial actual.

Las **alucinaciones en Large Language Models** (LLMs) no son errores ocasionales ni fallas técnicas menores. Son manifestaciones fundamentales de cómo estos sistemas procesan y generan información. Un modelo puede confabular datos históricos, inventar referencias bibliográficas inexistentes o fabricar razonamientos aparentemente lógicos pero completamente erróneos. En contextos de alto riesgo—diagnósticos médicos, asesoramiento legal, análisis financiero—estas confabulaciones pueden tener consecuencias graves.

El desafío es detectarlas antes de que el usuario las asuma como verdaderas. Un paper reciente publicado en Nature por investigadores de la Universidad de Oxford propone una solución elegante: utilizar **entropía semántica** para cuantificar la incertidumbre del modelo y anticipar cuándo está a punto de alucinar.

## El problema: cuando la confianza no refleja certeza

Los LLMs actuales, incluyendo GPT-4, Claude y Gemini, generan texto de manera autorregresiva: predicen la siguiente palabra basándose en las anteriores. En cada paso, el modelo asigna probabilidades a miles de posibles continuaciones y selecciona según esas distribuciones. Aquí surge el primer problema: **la confianza estadística no equivale a exactitud factual**.

Un modelo puede asignar alta probabilidad a una secuencia de tokens sin que esa secuencia represente información verídica. La métrica tradicional de **perplexity** mide qué tan "sorprendido" está el modelo por su propia generación, pero no captura si el contenido semántico es correcto. Un LLM puede estar muy seguro de una afirmación incorrecta si su corpus de entrenamiento contenía información errónea o si aprendió correlaciones espurias.

Los métodos previos de detección de alucinaciones se enfocaban en medir la **entropía léxica**: la variabilidad en las palabras específicas que el modelo genera al muestrear múltiples respuestas. Sin embargo, dos respuestas pueden usar vocabulario completamente diferente pero expresar el mismo significado. Considere estas dos generaciones:

> "La capital de Francia es París."  
> "París es la ciudad capital francesa."

Léxicamente distintas, semánticamente idénticas. La entropía léxica fallaría en distinguir esta consistencia semántica de una verdadera contradicción.

## La solución: entropía semántica como métrica de incertidumbre

El trabajo de Farquhar, Kossen, Kuhn y Gal introduce un cambio de paradigma: en lugar de medir variabilidad en tokens, **miden variabilidad en significados**. El método se fundamenta en una intuición simple pero poderosa: si un modelo realmente "sabe" la respuesta a una pregunta, múltiples muestreos deberían generar respuestas diferentes en forma pero equivalentes en contenido. Si el modelo está confabulando, las respuestas variarán no solo superficialmente sino semánticamente.

El proceso opera en tres etapas fundamentales:

**1. Generación de múltiples respuestas**  
Para una pregunta dada, el modelo genera *n* respuestas independientes mediante muestreo estocástico. No se trata de simplemente ejecutar el modelo varias veces con temperatura cero (lo que produciría respuestas idénticas), sino de permitir exploración en el espacio de probabilidades mediante sampling con temperatura positiva.

**2. Agrupación semántica mediante bidirectional entailment**  
Aquí reside la innovación técnica clave. Los autores utilizan un modelo auxiliar para determinar si dos respuestas son semánticamente equivalentes. Dos afirmaciones son equivalentes si cada una implica lógicamente a la otra (implicación bidireccional). Las respuestas se agrupan en clústeres de significado mediante este criterio.

Por ejemplo, ante la pregunta "¿Cuál es la velocidad de la luz?", estas respuestas pertenecerían al mismo clúster semántico:
- "Aproximadamente 300,000 km/s"
- "3×10⁸ metros por segundo"
- "La luz viaja a cerca de 300,000 kilómetros cada segundo"

Mientras que estas formarían clústeres distintos:
- "La velocidad de la luz es constante en el vacío"
- "Einstein demostró que nada puede viajar más rápido que la luz"

**3. Cálculo de entropía sobre distribuciones semánticas**  
Una vez agrupadas las respuestas por significado, se calcula la entropía de Shannon sobre la distribución de probabilidad de estos clústeres semánticos. Si todas las respuestas caen en un solo clúster, la entropía es cero (máxima certeza). Si las respuestas se distribuyen uniformemente entre múltiples significados inconsistentes, la entropía es alta (alta incertidumbre).

Formalmente, si *p(c)* es la probabilidad del clúster semántico *c*, la entropía semántica se define como:

**H(C) = -Σ p(c) log p(c)**

Esta métrica captura la incertidumbre epistémica del modelo: su desconocimiento sobre cuál es la respuesta correcta.

## Por qué funciona: fundamentos probabilísticos

La entropía semántica opera bajo el supuesto de que un modelo bien calibrado debería expresar su incertidumbre mediante la diversidad de sus generaciones. Cuando el modelo tiene evidencia clara en sus parámetros para responder una pregunta, el muestreo estocástico explorará regiones del espacio latente que, aunque produzcan variaciones superficiales, convergen al mismo significado.

En contraste, cuando el modelo carece de información suficiente, recurre a completaciones plausibles basadas en correlaciones estadísticas del entrenamiento. Estas completaciones pueden ser individualmente coherentes pero mutuamente inconsistentes en contenido. La entropía semántica detecta precisamente esta inconsistencia.

Los autores demuestran empíricamente que su método supera significativamente a baselines previas en múltiples benchmarks de question-answering. En tareas de TriviaQA y SQuAD, la entropía semántica alcanza una AUC-ROC superior a 0.85 en la detección de respuestas incorrectas, mientras que métricas basadas en perplexity o entropía léxica rondan 0.65-0.70.

Crucialmente, el método es **model-agnostic**: funciona con cualquier LLM que permita sampling, sin requerir acceso a los pesos internos ni reentrenamiento. Esto contrasta con enfoques que dependen de fine-tuning o modificaciones arquitectónicas.

## Implicaciones prácticas y limitaciones

La aplicabilidad de este método trasciende la academia. Sistemas de asistencia médica basados en LLMs podrían señalar automáticamente recomendaciones con alta entropía semántica para revisión humana. Motores de búsqueda augmentados con LLMs podrían advertir a usuarios cuando la respuesta generada tiene baja confiabilidad semántica.

Sin embargo, el método enfrenta desafíos de escalabilidad. Generar múltiples respuestas y calcular entailment bidireccional tiene costo computacional significativo. Para aplicaciones en tiempo real, se requeriría optimización mediante técnicas de early stopping o aproximaciones del cálculo de entropía con menos muestras.

Además, la calidad de la detección depende críticamente del modelo de entailment utilizado para agrupar respuestas. Si este modelo auxiliar comete errores al determinar equivalencia semántica, la métrica final se degrada. Los autores emplean modelos del estado del arte como DeBERTa fine-tuneados en datasets de Natural Language Inference, pero incluso estos tienen limitaciones en dominios técnicos especializados.

Otra consideración es que el método detecta **incertidumbre**, no necesariamente **incorrección**. Un modelo puede tener alta entropía semántica porque la pregunta es genuinamente ambigua o porque requiere conocimiento fuera de su distribución de entrenamiento. Distinguir estos casos de verdaderas alucinaciones requiere contexto adicional.

## Hacia LLMs más confiables

Este trabajo representa un avance metodológico importante en la evaluación de confiabilidad de sistemas de IA generativa. Al desplazar el foco desde métricas léxicas hacia métricas semánticas, los autores abordan el problema en el nivel apropiado de abstracción: el significado, no las palabras.

La investigación futura podría explorar la integración de entropía semántica directamente en el proceso de decodificación. En lugar de calcular la métrica post-hoc, el modelo podría ajustar dinámicamente su sampling strategy cuando detecta alta incertidumbre semántica interna. Técnicas de **uncertainty-aware decoding** podrían evitar que el modelo se adentre en regiones del espacio latente asociadas con confabulaciones.

También surge la pregunta de si la arquitectura misma de los LLMs podría modificarse para representar explícitamente incertidumbre semántica durante el entrenamiento. Enfoques como Bayesian deep learning o evidential neural networks ofrecen marcos teóricos para esto, aunque su aplicación a modelos de escala billón de parámetros permanece como desafío abierto.

## Reflexión final

Las alucinaciones en LLMs no desaparecerán pronto. Son consecuencias inherentes de cómo estos modelos aprenden: mediante compresión estadística de enormes colecciones de texto, sin verdadera comprensión ni acceso a bases de conocimiento verificadas. Lo que sí podemos hacer es desarrollar métodos cada vez más sofisticados para detectar cuándo un modelo está operando cerca de los límites de su conocimiento.

El concepto de entropía semántica nos recuerda que la confianza debe medirse en el espacio de significados, no de palabras. Cuando un LLM nos dice algo con aparente certeza, la pregunta relevante no es "¿qué tan probable es esta secuencia de tokens?" sino "¿cuánto acuerdo semántico hay entre las posibles respuestas que el modelo podría generar?".

A medida que integramos estos sistemas en infraestructuras críticas, la capacidad de cuantificar su incertidumbre se vuelve no solo deseable sino esencial. El trabajo de Farquhar y colaboradores nos acerca a ese objetivo.

---

## Referencias

Farquhar, S., Kossen, J., Kuhn, L., & Gal, Y. (2024). Detecting hallucinations in large language models using semantic entropy. *Nature*, 630, 551–557. [https://doi.org/10.1038/s41586-024-07421-0](https://doi.org/10.1038/s41586-024-07421-0)

---

*¿Tienes comentarios sobre este análisis? ¿Te gustaría que profundice en algún aspecto técnico específico? Déjame tus preguntas abajo.*
