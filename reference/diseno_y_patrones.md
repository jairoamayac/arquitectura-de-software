# Fundamentos de Ingeniería de Software — Diseño y Patrones

> Pontificia Universidad Javeriana - Ingeniería de Sistemas
> Ing. Orlando Vasquez
> Consolidado a partir de: *"Ingeniería de Software 8 — Diseño"* y *"Ingeniería de Software 14 — Patrones"*

---

## Tabla de contenido

1. [Diseño de Software](#parte-1-diseño-de-software)
   - [1.1 ¿Qué es el diseño?](#11-qué-es-el-diseño)
   - [1.2 Artefactos del diseño](#12-artefactos-del-diseño)
   - [1.3 Diagrama de clases del sistema](#13-diagrama-de-clases-del-sistema)
   - [1.4 Tipos de relaciones UML](#14-tipos-de-relaciones-uml)
   - [1.5 Diseño de base de datos](#15-diseño-de-base-de-datos)
   - [1.6 Paquetes](#16-paquetes)
   - [1.7 Arquitectura de software](#17-arquitectura-de-software)
   - [1.8 Interfaces](#18-interfaces)
   - [1.9 Componentes](#19-componentes)
   - [1.10 Modelo de despliegue](#110-modelo-de-despliegue)
   - [1.11 Principios de diseño](#111-principios-de-diseño)
   - [1.12 Modelo C4](#112-modelo-c4)
2. [Patrones de Diseño](#parte-2-patrones-de-diseño)
   - [2.1 Introducción a los patrones](#21-introducción-a-los-patrones)
   - [2.2 Patrones GRASP](#22-patrones-grasp)
   - [2.3 Patrones GoF](#23-patrones-gof)
   - [2.4 Patrones de diseño general](#24-patrones-de-diseño-general)
   - [2.5 Elementos esenciales de un patrón de diseño](#25-elementos-esenciales-de-un-patrón-de-diseño)
3. [Bibliografía](#bibliografía)

---

# PARTE 1: DISEÑO DE SOFTWARE

## 1.1 ¿Qué es el diseño?

El diseño es el lugar en el que las reglas de la creatividad —los requisitos de los participantes, las necesidades del negocio y las consideraciones técnicas— se unen para formular un producto o sistema.

- El diseño crea una **representación o modelo del software**, diferente del modelo de requisitos o análisis.
- El modelo de diseño proporciona detalles sobre **arquitectura del software, estructuras de datos, interfaces y componentes** necesarios para implementar el sistema.

### ¿Quién lo hace?

Ingenieros de software llevan a cabo cada una de las tareas del diseño:

- **Arquitecto**: Software / Hardware
- **Diseñador**: Sistema / Interfaces / Base de datos

### ¿Por qué es importante?

- El diseño permite **modelar** el sistema o producto que se va a construir.
- Este modelo se evalúa respecto de la calidad y se mejora **antes de escribir (o generar) código**.
- Después, se efectúan pruebas y se involucra a los usuarios finales.
- **El diseño es el lugar en el que se establece la calidad del software** — y donde se introducen muchos de los errores si no se hace bien.

### ¿Cuáles son los pasos?

El diseño representa al software de varias maneras:

1. Definir la **arquitectura** del sistema o producto.
2. Modelar las **interfaces** que conectan al software con los usuarios finales, otros sistemas/dispositivos, y sus propios componentes.
3. Diseñar los **componentes** del software que se utilizan para construir el sistema.

Cada perspectiva es una acción de diseño distinta, pero todas se apegan a un conjunto básico de conceptos de diseño.

### ¿Cuál es el producto final?

El trabajo principal del diseño de software es un **modelo de diseño** que agrupa las representaciones arquitectónicas, de interfaces, a nivel de componente y de despliegue.

---

## 1.2 Artefactos del diseño

Cinco artefactos a diseñar:

| # | Artefacto | Detalle |
|---|---|---|
| 01 | **Datos y/o clases** | Diagramas de clases; modelo lógico y físico de la base de datos |
| 02 | **Arquitectura del sistema** | El plano del software; visión general; estilos y patrones arquitectónicos |
| 03 | **Componentes** | Describe completamente los detalles internos de cada componente; usa diagramas de componentes |
| 04 | **Interfaces** | Permiten que la información fluya hacia dentro/afuera del sistema: IU, interfaces externas, interfaces internas |
| 05 | **Despliegue** | Indica cómo se acomodan la funcionalidad y los subsistemas en el ambiente físico; diagramas de despliegue |

---

## 1.3 Diagrama de clases del sistema

Elementos: relaciones, multiplicidad, navegabilidad, roles, nombre de relación.

### Tipos de objetos de análisis

- **Objetos de Entidad**: información persistente rastreada por el sistema (objetos de dominio / "objetos de negocio").
- **Objetos de Interfaz** (límites del sistema, *Boundary*): representan la interacción entre el usuario y el sistema.
- **Objetos de Control**: representan las tareas de control realizadas por el sistema.

> Estereotipo UML: `<<Entity>>`, `<<Control>>`, `<<Boundary>>`.

Las **entidades** representan datos del sistema (Cliente, Transacción, Carrito). Los **límites** interactúan con los actores (interfaces de usuario, gateways, proxies). Los **controladores** median entre límites y entidades, orquestando la ejecución de comandos provenientes del límite.

### Tipos de clases de análisis

| Tipo | Descripción |
|---|---|
| **Boundary** | Representa el límite entre el sistema y el ambiente (pantalla GUI, sistema externo, dispositivo) |
| **Control** | Representa la lógica de control y coordinación del sistema |
| **Entity** | Representa los elementos de información del sistema |

→ Relación directa con el **patrón Modelo-Vista-Controlador (MVC)**.

### Interacciones típicas entre clases de análisis

- *Boundary* y *Entity* **no interactúan** entre sí.
- *Control* puede interactuar con otros *Control*.
- *Boundary* **no debería interactuar** entre ellos.
- *Entity* interactúa con otros *Entity*.

### Identificando objetos Entidad

Heurística:
- Términos que desarrolladores/usuarios necesitan aclarar para comprender el caso de uso.
- Nombres recurrentes en los casos de uso.
- Entidades del mundo real que el sistema necesita tener en cuenta.
- Actividades del mundo real que el sistema necesita tener en cuenta.

### Identificando objetos Interfaz (Boundary)

- Representan la interfaz del sistema con los actores; cada actor interactúa con al menos un objeto de frontera.
- El objeto de límite recopila la información del actor y la traduce a una forma usable por entidades y controles.
- Identificar formularios para entrada de datos y avisos/mensajes de respuesta.
- Usar siempre términos del **usuario final**, no términos de la solución/implementación.

### Identificando objetos Control

- Responsables de coordinar objetos de frontera y entidad.
- Usualmente no tienen contraparte concreta en el mundo real.
- Estrecha relación entre un caso de uso y un objeto de control.
- Se crea al inicio de un caso de uso y deja de existir al final.

---

## 1.4 Tipos de relaciones UML

### 1. Asociación
Indica cómo una clase se relaciona con otra; línea sólida, binaria, uni/bidireccional.

| Multiplicidad | Significado |
|---|---|
| `1` | Uno y solo uno |
| `0..1` | Cero o uno |
| `0..*` | Cero o más |
| `1..*` | Al menos uno |
| `N..M` | Mínimo N, máximo M |

### 2. Agregaciones
Modela relación "todo o parte":
- **2.1 Agregación normal**: rombo vacío del lado del todo — el "todo" sigue funcionando sin alguna de las partes.
- **2.2 Agregación de composición**: rombo lleno del lado del todo — el "todo" necesita las "partes" para funcionar.

### 3. Dependencia
Una clase "depende" de otra para funcionar, pero la dependiente no es parte estructural de ella. Se representa con línea discontinua y flecha hacia el elemento independiente.

### 4. Generalización
Una clase hereda métodos y atributos definidos por una superclase.

### 5. Especialización
Creación de subclases a partir de una superclase. Se marca con línea continua y flecha vacía del lado de la superclase.

---

## 1.5 Diseño de base de datos

- ¿Relacional? ¿Documental?
- Diagrama ER
- Esquema de documento (Schema XML / JSON)
- Patrones de acceso a datos
- Modelo lógico de la BD relacional
- Modelo de BD No Relacional (documental)

---

## 1.6 Paquetes

Un **paquete de diseño** es una colección de clases, relaciones, diagramas y otros paquetes. Se utiliza para estructurar el modelo de diseño dividiéndolo en partes más pequeñas. *(RUP 2003)*

- Dividen el modelo en partes más pequeñas o módulos → facilitan el manejo de sistemas complejos.
- Sirven para estructurar el diseño, ordenarlo, administrar la complejidad y documentar.
- Similares a los *namespaces* en C++ o a los *packages* en Java.

---

## 1.7 Arquitectura de software

Estilos principales:

- Arquitectura en Capas
- Arquitectura de expertos (Blackboard)
- Arquitectura de Tubería y Filtro (Pipes & Filters)
- Arquitectura Cliente-Servidor
- Arquitectura Orientada a Servicios (SOA) / Microservicios

### Arquitectura en Capas

Se enfoca en distribuir roles y responsabilidades de forma jerárquica, separando responsabilidades.

| Capa | Responsabilidad |
|---|---|
| **Presentación** | Toda la lógica relacionada con la interfaz de usuario |
| **Negocio** | Procesa reglas de negocio: cálculos, validaciones, transformaciones; reglas de acceso a la capa de datos |
| **Datos** | Donde residen los datos |

**Beneficios:**
- Abstracción, encapsulamiento, capas de funcionalidad bien definidas.
- La separación reduce el riesgo e impacto de cambios tecnológicos.
- Permite pruebas por niveles de componentes y capas.

**Niveles vs. capas:** las *capas* se refieren a tareas específicas de los componentes; los *niveles* se refieren a cómo se organizan y agrupan las capas para formar la arquitectura completa.

**Ejemplo (3 capas, 2 niveles, app web):**
- *Primer nivel*: Presentación (UI) + Lógica empresarial.
- *Segundo nivel*: Acceso a datos (interacción con BD, persistencia).

### Arquitectura de Expertos (Blackboard)

Modelo aplicado principalmente a IA, donde múltiples expertos trabajan juntos en un problema complejo. Analogía de un tablero negro (*blackboard*) donde los expertos escriben sus soluciones.

**Elementos:**
- **Conocimiento**: información y habilidades de cada experto (humano o máquina).
- **Control**: coordina la interacción de los expertos; decide quién trabaja, cuándo, y cómo combinar soluciones.
- **Blackboard**: memoria compartida donde los expertos leen y escriben información; ahí se combinan las soluciones.

### Arquitectura de Tubería y Filtro (Pipes & Filters)

Componentes independientes ("filtros") conectados por "tuberías" (*pipes*) que permiten que los datos fluyan a través de ellos. Cada filtro hace una tarea específica; los datos se transforman secuencialmente; el resultado final lo produce el último filtro.

**Ejemplo en Windows:**
```
dir *.txt | find "certi.txt"
netstat -an | find "8181"
```

### Arquitectura Cliente-Servidor

Origen: redes de computadores en los 60 (IBM, *time-sharing* en mainframes); se popularizó en los 80 con LAN y PCs.

El cliente produce una solicitud de información a un servidor; el servidor responde a la demanda.

### Arquitectura Orientada a Servicios (SOA) / Microservicios

**SOA**: enfoque basado en servicios independientes y autónomos que se comunican entre sí. Independientes de plataforma, lenguaje y SO → flexibilidad y escalabilidad. Se basa en estándares: XML, SOAP, REST.

- **SOAP**: Simple Object Access Protocol
- **REST**: Representational State Transfer
- **XML**: Extensible Markup Language
- **JSON**: JavaScript Object Notation

**Microservicios**: diseñados para ser escalables y resilientes; se despliegan en contenedores (Docker, Kubernetes); se comunican vía API REST.

> Para patrones de comunicación, consistencia de datos y resiliencia entre servicios, ver
> `patrones_arquitectura_distribuida.md`.

### Arquitectura Hexagonal (Puertos y Adaptadores) / Clean Architecture

El dominio queda en el centro y **no depende de nada externo**. Se comunica con el mundo
a través de **puertos** (interfaces definidas por el dominio) que implementan
**adaptadores** (HTTP, base de datos, colas, proveedores externos).

```
   Adaptadores de entrada            Núcleo                Adaptadores de salida
 ┌─────────────────────┐   ┌──────────────────────────┐   ┌────────────────────────┐
 │ Controlador REST    │──▶│ Casos de uso (aplicación)│──▶│ Repositorio PostgreSQL │
 │ Consumidor de cola  │   │ Entidades y reglas       │   │ Cliente de proveedor   │
 │ CLI / cron          │   │ Puertos (interfaces)     │   │ Publicador de eventos  │
 └─────────────────────┘   └──────────────────────────┘   └────────────────────────┘
          Las dependencias del código apuntan siempre hacia el núcleo
```

- **Regla de dependencia**: infraestructura → aplicación → dominio, nunca al revés
  (Inversión de Dependencias aplicada a nivel arquitectónico).
- **Beneficios**: el dominio se prueba sin BD ni red; cambiar un proveedor implica
  escribir un adaptador nuevo, no tocar reglas de negocio.
- **Costo**: más interfaces y mapeos entre capas; excesivo para CRUD sin lógica de negocio.
- **Verificación**: test de dependencias en CI que falle si el dominio importa un
  framework o un SDK externo.

### Diseño Guiado por el Dominio (DDD) estratégico

Herramienta para trazar fronteras de módulos o servicios a partir del negocio, no de la
tecnología:

- **Bounded Context**: frontera dentro de la cual un modelo y su lenguaje son
  consistentes. Buen candidato a módulo (monolito modular) o servicio.
- **Lenguaje ubicuo**: los nombres del código son los del negocio dentro de cada contexto.
- **Context Map**: relaciones entre contextos (cliente-proveedor, conformista,
  *Anti-Corruption Layer* para aislarse de un modelo externo o legacy).
- **Agregado**: grupo de entidades que cambia de forma consistente dentro de una
  transacción; las referencias entre agregados son por ID.

> Regla práctica: una transacción modifica un solo agregado. Si un caso de uso necesita
> modificar varios de forma consistente, revisa las fronteras o usa una Saga.

---

## 1.8 Interfaces

### Tipos de interfaces

- Interfaces con otros sistemas (Legacy, Sensores IoT, Sistemas de terceros, Cloud Services, Bases de Datos).
- Interfaces de usuario GUI.

### Los 10 principios de usabilidad web de Jakob Nielsen

1. **Visibilidad del estado del sistema** — barras de progreso.
2. **Relación entre el sistema y el mundo real** — *skeumorphism*, lenguaje claro.
3. **Control y libertad del usuario** — botón de borrar en carrito de compras.
4. **Consistencia y estándares** — menús, iconos estándar.
5. **Prevención de errores** — autocompletar.
6. **Reconocer antes que recordar** — fuentes de texto claras.
7. **Flexibilidad y eficiencia de uso** — cualquiera puede usar el sistema.
8. **Diseño estético y minimalista** — *Material*, KISS (*Keep It Simple Stupid*).
9. **Ayudar a reconocer, diagnosticar y corregir errores** — ej. "404 not found".
10. **Ayuda y documentación** — FAQ, tours.

*(Fuente: semrush.com/blog/usabilidad-web-principios-jakob-nielsen)*

---

## 1.9 Componentes

- Los **Diagramas de Componentes** ilustran las piezas de software que conforman un sistema.
- Nivel de abstracción más alto que un diagrama de clases o de paquetes — un componente suele implementarse con una o más clases/objetos en tiempo de ejecución.
- Representan al sistema desde el punto de vista físico.
- Es recomendable definir las **interfaces del componente** para entender sus responsabilidades.

### Interfaces del sistema relacionadas a componentes

- Interfaces que el componente necesita **consumir** de terceros.
- Interfaces que el componente debe **proveer** como servicios a otras aplicaciones.

- Una **interfaz** contiene una colección de operaciones y especifica los servicios de una clase o componente.
- Se conecta al componente que la implementa mediante una relación de **realización**, y al componente que usa sus servicios mediante una **dependencia**.

---

## 1.10 Modelo de despliegue

- El **Diagrama de Despliegue** (UML) modela la disposición física de los artefactos de software en nodos de hardware.
- **Componentes**: nodos, protocolos (HTTP, HTTPS, TCP/IP, ODBC, FTP, SMTP, LDAP, etc.), propiedades de software/hardware mínimas requeridas.
- Puede unirse el diagrama de despliegue con el de componentes para una vista física + lógica completa.

---

## 1.11 Principios de diseño

### Indicios de un mal diseño

- **RIGIDEZ**: difícil de cambiar porque un cambio en cualquier parte afecta muchas otras partes del sistema (*Monolito*).
- **FRAGILIDAD**: al hacer un cambio, partes inesperadas "se rompen".
- **INMOVILIDAD**: difícil reutilizar una parte del sistema en otra aplicación porque no se puede separar de la actual.
- **VISCOSIDAD**: a la hora de implementar cambios, se "pone un parche" en vez de realizar el cambio respetando la intención del diseño original.

> Los principios de diseño son guías o sugerencias de buenas prácticas que ayudan a evitar un mal diseño del software.

### Buenas prácticas en el diseño

1. **Diseñar hacia las interfaces, no hacia las implementaciones.**
   Separa la interfaz abstracta de su implementación concreta. Las interfaces tienden a permanecer estables; las implementaciones pueden variar mucho (memoria, tiempo de respuesta, etc.).

2. **Principio "No hables con extraños" (Ley de Demeter)** — *Shy programming*.
   Las piezas de código no deben revelar sus particularidades ni interactuar con demasiadas otras piezas (clases, subsistemas, módulos).
   Las operaciones de una clase solo deberían llamar a objetos que cumplan una de estas 4 características:
   1. A sí mismos.
   2. A objetos pasados como argumento.
   3. A objetos creados por ella.
   4. A objetos con los cuales tiene relaciones directas.
   → Ayuda a obtener **bajo acoplamiento**.

3. **Single Responsibility Principle (SRP)**.
   Una clase o módulo debe tener **una única responsabilidad** y solo una razón para cambiar. Se enfoca en la cohesión y la modularidad.

4. **Principio "Open-Closed"**.
   El diseño debe quedar **abierto a extensiones** y **cerrado a modificaciones**. El comportamiento conocido se implementa; el desconocido tiene un punto de extensión, sin modificar el código existente. Fundamental para soluciones reutilizables y mantenibles.

5. **Segregación de Interfaces**.
   Los clientes no deberían depender de interfaces que no emplean. Las interfaces deben ser tan pequeñas como sea posible, manteniendo cohesión. Si una interfaz "gruesa" sirve a clientes con necesidades distintas, conviene dividirla.

6. **Once and Only Once Rule**.
   No repetir una misma responsabilidad en más de una clase o módulo — evita inconsistencias al modificar.

7. **Modularidad, alta cohesión y débil acoplamiento**.
   - **Cohesión**: medida en que un componente se dedica solo a la tarea para la que fue creado, delegando tareas complementarias a otros componentes.
   - **Acoplamiento**: medida en que los cambios de un componente tienden a necesitar cambios en otro componente.

---

## 1.12 Modelo C4

El **modelo C4** (*Context, Containers, Components, Code*) es un enfoque arquitectónico propuesto por **Simon Brown** para diseñar diagramas de arquitectura claros y concisos en distintos niveles de abstracción.

### Niveles

1. **Contexto (Context)**
   - Vista general del sistema y su entorno.
   - Identifica usuarios, sistemas externos e interacciones relevantes.
   - Diagramas de contexto: sistema vs. partes interesadas externas.

2. **Contenedores (Containers)**
   - Identifica los principales contenedores/agrupaciones lógicas de software.
   - Representan aplicaciones, servicios, bases de datos u otras unidades significativas de implementación/ejecución.
   - Diagramas de contenedores: relaciones y comunicación entre contenedores.

3. **Componentes (Components)**
   - Componentes internos de cada contenedor: unidades lógicas más pequeñas (clases, módulos, servicios).
   - Diagramas de componentes: estructura interna y relaciones.

4. **Código (Code)**
   - Nivel más detallado: código fuente real.
   - Diagramas de clases, secuencia, etc.
   - Permite identificar patrones de diseño, dependencias y detalles de implementación.

> El modelo C4 **no especifica notación particular** — se puede usar UML con paquetes, componentes y estereotipos, pizarras o herramientas de software. Cada elemento debería tener nombre, tipo, opción tecnológica (si aplica) y texto descriptivo. Es importante incluir clave/leyenda para mantener coherencia entre niveles.

### Diagrama de Arquitectura de Referencia vs. Diagrama de Arquitectura Real

| Aspecto | Arquitectura de Referencia | Arquitectura Real/Tradicional |
|---|---|---|
| **Generalidad** | General, adaptable, plantilla para un dominio | Específico y detallado para un proyecto particular |
| **Propósito** | Base común, marco estandarizado, orienta desarrollo y consistencia | Detalla cómo se construirá y operará un sistema específico |
| **Flexibilidad** | Flexible, adaptable y extensible | Menos flexible, decisiones tecnológicas específicas fijas |
| **Abstracción** | Nivel alto, conceptos y relaciones generales | Nivel bajo, detalles concretos y específicos |

### Elementos de un diagrama de arquitectura de software

1. **Componentes**: módulos, subsistemas, servicios, clases, etc. (ej. servidor web, servidor de aplicaciones, BD, APIs de terceros).
2. **Relaciones**: cómo interactúan los componentes (llamadas, flujos de datos, mensajería).
3. **Interfaces**: puntos de interacción entre componentes (ej. API REST entre frontend y backend).
4. **Conectores**: elementos visuales de las conexiones (líneas/flechas), muestran flujo de información y control.
5. **Datos de flujo**: qué datos se transfieren, su tipo y dirección.

*(Fuente: "El modelo C4 de documentación para la Arquitectura de Software" — Javier Vivanco, Medium)*

---

# PARTE 2: PATRONES DE DISEÑO

## 2.1 Introducción a los patrones

### El problema común de muchas compañías

Proyectos que dan solución a problemas que proyectos anteriores ya habían solucionado → **choca con buenos diseños** → **provoca el fracaso en el desarrollo del proyecto**.

> Necesidad: solución flexible para la reutilización.

### Patrones de Arquitectura vs. Patrones de Diseño

| Patrones de Arquitectura | Patrones de Diseño |
|---|---|
| Afectan la estructura global del sistema | Definen micro-arquitecturas de subsistemas de componentes |
| Se encargan de atributos de calidad (ej. rendimiento, alta disponibilidad) | Resuelven problemas de diseño dando claridad, adaptabilidad a requerimientos cambiantes |

> **Arquitectura**: relevante a la totalidad del sistema. **Diseño**: solo concierne a un subcomponente.

### ¿Qué es un patrón?

> "Los patrones son una vía para identificar un problema concreto con su solución. Es la manera de documentar la experiencia acumulada en la estrategia empleada para solucionar un problema dentro de un contexto determinado."

### Definición de patrón de software

> "Un patrón de software describe un problema particular y recurrente de diseño que surge en contextos específicos de diseño, y presenta un esquema genérico comprobado para su solución."
> — F. Buschmann, H. Rohnert, P. Sommerlad y M. Stal, 1996

**Resumiendo:** Es una solución general, fruto de la experiencia, a un problema general que puede adaptarse a un problema concreto. Los patrones están orientados **al problema**, no a la solución. A veces la solución requiere conectar varios patrones juntos.

### Surgimiento de los patrones (línea de tiempo)

| Época | Evento |
|---|---|
| **Finales de los 70** | Surgen como técnica de diseño arquitectónico. Padre: **Christopher Alexander** (arquitecto). Fuente: *"The Timeless Way of Building"*, 1979 |
| **Por los 80** | **Tom DeMarco** sugiere que pueden ser una idea interesante para el software |
| **Hasta mediados de los 90** | Olvidados — influencia cartesiana: comenzar cada sistema de software como si fuera nuevo |
| **1994/1995** | Explosión: **Patrones GoF** (*Gang of Four* — Erich Gamma, Richard Helm, Ralph Johnson y John Vlissides). Fuente: *"Design Patterns: Elements of Reusable Object-Oriented Software"*, 1994 |
| **1998** | **Patrones GRASP**. Fuente: *"Applying UML and Patterns"*, Craig Larman, 1998 |
| **A partir de 2001** | Catálogo de **Patrones J2EE**. Fuente: *"Core J2EE Patterns"*, Deepak Alur, John Crupi, Dan Malks, 2001 |

### ¿Qué ganamos con los patrones?

- Reutilizar el conocimiento acumulado.
- Comunicación operativa (ej. "la solución está en usar el patrón Factory").
- Herramientas para afrontar malas prácticas, requisitos cambiantes, y para cumplir principios de diseño eliminando síntomas de mal diseño.

### Patrones más utilizados

- **GRASP**: General Responsibility Assignment Patterns
- **GoF**: Gang of Four
- De la tecnología (ej. J2EE)
- Microservicios

---

## 2.2 Patrones GRASP

*(General Responsibility Assignment Software Patterns)*

| Patrones GRASP |
|---|
| Experto (Expert) |
| Creador (Creator) |
| Bajo Acoplamiento (Low Coupling) |
| Alta Cohesión (High Cohesion) |
| Controlador (Controller) |
| Polimorfismo |
| Fabricación Pura (Pure Fabrication) |
| Indirección |
| No Hables con Extraños |

### Patrón Experto (Expert)

| | |
|---|---|
| **Problema** | ¿Cuál es el principio general para asignar responsabilidades a los objetos? |
| **Solución** | Asignar una responsabilidad al **experto en información**: la clase que tiene la información necesaria para realizar la responsabilidad |
| **Beneficios** | Conserva el encapsulamiento; bajo acoplamiento; alta cohesión |
| **Contraindicaciones** | Debe respetarse la separación por módulos o niveles funcionales acordada (ej. la impresión de resultados o el acceso a BD no debe incluirse en cada clase) |

**Ejemplo:** Aplicación de ventas — ¿quién debe conocer el total de una venta? Se ilustra con un diagrama de secuencia donde se asignan responsabilidades a cada clase participante.

### Patrón Creador (Creator)

| | |
|---|---|
| **Problema** | ¿Quién debe ser el responsable de crear una nueva instancia de alguna clase? |
| **Solución** | El Objeto **B** es responsable de crear objetos **A** si: B agrega objetos A / B contiene objetos A / B registra objetos A / B usa objetos A / B posee información para inicializar A |
| **Beneficio** | Bajo acoplamiento (facilita mantenimiento y reutilización) |
| **Contraindicaciones** | Cuando la creación es muy compleja, debe delegarse a una clase auxiliar llamada **Factory** |

**Ejemplos:**
- *Agregar Items de Venta*: la clase `Sale` es responsable de crear cada `SalesLineItem` → método `makeLineItem()`.
- *Partido de fútbol*: clases `Equipo`, `Futbolista`, `Gol`, `Partido`, `Árbitro` — ¿cuál clase debería crear un "Gol"?

### Patrón Bajo Acoplamiento (Low Coupling)

| | |
|---|---|
| **Problema** | ¿Cómo lograr baja dependencia entre clases (para favorecer reutilización y mantenimiento simple)? |
| **Solución** | Asignar una responsabilidad de manera que el acoplamiento permanezca bajo |
| **Beneficios** | Una clase se afecta menos por cambios en otras; clases fáciles de entender por separado; clases fáciles de reutilizar |
| **Contraindicaciones** | Acoplarse a librerías de clases utilitarias ya probadas es recomendado porque son estables |

**Diseño centralizado** vs. **Diseño descentralizado (recomendado)**.

**Ejemplo:** ¿Qué clase debería crear un nuevo pago?
- *Solución 1* (alto acoplamiento).
- *Solución 2* (bajo acoplamiento).

### Patrón Alta Cohesión (High Cohesion)

> La cohesión es la medida en la que un componente se dedica a realizar solo la tarea para la cual fue creado, delegando las tareas complementarias a otros componentes.

| | |
|---|---|
| **Problema** | ¿Cómo lograr que cada clase asuma responsabilidades relacionadas y no dispersas? |
| **Solución** | Asignar una responsabilidad a una clase de manera que la cohesión permanezca alta |
| **Beneficios** | Mejora la claridad del diseño; simplifica mantenimiento y cambios; bajo acoplamiento; facilita la reutilización |

**Ejemplo:** ¿Qué clase debe crear un Pago?
- *Solución de baja cohesión*: la clase `Register` crea el pago y pide a `Sale` que añada el nuevo pago.
- *Solución de alta cohesión*: `Register` delega a `Sale` para que se encargue de lo relativo a los pagos.

> Generalmente: **Alta Cohesión ↔ Bajo Acoplamiento** (van de la mano).

### Patrón Controlador (Controller)

| | |
|---|---|
| **Problema** | ¿Quién es el responsable de manejar eventos externos que llegan a un nivel funcional? |
| **Solución** | Agregar una clase al nivel funcional que reciba eventos externos y decida a cuál clase de negocio solicitar su procesamiento |
| **Beneficios** | Se logra la separación de niveles funcionales; mayor potencial de componentes reutilizables |

### GRASP — No hables con extraños

Un objeto solo debería interactuar con aquellos objetos que conoce directamente, no comunicarse con objetos indirectos ("extraños") a través de intermediarios.

**Ejemplo:** aplicación de tienda con clientes y cuentas bancarias. El cliente tiene una cuenta bancaria, y cada cuenta tiene una clase de detalles bancarios con información más detallada.

### GRASP — Indirección

Busca reducir el acoplamiento entre dos o más clases introduciendo un **objeto intermediario**. En lugar de que las clases interactúen directamente, lo hacen a través de este intermediario, facilitando flexibilidad, mantenimiento y reutilización.

**Ejemplo:** sistema de compras en línea. Los clientes no interactúan directamente con los proveedores; todas las transacciones pasan por un **Sistema de Pago**, que actúa como capa de indirección.

---

## 2.3 Patrones GoF

*(Gang of Four — los cuatro autores del libro)*

### ¿Cómo usar un patrón de diseño?

1. Conocer la solución genérica que brinda el patrón y el tipo de problemática que resuelve.
2. Evaluar la posibilidad de aplicar el patrón en el problema que se tiene.
3. Definir en el modelo la implementación derivada de aplicar el patrón (nombres de clases, métodos y atributos).
4. Implementar los métodos correspondientes.

### Categorías GoF

- **Patrones de Creación**
- **Patrones Estructurales**
- **Patrones de Comportamiento**

---

### A) Patrones de Creación

#### 1. Singleton

Asegura que solo se cree **una instancia** de la clase y provee un punto global de acceso a ésta.

Útil cuando se quiere un único objeto instanciado (ej.: cola de impresión, conexión a BD, generador de consecutivos).

#### 2. Prototipo (Prototype)

**Problema que resuelve:** crear una copia exacta de un objeto ya instanciado, incluyendo atributos privados, sin tener que asignarlos uno a uno.

**Solución:** especificar los tipos de objetos a crear usando una instancia prototípica, y crear nuevos objetos copiando dicho prototipo.

#### 3. Factory Method (Fábrica simple)

Proporciona una interfaz para crear objetos en una superclase, pero permite a las subclases alterar el tipo de objetos que se crearán. Encapsula la creación de objetos en una clase separada, permitiendo que las subclases decidan qué clase concreta instanciar.

#### 4. Abstract Factory

Proporciona una interfaz para crear **familias de objetos relacionados o dependientes** sin especificar sus clases concretas.

**Ejemplo (autos):** Fábrica A (autos deportivos) vs. Fábrica B (autos familiares) — mercados distintos, pero fábricas y automóviles comparten métodos y atributos comunes.

**Ejemplo 2 (renderizado multiplataforma):** aplicación que renderiza gráficos y texto en Windows y macOS — Abstract Factory crea renderizadores adecuados para cada plataforma.

#### 5. Builder

Separa la construcción de un objeto complejo de su representación. El mismo proceso de construcción puede usarse para construir diferentes representaciones.

> El patrón Builder sugiere sacar el código de construcción del objeto de su propia clase y colocarlo en objetos independientes llamados **constructores**.

---

### B) Patrones Estructurales

#### Composite

Sirve para construir objetos complejos a partir de otros más simples y similares entre sí, mediante composición recursiva y estructura en forma de árbol.

#### Facade

Proporciona una interfaz simple y unificada para acceder a sistemas o subsistemas más complejos, ocultando la complejidad y dando una capa de abstracción a los clientes.

**Ejemplo:** `PersonaFacade` utiliza `PersonaDao`/`ClienteDao` para interactuar con la base de datos. `Main` usa `PersonaFacade` sin conocer los detalles de la base de datos.

#### Bridge

Separa una **abstracción** de su **implementación**, de modo que ambas puedan variar de manera independiente. Se basa en **composición**, no en herencia.

**Ejemplo:** sistema de entretenimiento multimedia (TVs, reproductores) con controles remotos que cambian entre modos de usuario (básico/avanzado) — evita la explosión combinatoria de clases.

#### Flyweight

Ahorra memoria/recursos compartiendo información común entre objetos similares en vez de repetirla en cada objeto.

- **Estado intrínseco**: información común, inmutable, almacenada en el objeto compartido (*flyweight*).
- **Estado extrínseco**: información que varía entre objetos, almacenada en cada objeto individual.

**Ejemplo:** fichas de un juego de ajedrez. Clase `Ficha` genérica + subclases concretas (`Peon`, `Torre`...) con la info extrínseca (posición). Un *Flyweight Factory* gestiona las fichas compartidas.

#### Proxy

Controla el acceso a un objeto o agrega funcionalidad adicional sin modificar su código fuente (creación diferida, control de credenciales/permisos).

**Funcionamiento:** Sujeto (Subject) / Objeto Real (Real Subject) / Proxy (intermediario entre cliente y objeto real).

**Ejemplo:** servicio de clima vía API en línea, con Proxy para administrar solicitud y caché de datos (5 minutos).

#### Adapter

Permite que clases con interfaces incompatibles trabajen juntas, funcionando como traductor entre dos interfaces.

**Elementos:** Cliente / Interfaz Objetivo / Clase Adaptable (Adaptee) / Adaptador (intermediario).

**Ejemplo:** dos APIs bancarias — API Antigua (XML) y API Nueva (JSON). Un Adapter traduce las solicitudes XML→JSON y las respuestas JSON→XML.

---

### C) Patrones de Comportamiento

#### State

Permite que un objeto cambie su comportamiento en función de su estado interno, creando una clase separada para cada estado posible.

#### Strategy

Define una familia de algoritmos, encapsula cada uno de ellos y los hace intercambiables, eligibles en tiempo de ejecución.

#### Observer

Un objeto (el "sujeto") notifica a otros objetos (los "observadores") cuando sucede un cambio en su estado. Relación uno a muchos, comunicación flexible y desacoplada.

**Ejemplo:** sistema de noticias en línea — el sujeto es la fuente de noticias; los observadores son los suscriptores.

#### Iterator

Proporciona una forma de acceder secuencialmente a los elementos de una colección sin exponer su estructura interna.

**Componentes:** Iterador (`hasNext()`, `next()`) / Agregado-Colección (`createIterator()`).

#### Cadena de Responsabilidad (Chain of Responsibility)

Permite pasar una solicitud a lo largo de una cadena de objetos hasta encontrar un manejador adecuado.

**Estructura:** Manejador Abstracto (`setNextHandler()`, `handle()`) / Manejadores Concretos / Cliente.

**Ejemplo:** Manejador de Correo — `ManejadorCorreoSpam`, `ManejadorCorreoImportante`, `ManejadorCorreoPredeterminado`, encadenados.

#### Decorator

Añade responsabilidades a un objeto de manera dinámica sin modificar su estructura original, mediante composición (envoltura).

**Ejemplo:** Sistema de mensajes — `EnvioMensaje` (componente), `EnvioMensajeBasico`, decoradores `DecoradorSMS`, `DecoradorFacebook`, `DecoradorWhatsApp`.

#### Visitor

Permite añadir nuevas operaciones a una estructura de objetos existente sin modificarla, mediante una clase "visitante" separada.

**Elementos:** Visitor / ConcreteVisitor / Element (`accept()`) / ConcreteElement / ObjectStructure.

#### Template Method

Define la estructura general de un algoritmo, permitiendo que algunos pasos sean implementados por las subclases, manteniendo intacto el flujo general.

**Ejemplo:** `Application`/`Document` con subclases `DrawApplication`/`DrawDocument` y `SpreadsheetApplication`/`SpreadsheetDocument`.

#### Command

Encapsula una solicitud (comando) como un objeto, permitiendo encolar solicitudes, historial y *undo*. Desacopla emisor de la solicitud del ejecutor de la acción.

**Ejemplo:** automatización de fábrica — `Command` abstracto con `execute()`, subclases `StartMachineCommand`, `StopMachineCommand`, `ChangeModeCommand`.

---

## 2.4 Patrones de diseño general

### Delegate (Delegación)

Un objeto reenvía o delega una llamada/tarea a otro objeto, manteniendo separación de responsabilidades. Ventaja: reemplaza la herencia (alto acoplamiento) por composición/delegación.

### Interfaces (diseño general)

El comportamiento de los objetos puede describirse mediante una caracterización abstracta de su interfaz (ej. un libro: prestar, devolver, reservar). Ventajas: sin acoplamiento cliente-implementación; la implementación puede cambiar sin afectar al cliente.

**¿Cómo resolver la creación del objeto que implementa la interfaz?**
- Solución obvia: `I i = new C();` → acopla al cliente con `C`.
- Mejor solución: **Inversion of Control (IoC)** / **Dependency Injection (DI)**.

### Dependency Injection (Inyección de Dependencias)

Toma la responsabilidad de **crear las instancias** que otro objeto necesita y se las suministra, en lugar de que la propia clase las cree.

**Ejemplo sin DI:**
```java
public class Vehiculo {
  private Motor motor = new Motor();

  public Double enAceleracionDePedal(int presionDePedal) {
    motor.setPresionDePedal(presionDePedal);
    int torque = motor.getTorque();
    Double velocidad = ...;
    return velocidad;
  }
}
```

**Con DI:** `Avion` no crea su propio `Motor`; lo recibe de `AvionFactory`. `Avion` solo lo utiliza.

---

## 2.5 Elementos esenciales de un patrón de diseño

1. **Nombre**: describe un problema de diseño.

2. **Problema a resolver**:
   - **Intención**: ¿Qué hace el patrón?
   - **Motivación**: escenario de ejemplo.
   - **Aplicabilidad**: ¿Cuándo aplicar? ¿A qué problema?

3. **Solución prevista**:
   - **Estructura**: diagramas de clases y de trazas de eventos.
   - **Participantes**: clases que participan en el patrón.
   - **Colaboraciones**: ¿cómo colaboran los objetos?
   - **Implementación**: consideraciones y variaciones.

4. **Consecuencias** (buenas y malas): impacto en espacio de almacenamiento, tiempo de ejecución, flexibilidad, portabilidad, extensibilidad.

---

## Bibliografía

- *UML y patrones. Introducción al análisis y diseño orientado a objetos.* Craig Larman. Prentice Hall Hispanoamérica, 1999. Capítulos 18, 19, 34 y 35 (GRASP).
- *Design Patterns, Elements of Reusable Object-Oriented Software.* Gamma, E., Helm, R., Johnson, R., Vlissides, J. 2000. (GoF)
- *El modelo C4 de documentación para la Arquitectura de Software*, Javier Vivanco — Medium.
- *Los 10 principios de usabilidad web de Jakob Nielsen* — semrush.com/blog/usabilidad-web-principios-jakob-nielsen
