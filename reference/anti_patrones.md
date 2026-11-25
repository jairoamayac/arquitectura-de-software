# Anti-patrones de Diseño de Software

> Catálogo de "malos diseños con nombre" — el complemento natural a GRASP/GoF: si los patrones
> son soluciones recurrentes a problemas, los anti-patrones son **errores recurrentes** con
> síntomas reconocibles y una ruta de salida conocida. Útiles para revisión de código y
> diagnóstico rápido de "por qué este módulo es doloroso de tocar".

Cada entrada sigue el mismo formato: **síntoma observable → por qué pasa → cómo se corrige (qué principio/patrón aplicar)**.

---

## Anti-patrones de clases y responsabilidades

### God Object / God Class
**Síntoma**: una clase (`Manager`, `Service`, `Util`, `Helper`) que concentra decenas de métodos no relacionados y conoce/manipula el estado de medio sistema.
**Por qué pasa**: nuevas responsabilidades se agregan a la clase más conveniente en el momento, no a la más correcta.
**Corrección**: aplicar **SRP** y **Alta Cohesión** — dividir por responsabilidad real; usar **Experto** para reasignar cada método a la clase que realmente tiene la información.

### Anemic Domain Model
**Síntoma**: clases de dominio que son solo bolsas de getters/setters (datos puros), mientras toda la lógica de negocio vive en clases "Service" externas.
**Por qué pasa**: diseño guiado por la base de datos (tablas → clases) en vez de por comportamiento.
**Corrección**: mover comportamiento a la entidad que tiene la información (patrón **Experto**); el dominio debe encapsular sus propias reglas, no solo exponer su estado.

### Feature Envy
**Síntoma**: un método de la clase A llama repetidamente a getters de la clase B para hacer cálculos que deberían vivir en B.
**Por qué pasa**: la responsabilidad se asignó a la clase equivocada.
**Corrección**: mover el método a la clase B (de nuevo, **Experto** + Ley de Demeter).

### Shotgun Surgery
**Síntoma**: un cambio de requisito pequeño obliga a tocar muchas clases distintas y dispersas.
**Por qué pasa**: una misma responsabilidad está repetida en múltiples lugares (viola **Once and Only Once**) o el acoplamiento es alto.
**Corrección**: consolidar la responsabilidad en un solo punto; revisar **Alta Cohesión** y **Bajo Acoplamiento**.

---

## Anti-patrones estructurales

### Big Ball of Mud
**Síntoma**: sistema sin una arquitectura discernible — todo depende de todo, sin capas ni módulos claros.
**Por qué pasa**: crecimiento orgánico sin revisión arquitectónica, presión de tiempo constante sin refactorización.
**Corrección**: introducir capas/paquetes (ver sección de Paquetes y Arquitectura en Capas), aplicar **Strangler Fig** para migrar incrementalmente sin "big bang rewrite".

### Spaghetti Code
**Síntoma**: flujo de control enredado, saltos y dependencias cruzadas difíciles de seguir; ausencia de estructura modular.
**Corrección**: extraer funciones con responsabilidad única, aplicar **Template Method** o **Strategy** donde haya lógica condicional repetida por "tipo".

### Golden Hammer
**Síntoma**: usar la misma herramienta/patrón/tecnología para todos los problemas, sin evaluar si aplica ("si todo lo que tienes es un martillo...").
**Ejemplo típico**: forzar microservicios o un patrón GoF donde un diseño simple bastaría.
**Corrección**: volver al principio rector — los patrones están orientados al problema, no al revés; diagnosticar antes de prescribir.

### Lava Flow
**Síntoma**: código muerto o de versiones anteriores que nadie se atreve a borrar "por si acaso algo depende de él".
**Corrección**: cobertura de pruebas + análisis estático de uso antes de eliminar; eliminar con confianza respaldada por tests, no dejar acumular.

### Vendor Lock-In por diseño
**Síntoma**: lógica de negocio mezclada directamente con llamadas a un SDK/proveedor específico, sin capa de abstracción.
**Corrección**: **Diseñar hacia interfaces, no implementaciones** — encapsular el proveedor detrás de una interfaz propia (similar a **Adapter** o **Facade**), de forma que cambiar de proveedor implique cambiar una implementación, no todo el sistema.

---

## Anti-patrones de acoplamiento y dependencias

### Acoplamiento circular (Circular Dependency)
**Síntoma**: el módulo A depende de B y B depende de A (directa o indirectamente), impidiendo compilar/desplegar uno sin el otro.
**Corrección**: introducir **Indirección** (interfaz o evento intermedio) para romper el ciclo; identificar cuál dirección de dependencia es la "correcta" según quién es más estable.

### Acoplamiento temporal oculto
**Síntoma**: un método solo funciona si se llama después de otro método específico, sin que el contrato lo exprese (ej. `init()` antes de `process()` sin validación).
**Corrección**: hacer el orden explícito en el diseño (parámetros del constructor, **Builder**, o validación defensiva que falle rápido si el estado previo no se cumplió).

### Inappropriate Intimacy
**Síntoma**: dos clases acceden libremente a los detalles internos (privados/protegidos) una de la otra, como si fueran una sola.
**Corrección**: Ley de Demeter — exponer solo lo necesario a través de una interfaz pública mínima; revisar **Segregación de Interfaces**.

---

## Anti-patrones de creación de objetos

### Constructor con demasiados parámetros (Telescoping Constructor)
**Síntoma**: constructores con 6+ parámetros, varios opcionales, generando sobrecargas confusas.
**Corrección**: patrón **Builder**.

### New por todas partes (falta de Factory)
**Síntoma**: la lógica de decisión de qué clase concreta instanciar está duplicada en docenas de lugares del código (`if (tipo == "X") new ClaseX(); else if...`).
**Corrección**: **Factory Method** o **Abstract Factory** — centralizar la decisión de creación en un solo punto.

---

## Anti-patrones de sistemas distribuidos

### Distributed Monolith
**Síntoma**: el sistema está separado en "microservicios" pero todos deben desplegarse juntos porque comparten esquema de BD o tienen llamadas síncronas en cadena que los acoplan como si fueran uno solo.
**Por qué pasa**: se aplicó la separación física (servicios) sin lograr la separación real de responsabilidades y datos.
**Corrección**: revisar **Database per Service**, introducir comunicación asíncrona (eventos) donde haya cadenas síncronas largas, y verificar que cada servicio pueda desplegarse de forma independiente.

### Chatty Services
**Síntoma**: completar una operación de negocio requiere decenas de llamadas síncronas entre servicios, generando latencia acumulada y fragilidad (si uno falla, falla todo).
**Corrección**: **BFF** o composición en el gateway; considerar **Event-Carried State Transfer** para evitar llamadas adicionales; revisar si los límites de servicio están bien trazados (quizá deberían fusionarse).

### Dual Write sin Outbox
**Síntoma**: el código escribe en la base de datos y luego publica un evento en dos pasos separados sin garantía atómica — si el segundo paso falla, el sistema queda inconsistente sin que nadie lo note.
**Corrección**: **Outbox Pattern**.

---

## Cómo usar este catálogo en revisión de código

1. Identifica el **síntoma observable** primero (no el nombre del anti-patrón) — pregúntate: ¿qué es doloroso aquí? ¿qué se rompe inesperadamente al cambiar algo?
2. Conecta el síntoma con la causa de la lista.
3. Aplica la corrección sugerida, citando el principio o patrón concreto (no "refactorizar" en abstracto).
4. Verifica que la corrección no introduzca un nuevo anti-patrón (ej. abusar de Factory en exceso puede llevar a un **Golden Hammer** de patrones de creación donde un constructor simple bastaba).
