---
title: "Creando Skills Personalizadas para Claude: De Código Inconsistente a Flujo de Trabajo Optimizado"
date: 2026-11-03
# authors:
#   - Vladímir Rodríguez
categories:
  - AI
  - Django
  - Desarrollo
tags:
  - Claude
  - AI Tools
  - Django
  - Skills
  - Productivity
---

# Creando Skills Personalizadas para Claude: De Código Inconsistente a Flujo de Trabajo Optimizado

Si has trabajado con LLMs como Claude para generar código, probablemente has experimentado esta frustración: escribes las mismas instrucciones una y otra vez.

En mi caso, trabajando con un proyecto Django, cada vez que necesitaba generar código tenía que recordarle a Claude:

- "Usa `pzLogger` para logging, no el logger estándar"
- "Lee las configuraciones de `settings.current_settings`, no valores hardcoded"
- "Todo en inglés: variables, comentarios, docstrings"
<!-- more -->
- "Usa 'consultant', nunca 'representative'"
- "Los endpoints con `@api_view` y `@token_required`"
- "Documenta con `drf_spectacular`, no `drf_yasg`"

Y así, cada. Maldita. Vez.
Las Custom Instructions ayudan, pero tienen limitaciones:

- Son globales (se aplican a TODO)
- Espacio limitado
- No puedes tener ejemplos extensos
- No se activan contextualmente

## La Solución: Claude Skills

Anthropic lanzó recientemente la funcionalidad de **Skills** - básicamente, paquetes de conocimiento especializado que puedes crear y que Claude activa automáticamente según el contexto.

Pensé: "¿Y si creo una skill con todas las convenciones de mi proyecto Django?"

## Primer Intento: La Skill Monolítica

Mi primer approach fue crear una skill con TODO:

```markdown
---
name: django-dev
description: Django development skill for everything...
---

# Django Development Skill

## Core Principles
1. Logger
2. Settings
3. English
4. Consultant terminology
5. API endpoints
6. Swagger docs
... y 20 páginas más
```

**Resultado:** Funcionó... más o menos.

El problema era que la skill se activaba para TODO - incluso cuando solo quería crear una función auxiliar simple. Me daba toda la documentación de Swagger, patrones de API, etc., cuando solo necesitaba las convenciones básicas.

## La Revelación: Una Skill No Es Un Monolito

Mientras iteraba sobre la skill, me di cuenta de algo:

> **No todo el código Django es igual. Los endpoints de API tienen necesidades muy diferentes a los modelos o servicios.**

Entonces, ¿por qué tener una sola skill gigante?

## La Refactorización: Dos Skills Especializadas

Dividí mi skill en dos:

### 1. `django-core` - Las Bases

Para código Django general:

```markdown
---
name: django-core
description: Core Django conventions for models, services, utilities, 
and general Python code. Use when generating Django code that is NOT 
an API endpoint...
---
```

**Se activa cuando creo:**
- Modelos
- Servicios
- Utilidades
- Signals
- Management commands
- Tests

**Lo que aplica:**
- Logger obligatorio
- Sin hardcoding
- Nomenclatura en inglés
- Terminología consistente

### 2. `django-api` - Para Endpoints

Específica para REST APIs:

```markdown
---
name: django-api
description: Django REST Framework API endpoint development... 
Use ONLY when creating or modifying API endpoints...
---
```

**Se activa cuando creo:**
- Endpoints REST
- Documentación de API
- Autenticación

**Lo que añade encima de django-core:**
- Patrones de `@api_view`
- Decorador `@token_required`
- Documentación `drf_spectacular`
- Serializers de request parameters
- OpenApiExample

## El Momento "Ajá"

La magia está en la **descripción** del frontmatter:

```yaml
description: Use ONLY when creating or modifying API endpoints, 
views, or REST API functionality...
```

Esta línea es la que Claude lee para decidir si activar la skill o no. Es como el "trigger" de la skill.

## Anatomía de una Skill Bien Diseñada

Después de muchas iteraciones, aprendí que una buena skill tiene:

### 1. Description Claro y Específico

```yaml
description: Django REST Framework API endpoint development with 
drf_spectacular. Use ONLY when creating or modifying API endpoints. 
For general Django code (models, services), use django-core instead.
```

**Mal:**
```yaml
description: Django development
```

**Bien:**
```yaml
description: Use when the user asks to create Django API endpoints 
including @api_view, @token_required, and @extend_schema documentation
```

### 2. Progressive Disclosure

No pongas TODO en `SKILL.md`. Usa referencias:

```
django-api-skill/
├── SKILL.md                    # Solo lo esencial
├── references/
│   └── drf-spectacular-guide.md  # Detalles profundos
```

Claude solo carga las referencias cuando las necesita.

### 3. Ejemplos Concretos

Menos "explica cómo hacer X", más "aquí está el código exacto para X":

**Menos útil:**
```markdown
Use @api_view decorator for API endpoints with proper authentication.
```

**Más útil:**
```python
@api_view(["POST"])
@token_required
def my_endpoint(request: Request) -> Response:
    """Endpoint description."""
    params = MyParamsSerializer(data=request.data)
    if not params.is_valid():
        pzLogger.error(f"Errors: {params.errors}")
        return Response(params.errors, status=400)
    # ...
```

### 4. Checklist Verificable

Al final de cada skill:

```markdown
## Code Generation Checklist

Before generating API endpoint code, ensure:

- [ ] Using @api_view decorator
- [ ] Using @token_required 
- [ ] Using @extend_schema
- [ ] Request parameters validated with serializer
- [ ] All response codes documented
- [ ] Logger used for operations
```

## Resultados: Antes vs Después

### Antes (Sin Skills)

**Mi prompt:**
```
Crea un endpoint para calcular comisiones. Recibe consultant_id 
y period. Usa pzLogger para logging. Lee configuración de settings. 
Todo en inglés. Documenta con drf_spectacular. Usa @token_required.
Valida con serializer...
```

→ 5 minutos escribiendo el prompt  
→ Claude olvida algo  
→ Hay que corregir  
→ Frustración

### Después (Con Skills)

**Mi prompt:**
```
Crea un endpoint para calcular comisiones. Recibe consultant_id 
y period.
```

→ 30 segundos  
→ Claude activa `django-api` automáticamente  
→ Código perfecto con todas las convenciones  
→ ✨ Magia ✨

## Lecciones Aprendidas

### 1. El Description Es Crítico

Pasé 80% del tiempo refinando el `description`. Es lo que hace que Claude active tu skill en el momento correcto.

### 2. Menos Es Más

Mi primer SKILL.md tenía 1000 líneas. El actual tiene 300. El resto está en referencias que Claude carga solo cuando las necesita.

### 3. Skills Pequeñas y Enfocadas > Skills Gigantes

Dos skills de 300 líneas cada una > Una skill de 600 líneas.

### 4. Itera Con Casos Reales

Crea la skill, úsala en tu trabajo real, observa dónde falla, ajusta. Repite.

No intentes hacer la skill perfecta desde el inicio.

### 5. La Skill No Reemplaza Tu Conocimiento

La skill NO enseña Django a Claude. Claude ya sabe Django.

La skill le dice: "En ESTE proyecto, así es como hacemos las cosas".

## Casos de Uso Inesperados

Descubrí que las skills son útiles para:

### Testing Automation

Agregué patrones de testing a `django-core`:

```
"Crea tests para el servicio ConsultantService"
```

Claude genera tests completos con:
- Fixtures
- Mocking
- Assertions
- Logging verification

### Documentation

```
"Documenta este endpoint con el patrón estándar"
```

Claude añade `@extend_schema` completo con ejemplos.

### Code Review

Cuando Claude revisa mi código, usa la skill para detectar:
- Falta de logging
- Valores hardcoded
- Nomenclatura incorrecta

## El Flujo de Trabajo Ideal

Así es como trabajo ahora:

1. **Modelo (django-core):**
   ```
   "Crea el modelo Commission"
   ```

2. **Servicio (django-core):**
   ```
   "Crea servicio para calcular comisiones"
   ```

3. **Endpoint (django-api):**
   ```
   "Expone el servicio como endpoint POST"
   ```

Claude sabe automáticamente qué skill usar en cada paso.

## Implementación Técnica: Tips

### Estructura de Archivos

```
my-skill/
├── SKILL.md                 # Required, ~300 líneas
├── references/              # Optional, detalles
│   ├── advanced-patterns.md
│   └── api-guide.md
└── scripts/                 # Optional, helpers
    └── generate_boilerplate.py
```

### El Frontmatter Perfecto

```yaml
---
name: nombre-corto-sin-espacios
description: |
  Descripción específica que menciona:
  - Cuándo usar esta skill
  - Qué tipo de código genera
  - Cuándo NO usarla (importante!)
  - Keywords que debe buscar
---
```

### Referencias Inteligentes

En SKILL.md:

```markdown
For comprehensive patterns, see:
- [references/api-guide.md](references/api-guide.md)
```

Claude solo las lee si las necesita.

### Testing Your Skills

Crea un documento de test:

```markdown
# Test Cases

## Should Trigger
- "Create an API endpoint"
- "Add authentication to view"

## Should NOT Trigger  
- "Create a model"
- "Write a utility function"
```

Prueba cada caso.

## Errores Comunes a Evitar

### ❌ Description Demasiado Genérico

```yaml
description: Django development skill
```

Claude no sabrá cuándo activarla.

### ❌ Poner TODO en SKILL.md

Tu skill será un monstruo de 2000 líneas que contamina el context window.

### ❌ No Especificar Cuándo NO Usar

```yaml
description: Use for API endpoints. 
NOT for models, services, or utilities.
```

Esto es tan importante como decir cuándo SÍ usar.

### ❌ Olvidar Ejemplos Concretos

"Use proper error handling" → ❌  
"return Response({'error': 'Not found'}, status=404)" → ✅

### ❌ No Iterar

Tu primera skill será mala. Y está bien. Úsala, obsérvala, mejórala.

## Métricas: ¿Funcionó?

Después de un mes usando estas skills:

- **Tiempo de setup por prompt:** 5 min → 30 seg (90% reducción)
- **Código que requiere corrección:** ~40% → ~5%
- **Consistencia del código:** Subjetivo pero MUCHO mejor
- **Frustración del desarrollador:** Alta → Baja
- **Satisfacción:** 📈📈📈

## El Futuro: Skills Como Documentación Viva

Me di cuenta de algo poderoso: **Las skills son documentación ejecutable**.

En lugar de tener un documento "Django Conventions" que nadie lee, tengo skills que FUERZAN las convenciones automáticamente.

Es como tener un senior developer mirando sobre tu hombro, pero que nunca se cansa y siempre es consistente.

## Conclusión: Skills Son el Futuro del AI-Assisted Development

Si trabajas con LLMs para generar código, especialmente en proyectos con convenciones específicas, las skills son un game-changer.

No es solo sobre "hacer que el AI funcione mejor". Es sobre:

- **Menos fricción** en tu flujo de trabajo
- **Más consistencia** en tu codebase
- **Menos tiempo** explicando, más tiempo construyendo
- **Documentación** que realmente se usa (porque está integrada en el proceso)

## Para Empezar

Si quieres crear tus propias skills:

1. **Identifica las convenciones** que repites constantemente
2. **Empieza pequeño** - una skill simple de 200 líneas
3. **Enfócate en el description** - es el 80% del trabajo
4. **Prueba con casos reales** inmediatamente
5. **Itera** basándote en lo que funciona y lo que no

Las skills de Claude son una de esas features que parece simple en la superficie pero que, cuando las dominas, transforman completamente cómo trabajas con AI.

## Recursos

- [Documentación oficial de Claude Skills](https://docs.anthropic.com)
- Mis skills en GitHub: [link] (si decides compartirlas)

---

**¿Has creado skills personalizadas?** Me encantaría saber qué casos de uso has descubierto. Déjame un comentario o contáctame en [tu contacto].

---

*Escrito con la ayuda de Claude (usando sus propias skills, por supuesto 😉)*
