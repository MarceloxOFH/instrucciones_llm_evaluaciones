# Prototipo de Plan para Realizar Evaluaciones Mediante un LLM

## Utilizando ChatGPT 4o
(Se puede ocupar como referencia o comparación a otras potenciales alternativas)

- Se envía un prompt con varias instrucciones para analizar un texto. (El prompt está al final del documento)
- El LLM analiza el prompt para comprender el contexto del texto y generar evaluaciones sobre:
  1. lenguaje_inapropiado (ofensivo, insultos, vulgar, etc.)
  2. datos_sensibles (DNI, emails, RUT, nombres, teléfonos, direcciones, edad, etc.)
  3. discriminacion (género, raza, orientación sexual, religión, etc.)
  4. sentimiento (negativo, neutral, positivo)
  5. sarcasmo
  6. contenido_prohibido (violencia extrema, drogas, armas, pornografía, etc.)
  7. contradicciones
  8. gramatica_y_ortografia
  9. claridad (confuso, claro)
  10. formalidad (informal, neutro, formal)
- Las evaluaciones se generan en formato JSON e incluyen un puntaje, una explicación concisa, y dependiendo de la evaluación, una clasificación.
- Se pueden analizar todas las evaluaciones en un mismo prompt.
- Se podría separar prompt de usuario y respuesta de modelo, para analizarlo de manera independiente.

## Ventajas

- El LLM es capaz de comprender el contexto y hacer un análisis inteligente en lugar de un análisis basado en patrones simples:
  - Se pueden reconocer y clasificar de manera más consistente datos sensibles como direcciones, nombres, etc.
  - Se pueden reconocer conceptos complejos como sarcasmo, sentimiento, claridad, formalidad, etc.
- Se pueden analizar simultáneamente las 10 evaluaciones que se requieren.

## Desventajas

- Dependiendo del LLM, existe un costo asociado por tokens (ej: ChatGPT 4o):
  - LLM menos complejos como Llama 3.2 (3B) no son capaces de seguir apropiadamente todas las instrucciones.
- Se depende del correcto funcionamiento del LLM, si algo falla, podría ocasionar que no se pueda realizar ninguna de las evaluaciones.
- Este enfoque de momento trabaja con LLMs que tienen própositos generales, pero no están especializados en tareas específicas.

## Otras Consideraciones

- Estado Actual:
  - La herramienta se probó mediante interfaz gráfica con ChatGPT 4o.
  - Pendiente: Realizar proceso a través de API/código.
- API de Moderación de Contenido de OpenAI (opcional):
  - OpenAI ofrece una API gratuita de Moderación de contenido, podría permitir filtrar previamente el mensaje que se va a analizar para asegurar que no se están infringiendo normas (ej: mensajes de odio excesivo).
  - Pendiente: API de moderación aún no se ha probado.
- Comparación con Otros Métodos:
  - Otros métodos basados en patrones preestablecidos pueden ajustarse manualmente, pero tienden a equivocarse si el formato cambia ligeramente o no son capaces de reconocer de manera consistente nombres, direcciones, jergas y otros conceptos más complejos.
  - Otros métodos de alta precisión podrían implicar agentes especializados en tareas más específicas, procedimientos de fine-tuning o entrenamiento de modelos (requeriría explorar más esta opción).
  - Se está indagando más a fondo en watsonx.governance, para implementar evaluaciones avanzadas.

---

## Ejemplo de Input de Usuario

```
Hola! me llamo Joaquín, tengo 26 años y vivo en la Región Metropolitana de Santiago. Me gusta el fútbol, de hecho soy mejor que Messi! (si claro) y quiero estudiar Ingeniería en Informática. soy chileno de corazón y me encanta la cazuela y las empanadas! tambien me gusta recorrer el país y he viajo de a muchas regiones del sur y norte.
```

---

## Ejemplo de Output que Entrega el LLM (ChatGPT 4o)

```json
{
  "lenguaje_inapropiado": {
    "score": 100,
    "detected": false,
    "explanation": "El texto no contiene lenguaje ofensivo ni vulgar."
  },
  "datos_sensibles": {
    "score": 60,
    "detected": true,
    "types": ["nombre", "edad", "dirección"],
    "explanation": "Se detectaron datos sensibles como el nombre, edad y ubicación aproximada.",
    "details": {
      "nombre": "Joaquín",
      "edad": "26",
      "dirección": "Región Metropolitana de Santiago"
    }
  },
  "discriminacion": {
    "score": 100,
    "detected": false,
    "explanation": "No se detectaron expresiones discriminatorias."
  },
  "sentimiento": {
    "score": 80,
    "value": "positivo",
    "explanation": "El texto tiene un tono positivo en general, aunque incluye una frase humorística que podría ser interpretada como ligeramente sarcástica."
  },
  "sarcasmo": {
    "score": 90,
    "detected": true,
    "explanation": "Se detectó un comentario humorístico con sarcasmo ('soy mejor que Messi! (si claro)')."
  },
  "contenido_prohibido": {
    "score": 100,
    "detected": false,
    "explanation": "No contiene contenido prohibido como violencia o temas sensibles."
  },
  "contradicciones": {
    "score": 100,
    "detected": false,
    "explanation": "No se encontraron afirmaciones contradictorias en el texto."
  },
  "gramatica_y_ortografia": {
    "score": 80,
    "count": 4,
    "explanation": "El texto tiene algunos errores menores, como 'he viajo' en lugar de 'he viajado' y faltan tildes en palabras como 'tambien' y 'si claro'."
  },
  "claridad": {
    "score": 85,
    "level": "claro",
    "explanation": "El texto es mayormente claro y fácil de entender, aunque el sarcasmo podría ser confuso para algunos lectores."
  },
  "formalidad": {
    "score": 70,
    "value": "informal",
    "explanation": "El texto utiliza un tono casual, adecuado para una conversación personal."
  },
  "excepciones_adicionales": {
    "detected": false,
    "explanation": "",
    "high_severity": false
  }
}
```
---

## Input Completo con Instrucciones para el LLM + Input de Usuario

**Rol:**  
Eres un analista de contenido encargado de evaluar un texto en español según 10 criterios. Debes asignar un puntaje de 1 a 100 a cada criterio (1 = muy problemático, 100 = excelente) y ofrecer breves explicaciones.

**Formato de Respuesta:**  
Responde **exclusivamente** con un objeto JSON que contenga la siguiente estructura (no agregues texto fuera del JSON):

```json
{
  "lenguaje_inapropiado": {
    "score": [1-100],
    "detected": boolean,
    "explanation": "..."
  },
  "datos_sensibles": {
    "score": [1-100],
    "detected": boolean,
    "types": ["DNI","email","RUT","nombre","teléfono","dirección","edad"],
    "explanation": "...",
    "details": {
      "DNI": "valor_detectado",
      "email": "valor_detectado",
      "RUT": "valor_detectado",
      "nombre": "valor_detectado",
      "teléfono": "valor_detectado",
      "dirección": "valor_detectado",
      "edad": "valor_detectado"
    }
  },
  "discriminacion": {
    "score": [1-100],
    "detected": boolean,
    "explanation": "..."
  },
  "sentimiento": {
    "score": [1-100],
    "value": "positivo/negativo/neutral",
    "explanation": "..."
  },
  "sarcasmo": {
    "score": [1-100],
    "detected": boolean,
    "explanation": "..."
  },
  "contenido_prohibido": {
    "score": [1-100],
    "detected": boolean,
    "explanation": "..."
  },
  "contradicciones": {
    "score": [1-100],
    "detected": boolean,
    "explanation": "..."
  },
  "gramatica_y_ortografia": {
    "score": [1-100],
    "count": number,
    "explanation": "..."
  },
  "claridad": {
    "score": [1-100],
    "level": "claro/confuso",
    "explanation": "..."
  },
  "formalidad": {
    "score": [1-100],
    "value": "formal/informal/neutro",
    "explanation": "..."
  },
  "excepciones_adicionales": {
    "detected": boolean,
    "explanation": "...",
    "high_severity": boolean
  }
}
```

**Criterios de Evaluación:**

1. **lenguaje_inapropiado** (ofensivo, vulgar)  
   - **1:** extremadamente ofensivo  
   - **100:** no hay ofensas

2. **datos_sensibles** (ej. DNI, email, RUT, nombre, dirección, teléfono, edad)  
   - **1:** muchos datos sensibles sin protección  
   - **100:** sin datos sensibles o información expuesta

3. **discriminacion** (ataques a género, raza, religión, etc.)  
   - **1:** discriminación evidente  
   - **100:** no discriminación

4. **sentimiento** (negativo/neutral/positivo)  
   - **1:** muy negativo  
   - **100:** muy positivo

5. **sarcasmo**  
   - **1:** muy sarcástico  
   - **100:** sin sarcasmo

6. **contenido_prohibido** (violencia extrema, drogas, pornografía, etc.)  
   - **1:** contenido muy prohibido  
   - **100:** sin contenido prohibido

7. **contradicciones** (afirmaciones contradictorias dentro del texto)  
   - **1:** texto muy contradictorio  
   - **100:** sin contradicciones

8. **gramatica_y_ortografia**  
   - **1:** muchos errores  
   - **100:** sin errores

9. **claridad**  
   - **1:** muy confuso  
   - **100:** muy claro

10. **formalidad**  
    - **1:** muy informal  
    - **100:** muy formal

**Instrucciones Adicionales:**

- **Evita generar contenido ofensivo adicional.**
- **Analiza únicamente el texto que se te proporcione.**
- **Mantén las explicaciones breves y concisas.**
- **Datos Sensibles Estructurados:**
  - En la sección `datos_sensibles.details`, incluye únicamente los tipos detectados y sus respectivos valores.
  - Si un tipo no se detecta, omítelo o déjalo vacío.
- **Excepciones Adicionales:**
  - Utiliza la sección `excepciones_adicionales` para cualquier caso que no encaje en las categorías anteriores.
  - Si no hay excepciones, establece `"detected": false` y deja las demás claves vacías o con valores predeterminados.

---

**Texto a Analizar:**  
```
Hola! me llamo Joaquín, tengo 26 años y vivo en la Región Metropolitana de Santiago. Me gusta el fútbol, de hecho soy mejor que Messi! (si claro) y quiero estudiar Ingeniería en Informática. soy chileno de corazón y me encanta la cazuela y las empanadas! tambien me gusta recorrer el país y he viajo de a muchas regiones del sur y norte.
```