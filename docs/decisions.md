# Decisiones de arquitectura

Este documento registra algunas de las decisiones tomadas durante la evolución conceptual de la solución.

---

## ADR-001 — Automatizar el proceso manual

### Problema

El proceso requería múltiples tareas repetitivas realizadas manualmente.

### Decisión

Automatizar las tareas repetitivas mediante programación.

### Primera implementación

Python.

### Resultado

Reducción de intervención manual y estandarización del procesamiento.

---

## ADR-002 — Centralizar las automatizaciones

### Problema

A medida que aumenta la cantidad de scripts y ejecutables, también aumenta la complejidad de mantenimiento.

### Decisión

Centralizar la orquestación utilizando una plataforma de automatización.

### Tecnología

n8n.

### Beneficios

- administración centralizada;
- historial de ejecuciones;
- mantenimiento simplificado;
- menor dependencia de estaciones de trabajo;
- mejor trazabilidad.

---

## ADR-003 — Utilizar APIs

### Problema

Las integraciones dependientes de aplicaciones instaladas localmente generan acoplamiento con una estación de trabajo.

### Decisión

Priorizar integraciones mediante APIs cuando sea posible.

### Ejemplo

Microsoft Graph API.

### Beneficios

- menor dependencia local;
- autenticación centralizada;
- integración programática;
- mayor flexibilidad.

---

## ADR-004 — Separar operación de implementación

### Problema

Los usuarios operativos no deberían necesitar ejecutar ni mantener componentes técnicos.

### Decisión

Mantener la automatización dentro de una plataforma administrada y entregar al usuario únicamente el resultado necesario.

### Resultado esperado

Menor intervención operativa y mayor control técnico.
