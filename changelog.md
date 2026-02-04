# 📝 Changelog

Historial de cambios y versiones de la API de BuhoPago.

---

## Versión 1.1.1 (2026-02-04)

### 🔧 Correcciones Técnicas y Mejoras

#### ⚠️ BREAKING CHANGE: Tipo de `transaction_id`
Cambio en el tipo de dato de `transaction_id` en Direct Payments API para mayor consistencia y eficiencia:

**Endpoints afectados:**
- `POST /public-api/direct-payment/generate-otp` (response)
- `POST /public-api/direct-payment/verify-otp` (request)

**Cambio:**
```javascript
// ❌ Antes (v1.1.0)
{
  "transaction_id": "95"  // String
}

// ✅ Ahora (v1.1.1)
{
  "transaction_id": 95  // Integer
}
```

**Acción Requerida:**
- Actualiza tu código cliente para manejar `transaction_id` como `integer` en lugar de `string`
- Los parsers JSON deberían manejar esto automáticamente
- Verifica que no estés forzando conversión a string

#### 📝 Formato de Cédula Mejorado
- Ahora acepta formato sin prefijo: `"30552028"` (recomendado)
- También sigue aceptando formato con prefijo: `"V-30552028"`
- Se recomienda migrar al formato sin prefijo para evitar problemas

#### 🏦 Validación de Códigos de Banco
- Asegúrate de usar códigos de banco de 4 dígitos:
  - ✅ `"0105"` (Mercantil)
  - ✅ `"0102"` (Banco de Venezuela)
  - ✅ `"0134"` (Banesco)
  - ❌ `"105"` (Inválido - falta el cero inicial)

#### 🔄 Servicio OTP Multi-Tenant
- Mejoras internas en el servicio de OTP para mejor soporte multi-tenant
- Optimización de imports y dependencias
- Mayor confiabilidad en el envío de códigos OTP

#### ✅ Estabilidad General
- Corrección de errores internos de validación
- Mejor manejo de excepciones
- Logs más descriptivos para debugging

### 📚 Documentación
- Actualizada documentación con tipos de datos correctos
- Añadidas tablas de referencia para formatos de datos
- Ejemplos de código actualizados con mejores prácticas
- Nueva sección de códigos de banco comunes

### 🐛 Bug Fixes
- Corregido: Error de validación Pydantic en `DirectPaymentOTPResponse`
- Corregido: Imports incorrectos del servicio OTP
- Corregido: Inconsistencia entre tipos de `transaction_id` en diferentes schemas

---

## Versión 1.2.1 (2026-02-04)

### 🔧 Mejoras Internas

#### Enhanced Transaction Tracking
Mejoras significativas en el tracking y auditoría de transacciones para mejor trazabilidad de pagos multi-moneda:

**Nuevos campos en PaymentTransaction:**
- `currency_original` - Moneda original del pago (USD, EUR, VES)
- `amount_original` - Monto original antes de conversión a VES
- `exchange_rate` - Tasa de cambio aplicada en el momento del pago
- `description` - Descripción del pago
- `reference_id` - ID de referencia externa para integración
- `metadata` - Metadata adicional en formato JSON

**Impacto:**
- ✅ Mejor auditoría: Ahora se guarda la moneda y monto original de cada transacción
- ✅ Trazabilidad completa: Conocer exactamente qué tasa de cambio se usó
- ✅ Consistencia: Todos los endpoints de guest payments ahora guardan esta información
- ✅ Sin breaking changes: Los campos son opcionales y compatibles con código existente

**Endpoints Mejorados:**
- `POST /public-api/direct-payment/generate-otp` - Ahora guarda información completa de moneda
- `POST /payment-links/{slug}/guest/generate-otp` - Tracking mejorado para pagos guest
- `POST /guest/generate-otp` - Pagos directos guest con auditoría completa

**Migración Requerida:**
```bash
alembic revision --autogenerate -m "Add transaction tracking fields"
alembic upgrade head
```

---

## Versión 1.2.0 (2026-02-04)

### ✨ Nuevas Características

#### 💳 Credits API - Créditos Inmediatos (Empresarial)
Nueva funcionalidad para usuarios empresariales con permisos especiales que permite retirar fondos procesados mediante créditos inmediatos.

**Endpoints:**
- **POST `/api/v1/credits/execute`** - Ejecutar crédito inmediato a cuenta bancaria
- **GET `/api/v1/credits/capacity`** - Consultar capacidad de procesamiento disponible

**Sistema de Capacidad de Procesamiento:**
- Auto-tracking de volumen procesado en transacciones completadas
- Límite de retiro basado en transacciones recibidas
- Control por API key con scope `credits:execute`

**Características de Seguridad:**
- Nuevo scope de permisos: `credits:execute`
- Validación de capacidad disponible antes de ejecutar
- Auditoría completa de todas las operaciones
- Restricción a usuarios con KYC aprobado

**Caso de Uso:**
Ideal para empresas que:
- Reciben transacciones de terceros
- Necesitan convertir fondos a stablecoins
- Requieren liquidez inmediata basada en volumen procesado

### 🗄️ Base de Datos

#### Nueva Tabla: `api_key_processing_capacity`
Tracking de volumen procesado y capacidad de retiro por API key:
- `volume_processed_total` - Total histórico procesado
- `volume_credited_total` - Total retirado con créditos
- `volume_available` - Disponible para retirar
- Índices optimizados para consultas rápidas
- Relación uno-a-uno con API keys

### 🔧 Mejoras Técnicas

#### Auto-tracking de Volumen
- Las transacciones completadas automáticamente incrementan la capacidad disponible
- Actualización en tiempo real del volumen procesado
- Sistema de manejo de errores robusto (no bloquea transacciones)

#### Servicios
- Nuevo servicio: `ProcessingCapacityService` para gestión de volumen
- Integración con `CreditService` existente para ejecución de créditos
- Polling automático para verificar estado de créditos

### 📚 Documentación
- Nueva guía completa: [Credits API Documentation](credits-api.md)
- Ejemplos en Python, JavaScript/Node.js y cURL
- Diagramas de flujo y mejores prácticas
- Guía de manejo de errores específicos

### ⚠️ Requisitos para Credits API
- API key con scope `credits:execute` (solicitar a soporte)
- KYC aprobado y verificado
- Al menos una cuenta bancaria registrada
- Volumen procesado disponible

---

## Versión 1.1.0 (2026-02-03)

### ✨ Nuevas Características

#### Bank Accounts API
- **POST `/bank-accounts`** - Crear cuenta bancaria
- **GET `/bank-accounts`** - Listar cuentas bancarias
- **GET `/bank-accounts/{id}`** - Obtener cuenta específica
- **PUT `/bank-accounts/{id}`** - Actualizar cuenta bancaria
- **DELETE `/bank-accounts/{id}`** - Eliminar cuenta bancaria (soft/hard delete)

#### Direct Payments API
- **POST `/direct-payment/generate-otp`** - Generar OTP para pago directo
- **POST `/direct-payment/verify-otp`** - Verificar OTP y procesar pago
- 🎯 Caso de uso completo: Tienda online con captura de datos en interfaz propia

### 📋 Requisitos
- **Obligatorio**: Al menos una cuenta bancaria activa para crear payment links

### ⚡ Optimizaciones
- QR codes deshabilitados para links creados vía API (reducción de costos)
- Uso de patrones CRUD para mejor mantenibilidad

### 📚 Documentación
- Script de pruebas completo (`pruebaApi.py`) con 26 tests
- Documentación ampliada con ejemplos en HTML, JavaScript y Python Flask
- Casos de uso prácticos para diferentes escenarios

---

## Versión 1.0.0 (2024-01-15)

### 🚀 Lanzamiento Inicial

#### Payment Links
- **POST `/payment-links`** - Crear payment links
- **GET `/payment-links`** - Listar payment links
- **GET `/payment-links/{id}`** - Consultar payment link específico
- **DELETE `/payment-links/{id}`** - Desactivar payment link

#### Características Generales
- Autenticación con API Keys (Bearer token)
- Soporte para múltiples monedas: USD, EUR, VES
- Sistema de webhooks para notificaciones
- Conversión automática de divisas a VES
- Metadata personalizable en payment links

#### Documentación
- Documentación completa de API
- Ejemplos en cURL y Python
- Guías de integración

---

## 🔮 Próximas Versiones

### Planificado para v1.3.0
- Soporte para pagos recurrentes
- API de transacciones y reportes avanzados
- Webhooks con retry automático y firma HMAC
- Soporte para refunds/devoluciones
- Dashboard analytics API
- Endpoints de administración de scopes para API keys

### En Consideración
- SDKs oficiales (Python, JavaScript, PHP)
- GraphQL API
- Soporte para más monedas
- Integración con plataformas de e-commerce

---

## 📢 Cómo Actualizar

### Breaking Changes

#### v1.1.1 - Tipo de `transaction_id`
**Impacto:** Bajo - Afecta solo a usuarios de Direct Payments API

Si estás usando los endpoints de Direct Payments, necesitas actualizar tu código para manejar `transaction_id` como `integer`:

```python
# Python - Antes
transaction_id = str(response['transaction_id'])  # ❌ Ya no es necesario

# Python - Ahora
transaction_id = response['transaction_id']  # ✅ Ya es int

# JavaScript - Antes
const transactionId = response.transaction_id.toString()  # ❌ Ya no es necesario

# JavaScript - Ahora
const transactionId = response.transaction_id  # ✅ Ya es number
```

### Versiones Anteriores
No hay breaking changes en versiones anteriores. Todos los endpoints existentes permanecen compatibles.

### Nuevas Características
Los nuevos endpoints están disponibles inmediatamente. Solo necesitas:

1. Asegurarte de tener al menos una cuenta bancaria activa
2. Actualizar tu integración si quieres usar Direct Payments

### Deprecaciones
Ninguna por el momento.

---

## 🐛 Bug Fixes y Mejoras

### v1.1.0
- Corregido: Validación de campos en bank accounts
- Mejorado: Manejo de errores en conversión de moneda
- Optimizado: Performance en listado de payment links
- Corregido: Timeout en OTP para direct payments

---

## 💬 Feedback

¿Tienes sugerencias para nuevas features? Contáctanos en:
- **Email**: soporte@buhopago.com
- **Feedback**: feedback@buhopago.com
