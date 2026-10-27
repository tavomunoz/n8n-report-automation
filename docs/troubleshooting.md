# Troubleshooting

Este documento describe posibles errores y situaciones que deben contemplarse en una arquitectura de automatización de reportería.

El objetivo es que los procesos puedan finalizar de manera controlada, registrar correctamente los problemas y facilitar su diagnóstico.

---

# 1. Error de autenticación

## Síntoma

La API devuelve un error de autenticación o no permite acceder al recurso solicitado.

## Posibles causas

- Tenant ID incorrecto;
- Client ID incorrecto;
- Client Secret incorrecto;
- credencial expirada;
- permisos insuficientes;
- configuración incorrecta de la aplicación.

## Verificar

- Tenant ID;
- Client ID;
- Client Secret;
- fecha de expiración de las credenciales;
- permisos asignados a la aplicación;
- permisos requeridos por el recurso.

## Importante

Las credenciales reales nunca deben registrarse dentro de logs ni mensajes de error.

---

# 2. No se encuentra información

## Síntoma

El workflow se ejecuta correctamente pero no encuentra información para procesar.

Esto no necesariamente representa un error técnico.

Puede significar simplemente que no existen datos para el período solicitado.

## Verificar

- filtros utilizados;
- fecha de procesamiento;
- origen de la información;
- disponibilidad de datos;
- parámetros recibidos.

## Comportamiento esperado

El proceso debería finalizar de manera controlada.

Ejemplo:

```text
Buscar información
        │
        ▼
¿Existen datos?
        │
   ┌────┴────┐
   │         │
  NO        SÍ
   │         │
   ▼         ▼
Registrar   Continuar
sin datos   procesamiento
```

No encontrar información no debería provocar necesariamente una falla general del workflow.

---

# 3. Archivo inexistente

## Síntoma

El proceso esperaba recibir o descargar un archivo pero el archivo no existe.

## Posibles causas

- archivo todavía no generado;
- nombre incorrecto;
- ubicación incorrecta;
- filtro incorrecto;
- problema en el sistema origen.

## Comportamiento esperado

El workflow debería:

1. detectar que el archivo no existe;
2. evitar continuar con el procesamiento;
3. registrar la situación;
4. finalizar de manera controlada.

---

# 4. Archivo vacío

## Síntoma

El archivo existe pero no contiene registros para procesar.

## Riesgo

Continuar procesando un archivo vacío podría generar:

- reportes incorrectos;
- archivos sin información;
- errores en etapas posteriores;
- notificaciones innecesarias.

## Comportamiento esperado

```text
Archivo recibido
       │
       ▼
¿Tiene registros?
       │
   ┌───┴───┐
   │       │
  NO      SÍ
   │       │
   ▼       ▼
Registrar Procesar
sin datos información
```

El workflow debería finalizar de manera controlada indicando que no existían registros.

---

# 5. Columnas faltantes

## Síntoma

El archivo recibido no contiene todas las columnas necesarias para ejecutar el procesamiento.

Por ejemplo, para este proyecto demostrativo esperamos:

```text
fecha
canal
estado
cantidad
```

Si alguna de estas columnas no existe, el procesamiento no debería continuar.

## Validación

```text
Archivo
   │
   ▼
Validar columnas
   │
   ├── Correctas
   │       │
   │       ▼
   │    Procesar
   │
   └── Incorrectas
           │
           ▼
      Error controlado
```

## Comportamiento esperado

Registrar:

- columna faltante;
- etapa donde ocurrió la validación;
- fecha de ejecución.

---

# 6. Formato de datos incorrecto

## Síntoma

Las columnas existen pero contienen valores con formatos inesperados.

Ejemplos:

```text
fecha = "texto"
cantidad = "ABC"
canal = vacío
estado = vacío
```

cuando deberían contener valores válidos.

## Comportamiento esperado

El proceso debería validar los datos antes de realizar cálculos o generar reportes.

Ejemplo:

```text
Datos
  │
  ▼
Validación
  │
  ├── Válidos
  │      │
  │      ▼
  │   Procesar
  │
  └── Inválidos
         │
         ▼
    Registrar error
```

---

# 7. Error durante el procesamiento

## Síntoma

Los datos fueron obtenidos correctamente pero ocurre un error durante una transformación o cálculo.

## Ejemplos

- conversión de tipos;
- cálculo incorrecto;
- valor inesperado;
- estructura de datos inválida;
- excepción durante una transformación.

## Registrar

Cuando sea posible, registrar:

- etapa;
- fecha;
- tipo de error;
- mensaje de error;
- identificador de ejecución.

## Nunca registrar

No deben almacenarse en logs:

- passwords;
- tokens;
- Client Secrets;
- credenciales;
- datos personales sensibles.

---

# 8. Error de API

## Síntoma

Una integración externa devuelve un error.

Ejemplos:

```text
401 Unauthorized
403 Forbidden
404 Not Found
429 Too Many Requests
500 Internal Server Error
```

## Interpretación general

### 401

Problema relacionado con autenticación.

### 403

La aplicación está autenticada pero no posee permisos suficientes.

### 404

El recurso solicitado no fue encontrado.

### 429

Se alcanzó un límite temporal de solicitudes.

### 500

El servicio externo presentó un error interno.

## Comportamiento esperado

El workflow debería registrar:

- servicio;
- código HTTP;
- etapa;
- fecha;
- mensaje técnico permitido.

Nunca debería registrar tokens o credenciales.

---

# 9. Error de almacenamiento

## Síntoma

El reporte fue generado correctamente pero no puede almacenarse en el destino esperado.

## Verificar

- autenticación;
- permisos;
- ubicación destino;
- disponibilidad del servicio;
- nombre del archivo;
- existencia de la carpeta destino.

## Importante

La generación del reporte y su almacenamiento son etapas diferentes.

```text
Procesamiento
      │
      ▼
Generar reporte
      │
      ▼
¿Reporte generado?
      │
      ▼
Almacenar
      │
      ▼
¿Almacenamiento correcto?
```

Esto permite identificar exactamente dónde ocurrió el problema.

---

# 10. Error de notificación

## Síntoma

El procesamiento terminó correctamente pero falla el envío de la notificación.

## Importante

La generación del reporte y la notificación deben considerarse etapas independientes.

Un error enviando una notificación no necesariamente significa que el procesamiento haya fallado.

Ejemplo:

```text
Procesamiento
      │
      ▼
Reporte generado
      │
      ▼
Reporte almacenado
      │
      ▼
Enviar notificación
      │
   ┌──┴──┐
   │     │
  OK   ERROR
   │     │
   ▼     ▼
Fin   Registrar
      error de
      notificación
```

---

# 11. Error inesperado

## Síntoma

Ocurre una excepción que no pertenece a ninguno de los escenarios previamente contemplados.

## Comportamiento esperado

El workflow debería:

1. detener el procesamiento de manera controlada;
2. registrar la etapa donde ocurrió el problema;
3. registrar el mensaje técnico permitido;
4. conservar la trazabilidad de la ejecución;
5. permitir posteriormente analizar la causa.

---

# Estrategia general de manejo de errores

La arquitectura debería seguir conceptualmente este modelo:

```text
                INICIO
                  │
                  ▼
          Obtener información
                  │
                  ▼
          ¿Existe información?
             │          │
            NO         SÍ
             │          │
             ▼          ▼
        Registrar    Validar
        sin datos      datos
                        │
                        ▼
                  ¿Datos válidos?
                    │       │
                   NO      SÍ
                    │       │
                    ▼       ▼
                 Error   Procesar
                           │
                           ▼
                    Generar reporte
                           │
                           ▼
                       Almacenar
                           │
                           ▼
                       Notificar
                           │
                           ▼
                          FIN
```

---

# Principio de diseño

Un proceso automatizado no debería limitarse a funcionar cuando todo sale correctamente.

También debe poder responder de manera controlada cuando:

- no existen datos;
- los datos son incorrectos;
- falla una integración;
- falla una autenticación;
- falla el procesamiento;
- falla el almacenamiento;
- falla una notificación.

Esto permite construir automatizaciones más:

- confiables;
- mantenibles;
- observables;
- trazables;
- fáciles de diagnosticar.

---

# Seguridad durante el troubleshooting

Los mecanismos de diagnóstico nunca deben exponer información sensible.

No registrar:

```text
CLIENT_SECRET
ACCESS_TOKEN
PASSWORD
API_KEY
datos personales
credenciales
```

En cambio, registrar información técnica segura como:

```text
fecha
workflow
etapa
estado
código HTTP
mensaje de error permitido
identificador de ejecución
```

---

# Objetivo

El objetivo del troubleshooting no es solamente detectar que un proceso falló.

También debe permitir responder:

```text
¿Qué falló?
      ↓
¿En qué etapa?
      ↓
¿Por qué falló?
      ↓
¿Qué información puedo utilizar para diagnosticarlo?
      ↓
¿Cómo evito que vuelva a ocurrir?
```

Esta trazabilidad forma parte de la evolución desde una automatización básica hacia una solución mantenible y administrable.
