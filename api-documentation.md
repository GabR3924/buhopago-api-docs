# BuhoPago API - Documentación para Desarrolladores

## Introducción

La API de BuhoPago permite a desarrolladores externos integrar pagos instantáneos (débito inmediato) en sus aplicaciones. Con nuestra API, puedes:

- Crear enlaces de pago dinámicos (payment links)
- Consultar el estado de los pagos
- Recibir notificaciones webhook cuando se complete un pago
- Gestionar múltiples transacciones de forma programática

## Tabla de Contenidos

1. [Autenticación](#autenticación)
2. [Configuración Inicial - Bank Accounts](#configuración-inicial---bank-accounts)
3. [Endpoints Disponibles](#endpoints-disponibles)
4. [💳 Direct Payments - Pagos Directos](#-direct-payments---pagos-directos)
5. [Modelos de Datos](#modelos-de-datos)
6. [Códigos de Respuesta](#códigos-de-respuesta)
7. [Webhooks](#webhooks)
8. [Ejemplos de Integración](#ejemplos-de-integración)
9. [Mejores Prácticas](#mejores-prácticas)
10. [Limitaciones y Cuotas](#limitaciones-y-cuotas)

---

## Autenticación

La API de BuhoPago utiliza **API Keys** para autenticación. Todas las peticiones deben incluir tu API key en el header `Authorization`.

### Obtener tu API Key

1. Inicia sesión en tu cuenta de BuhoPago
2. Ve a la sección de "API Keys" en tu dashboard
3. Crea una nueva API key:

```bash
POST https://api.buhopago.com/api-keys/create
Authorization: Bearer <tu-token-de-sesion>
Content-Type: application/json

{
  "name": "Mi Tienda Online - Producción",
  "environment": "live",
  "expires_in_days": 365
}
```

**Respuesta:**
```json
{
  "key": "bp_live_abc123def456ghi789...",
  "key_info": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Mi Tienda Online - Producción",
    "key_prefix": "bp_live_abc1...",
    "environment": "live",
    "is_active": true,
    "created_at": "2024-01-15T10:30:00Z",
    "expires_at": "2025-01-15T10:30:00Z"
  },
  "warning": "⚠️ Guarda esta key de forma segura. No podrás volver a verla."
}
```

> **⚠️ IMPORTANTE:** La API key completa solo se muestra **una vez**. Guárdala en un lugar seguro (variables de entorno, gestor de secretos, etc.).

### Tipos de API Keys

- **`bp_live_xxx`**: Para producción (pagos reales)
- **`bp_test_xxx`**: Para pruebas (modo sandbox)

### Usar tu API Key

Incluye tu API key en el header `Authorization` de todas las peticiones:

```http
Authorization: Bearer bp_live_abc123def456ghi789...
```

---

## Configuración Inicial - Bank Accounts

### ⚠️ IMPORTANTE: Requisito Previo

**Debes tener al menos una cuenta bancaria configurada antes de poder crear payment links.** Si intentas crear un payment link sin tener una cuenta bancaria, recibirás un error `400 Bad Request`.

### Verificar si tienes cuentas bancarias

Antes de empezar, verifica si ya tienes cuentas bancarias configuradas:

```bash
curl https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_xxx"
```

Si la respuesta es un array vacío `[]`, necesitas crear tu primera cuenta bancaria.

### Crear tu primera cuenta bancaria

```bash
curl -X POST https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Mi Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente"
  }'
```

**Respuesta (201 Created):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "account_holder": "Mi Empresa S.A.",
  "rif": "J-12345678-9",
  "account_type": "Corriente",
  "active": true,
  "created_at": "2026-02-03T10:30:00Z",
  "updated_at": "2026-02-03T10:30:00Z"
}
```

Una vez creada tu cuenta bancaria, ya puedes crear payment links. El sistema usará automáticamente esta cuenta para recibir los pagos.

### Ver todos los endpoints de Bank Accounts

Para más detalles sobre la gestión de cuentas bancarias, consulta la sección [Endpoints de Bank Accounts](#7-gestión-de-bank-accounts).

---

## Endpoints Disponibles

### Base URL

```
https://api.buhopago.com/public-api
```

### 1. Crear Payment Link

Crea un nuevo enlace de pago para recibir dinero.

**Endpoint:**
```http
POST /public-api/payment-links
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
Content-Type: application/json
```

**Body:**
```json
{
  "amount": 50.00,
  "currency": "USD",
  "description": "Suscripción mensual - Plan Premium",
  "webhook_url": "https://tusitio.com/webhooks/buhopago",
  "return_url": "https://tusitio.com/success",
  "cancel_url": "https://tusitio.com/cancel",
  "reference_id": "order_12345",
  "expires_in_hours": 24,
  "metadata": {
    "customer_id": "cus_abc123",
    "plan": "premium",
    "custom_field": "valor_personalizado"
  }
}
```

**Parámetros:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `amount` | float | ✅ | Monto del pago (debe ser > 0) |
| `currency` | string | ✅ | Moneda: `USD`, `EUR`, o `VES` |
| `description` | string | ✅ | Descripción del pago (3-200 caracteres) |
| `webhook_url` | string | ❌ | URL para recibir notificación de pago completado |
| `return_url` | string | ❌ | URL de redirección después del pago exitoso |
| `cancel_url` | string | ❌ | URL de redirección si el usuario cancela |
| `reference_id` | string | ❌ | ID de referencia externa (ej: order_id) |
| `expires_in_hours` | integer | ❌ | Horas hasta expiración (máx 8760 = 1 año) |
| `metadata` | object | ❌ | Datos adicionales en formato JSON |

**Respuesta exitosa (201 Created):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "slug": "pay_abc123def",
  "url": "https://buhopago.com/pay/pay_abc123def",
  "amount": 50.00,
  "currency": "USD",
  "description": "Suscripción mensual - Plan Premium",
  "status": "active",
  "qr_url": "https://api.buhopago.com/qr/pay_abc123def.png",
  "reference_id": "order_12345",
  "metadata": {
    "customer_id": "cus_abc123",
    "plan": "premium"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "expires_at": "2024-01-16T10:30:00Z",
  "payment_count": 0,
  "total_collected": 0.0
}
```

---

### 2. Consultar Payment Link

Obtiene la información de un payment link específico.

**Endpoint:**
```http
GET /public-api/payment-links/{link_id}
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
```

**Respuesta exitosa (200 OK):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "slug": "pay_abc123def",
  "url": "https://buhopago.com/pay/pay_abc123def",
  "amount": 50.00,
  "currency": "USD",
  "description": "Suscripción mensual - Plan Premium",
  "status": "active",
  "qr_url": "https://api.buhopago.com/qr/pay_abc123def.png",
  "reference_id": "order_12345",
  "metadata": {
    "customer_id": "cus_abc123"
  },
  "created_at": "2024-01-15T10:30:00Z",
  "expires_at": "2024-01-16T10:30:00Z",
  "payment_count": 2,
  "total_collected": 100.00
}
```

---

### 3. Listar Payment Links

Lista todos los payment links de tu cuenta.

**Endpoint:**
```http
GET /public-api/payment-links
```

**Query Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `status` | string | `null` | Filtrar por estado: `active`, `expired` |
| `page` | integer | `1` | Número de página |
| `page_size` | integer | `50` | Elementos por página (máx 100) |

**Ejemplo:**
```http
GET /public-api/payment-links?status=active&page=1&page_size=20
Authorization: Bearer bp_live_xxx
```

**Respuesta exitosa (200 OK):**
```json
{
  "links": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "slug": "pay_abc123def",
      "url": "https://buhopago.com/pay/pay_abc123def",
      "amount": 50.00,
      "currency": "USD",
      "description": "Pago 1",
      "status": "active",
      "created_at": "2024-01-15T10:30:00Z"
    },
    {
      "id": "660f9511-f3ac-52e5-b827-557766551111",
      "slug": "pay_xyz789ghi",
      "url": "https://buhopago.com/pay/pay_xyz789ghi",
      "amount": 100.00,
      "currency": "USD",
      "description": "Pago 2",
      "status": "active",
      "created_at": "2024-01-14T15:20:00Z"
    }
  ],
  "total": 42,
  "page": 1,
  "page_size": 20
}
```

---

### 4. Desactivar Payment Link

Desactiva un payment link (ya no aceptará nuevos pagos).

**Endpoint:**
```http
DELETE /public-api/payment-links/{link_id}
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
```

**Respuesta exitosa (204 No Content):**
```
(Sin contenido)
```

---

### 5. Ver Transacciones de un Link

Obtiene todas las transacciones asociadas a un payment link.

**Endpoint:**
```http
GET /public-api/payment-links/{link_id}/transactions
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
```

**Respuesta exitosa (200 OK):**
```json
[
  {
    "id": "txn_abc123",
    "payment_link_slug": "pay_abc123def",
    "amount": 50.00,
    "status": "completed",
    "payer_email": "cliente@example.com",
    "completed_at": "2024-01-15T11:45:00Z",
    "bank_reference": "REF123456"
  }
]
```

---

### 6. Health Check

Verifica que la API esté funcionando. **No requiere autenticación.**

**Endpoint:**
```http
GET /public-api/health
```

**Respuesta exitosa (200 OK):**
```json
{
  "status": "ok",
  "version": "1.0.0",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

### 7. Gestión de Bank Accounts

Los siguientes endpoints te permiten gestionar tus cuentas bancarias. **Recuerda que necesitas al menos una cuenta bancaria activa para poder crear payment links.**

#### 7.1. Crear Cuenta Bancaria

Crea una nueva cuenta bancaria para recibir pagos.

**Endpoint:**
```http
POST /public-api/bank-accounts
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
Content-Type: application/json
```

**Body:**
```json
{
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "account_holder": "Mi Empresa S.A.",
  "rif": "J-12345678-9",
  "account_type": "Corriente"
}
```

**Parámetros:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `bank` | string | ✅ | Nombre del banco (máx 100 caracteres) |
| `account_number` | string | ✅ | Número de cuenta (máx 20 caracteres) |
| `account_holder` | string | ✅ | Titular de la cuenta (máx 200 caracteres) |
| `rif` | string | ✅ | RIF del titular (máx 20 caracteres) |
| `account_type` | string | ❌ | Tipo de cuenta: "Corriente", "Ahorro", etc. |

**Respuesta exitosa (201 Created):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "account_holder": "Mi Empresa S.A.",
  "rif": "J-12345678-9",
  "account_type": "Corriente",
  "active": true,
  "created_at": "2026-02-03T10:30:00Z",
  "updated_at": "2026-02-03T10:30:00Z"
}
```

---

#### 7.2. Listar Cuentas Bancarias

Obtiene todas las cuentas bancarias configuradas.

**Endpoint:**
```http
GET /public-api/bank-accounts
```

**Query Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `active_only` | boolean | `true` | Mostrar solo cuentas activas |

**Ejemplo:**
```http
GET /public-api/bank-accounts?active_only=true
Authorization: Bearer bp_live_xxx
```

**Respuesta exitosa (200 OK):**
```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "user_id": "123e4567-e89b-12d3-a456-426614174000",
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Mi Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente",
    "active": true,
    "created_at": "2026-02-03T10:30:00Z",
    "updated_at": "2026-02-03T10:30:00Z"
  },
  {
    "id": "660f9511-f3ac-52e5-b827-557766551111",
    "user_id": "123e4567-e89b-12d3-a456-426614174000",
    "bank": "Banesco",
    "account_number": "01340987654321098765",
    "account_holder": "Mi Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Ahorro",
    "active": true,
    "created_at": "2026-02-03T11:00:00Z",
    "updated_at": "2026-02-03T11:00:00Z"
  }
]
```

---

#### 7.3. Obtener Cuenta Específica

Consulta información de una cuenta bancaria específica.

**Endpoint:**
```http
GET /public-api/bank-accounts/{bank_account_id}
```

**Respuesta exitosa (200 OK):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "account_holder": "Mi Empresa S.A.",
  "rif": "J-12345678-9",
  "account_type": "Corriente",
  "active": true,
  "created_at": "2026-02-03T10:30:00Z",
  "updated_at": "2026-02-03T10:30:00Z"
}
```

**Errores:**
- `404 Not Found`: Cuenta bancaria no encontrada

---

#### 7.4. Actualizar Cuenta Bancaria

Actualiza información de una cuenta bancaria existente.

**Endpoint:**
```http
PUT /public-api/bank-accounts/{bank_account_id}
```

**Body (todos los campos son opcionales):**
```json
{
  "bank": "Banco Provincial",
  "account_number": "01080123456789012345",
  "account_holder": "Nueva Razón Social C.A.",
  "rif": "J-87654321-0",
  "account_type": "Ahorro",
  "active": true
}
```

**Respuesta exitosa (200 OK):**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "bank": "Banco Provincial",
  "account_number": "01080123456789012345",
  "account_holder": "Nueva Razón Social C.A.",
  "rif": "J-87654321-0",
  "account_type": "Ahorro",
  "active": true,
  "created_at": "2026-02-03T10:30:00Z",
  "updated_at": "2026-02-03T12:00:00Z"
}
```

---

#### 7.5. Eliminar Cuenta Bancaria

Elimina (desactiva) una cuenta bancaria.

**Endpoint:**
```http
DELETE /public-api/bank-accounts/{bank_account_id}
```

**Query Parameters:**

| Parámetro | Tipo | Default | Descripción |
|-----------|------|---------|-------------|
| `hard_delete` | boolean | `false` | Eliminar permanentemente |

**Comportamiento:**

- **Soft Delete** (`hard_delete=false`): Marca la cuenta como `active=false` pero la preserva en la base de datos
- **Hard Delete** (`hard_delete=true`): Elimina permanentemente la cuenta y todos sus payment links asociados ⚠️

**Ejemplo:**
```bash
# Soft delete (recomendado)
curl -X DELETE https://points0.com/public-api/bank-accounts/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer bp_live_xxx"

# Hard delete (elimina permanentemente)
curl -X DELETE "https://points0.com/public-api/bank-accounts/550e8400-e29b-41d4-a716-446655440000?hard_delete=true" \
  -H "Authorization: Bearer bp_live_xxx"
```

**Respuesta exitosa (204 No Content):**
```
(Sin contenido)
```

---

## 💳 Direct Payments - Pagos Directos

Los **Direct Payments** (Pagos Directos) permiten a tus clientes capturar los datos del pagador **en su propia interfaz** y procesar el pago directamente sin necesidad de redireccionar a BuhoPago.

### ¿Cuándo usar Direct Payments?

✅ **Usa Direct Payments cuando:**
- Quieres mantener al usuario en TU interfaz durante todo el proceso
- Necesitas una respuesta inmediata del estado del pago
- Tu aplicación captura los datos bancarios del pagador
- Quieres control total de la experiencia de usuario

❌ **Usa Payment Links cuando:**
- Prefieres que BuhoPago maneje la interfaz de pago
- No quieres capturar datos bancarios en tu interfaz
- Quieres compartir un link de pago por email/WhatsApp

---

### 8.1. Generar OTP para Pago Directo

Inicia el proceso de pago directo enviando los datos del pagador y generando un OTP.

**Endpoint:**
```http
POST /public-api/direct-payment/generate-otp
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
Content-Type: application/json
```

**Body:**
```json
{
  "amount": 50.00,
  "currency": "USD",
  "description": "Pago de servicio premium",

  "payer_bank": "0105",
  "payer_phone": "04120246027",
  "payer_id_number": "30552028",
  "payer_email": "cliente@example.com",

  "reference_id": "ORDER-12345",
  "metadata": {
    "producto": "Plan Premium",
    "periodo": "Mensual"
  }
}
```

**Parámetros:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `amount` | float | ✅ | Monto a cobrar (> 0) |
| `currency` | string | ✅ | Moneda: `USD`, `EUR`, o `VES` |
| `description` | string | ❌ | Descripción del pago (máx 255 caracteres) |
| `payer_bank` | string | ✅ | Código del banco (ej: `0102`, `0134`) |
| `payer_phone` | string | ✅ | Teléfono del pagador (10-15 caracteres) |
| `payer_id_number` | string | ✅ | Cédula del pagador (ej: `30552028`, también acepta `V-30552028`) |
| `payer_email` | string | ❌ | Email del pagador |
| `reference_id` | string | ❌ | ID de referencia externa |
| `metadata` | object | ❌ | Datos adicionales personalizados |

**Respuesta exitosa (200 OK):**
```json
{
  "message": "OTP enviado exitosamente al teléfono del pagador",
  "transaction_id": 95,
  "amount_original": 50.00,
  "currency": "USD",
  "amount_bs": 1825.50,
  "exchange_rate": 36.51
}
```

**⚠️ Importante:**
- El OTP se envía por SMS al teléfono del pagador
- Guarda el `transaction_id` para el siguiente paso (es un número entero, no string)
- El `amount_bs` es el monto convertido a bolívares que se debitará

**📋 Formato de Datos:**

| Campo | Formato | Ejemplo Correcto | Ejemplo Incorrecto |
|-------|---------|------------------|-------------------|
| `payer_id_number` | Sin prefijo preferido | `"30552028"` ✅ | `"V-30552028"` ⚠️ |
| `payer_bank` | Código de 4 dígitos | `"0105"` ✅ | `"105"` ❌ |
| `payer_phone` | Con código de área | `"04120246027"` ✅ | `"4120246027"` ❌ |

**Códigos de Banco Comunes:**
- `0102` - Banco de Venezuela
- `0105` - Mercantil
- `0108` - Banco Provincial (BBVA)
- `0134` - Banesco
- `0191` - Banco Nacional de Crédito (BNC)

#### Ejemplo con VES (Bolívares)

Si prefieres trabajar directamente en VES sin conversión:

```json
{
  "amount": 3300.00,
  "currency": "VES",
  "description": "Pago de servicio premium",
  "payer_bank": "0105",
  "payer_phone": "04120246027",
  "payer_id_number": "30552028",
  "payer_email": "cliente@example.com"
}
```

**Respuesta:**
```json
{
  "message": "OTP enviado exitosamente al teléfono del pagador",
  "transaction_id": 96,
  "amount_original": 3300.00,
  "currency": "VES",
  "amount_bs": 3300.00,
  "exchange_rate": 1.0
}
```

✅ **Ventajas de usar VES:**
- No hay conversión de moneda (más rápido)
- El monto que indicas es exactamente el que se debita
- Ideal cuando trabajas directamente en bolívares

---

### 8.2. Verificar OTP y Procesar Pago

Verifica el OTP ingresado por el pagador y procesa el débito bancario.

**Endpoint:**
```http
POST /public-api/direct-payment/verify-otp
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
Content-Type: application/json
```

**Body:**
```json
{
  "otp_code": "123456",
  "transaction_id": 95,
  "phone": "04241234567",
  "amount": 1825.50
}
```

**Parámetros:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `otp_code` | string | ✅ | Código OTP ingresado por el pagador (4-10 caracteres) |
| `transaction_id` | integer | ✅ | ID de transacción del paso anterior |
| `phone` | string | ✅ | Teléfono del pagador (para validación) |
| `amount` | float | ✅ | Monto en bolívares a debitar |

**Respuesta Exitosa (200 OK):**
```json
{
  "status": "success",
  "message": "Pago procesado exitosamente",
  "transaction": {
    "id": "txn_abc123def456",
    "amount": 1825.50,
    "currency": "USD",
    "payer_email": "cliente@example.com",
    "payer_phone": "04241234567",
    "bank_reference": "REF-789456123",
    "completed_at": "2026-02-03T15:30:00Z"
  }
}
```

**Respuesta Rechazada (200 OK):**
```json
{
  "status": "failed",
  "message": "Pago rechazado por el banco",
  "error": "Fondos insuficientes",
  "transaction_id": "txn_abc123def456"
}
```

**Respuesta Pendiente (200 OK):**
```json
{
  "status": "pending",
  "message": "Pago en proceso de verificación",
  "transaction_id": "txn_abc123def456"
}
```

---

### 📝 Caso de Uso Completo: Tienda Online

**Escenario:** Una tienda online quiere procesar pagos sin que el usuario salga de su sitio web.

#### Paso 1: Interfaz de la Tienda (Frontend)

```html
<!-- Formulario en tu sitio web -->
<form id="payment-form">
  <h2>Procesar Pago - $50.00</h2>

  <label>Banco:</label>
  <select id="bank" required>
    <option value="0102">Banco de Venezuela</option>
    <option value="0105">Mercantil</option>
    <option value="0134">Banesco</option>
    <option value="0108">Banco Provincial (BBVA)</option>
  </select>

  <label>Cédula:</label>
  <input type="text" id="cedula" placeholder="30552028" required>

  <label>Teléfono:</label>
  <input type="tel" id="phone" placeholder="0424-1234567" required>

  <label>Email:</label>
  <input type="email" id="email" placeholder="tu@email.com">

  <button type="submit">Pagar $50.00</button>
</form>

<!-- Modal para ingresar OTP -->
<div id="otp-modal" style="display:none;">
  <h3>Ingrese el código OTP enviado a su teléfono</h3>
  <input type="text" id="otp-code" maxlength="6">
  <button onclick="verifyOTP()">Verificar</button>
</div>
```

#### Paso 2: Lógica del Backend (Python/Flask)

```python
from flask import Flask, request, jsonify
import requests

app = Flask(__name__)

API_KEY = 'bp_live_tu_api_key_aqui'
BASE_URL = 'https://points0.com/public-api'

headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}

# Variable global para almacenar transaction_id (en producción usa session/redis)
current_transaction = {}

@app.route('/api/iniciar-pago', methods=['POST'])
def iniciar_pago():
    """Paso 1: Generar OTP"""
    data = request.json

    payload = {
        'amount': 50.00,
        'currency': 'USD',
        'description': 'Compra en tienda online',
        'payer_bank': data['bank'],
        'payer_phone': data['phone'],
        'payer_id_number': data['cedula'],
        'payer_email': data.get('email'),
        'reference_id': f"ORDER-{data.get('order_id', '12345')}"
    }

    response = requests.post(
        f'{BASE_URL}/direct-payment/generate-otp',
        json=payload,
        headers=headers
    )

    if response.status_code == 200:
        result = response.json()

        # Guardar info de la transacción
        current_transaction['transaction_id'] = result['transaction_id']
        current_transaction['phone'] = data['phone']
        current_transaction['amount_bs'] = result['amount_bs']

        return jsonify({
            'success': True,
            'message': f'OTP enviado a {data["phone"]}',
            'amount_bs': result['amount_bs'],
            'exchange_rate': result['exchange_rate']
        })
    else:
        return jsonify({
            'success': False,
            'error': response.json()
        }), 400

@app.route('/api/verificar-otp', methods=['POST'])
def verificar_otp():
    """Paso 2: Verificar OTP y procesar pago"""
    data = request.json

    payload = {
        'otp_code': data['otp_code'],
        'transaction_id': current_transaction['transaction_id'],
        'phone': current_transaction['phone'],
        'amount': current_transaction['amount_bs']
    }

    response = requests.post(
        f'{BASE_URL}/direct-payment/verify-otp',
        json=payload,
        headers=headers
    )

    result = response.json()

    if result['status'] == 'success':
        # Pago exitoso - Actualizar tu base de datos
        return jsonify({
            'success': True,
            'message': 'Pago procesado exitosamente',
            'transaction': result['transaction']
        })
    elif result['status'] == 'failed':
        return jsonify({
            'success': False,
            'message': result['message'],
            'error': result.get('error')
        })
    else:
        return jsonify({
            'success': False,
            'message': 'Pago en proceso, intente nuevamente'
        })

if __name__ == '__main__':
    app.run(debug=True)
```

#### Paso 3: JavaScript del Frontend

```javascript
// Paso 1: Enviar datos y generar OTP
document.getElementById('payment-form').addEventListener('submit', async (e) => {
  e.preventDefault();

  const data = {
    bank: document.getElementById('bank').value,
    cedula: document.getElementById('cedula').value,
    phone: document.getElementById('phone').value,
    email: document.getElementById('email').value,
    order_id: '12345' // ID de tu orden
  };

  const response = await fetch('/api/iniciar-pago', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });

  const result = await response.json();

  if (result.success) {
    alert(`OTP enviado. Monto a debitar: Bs ${result.amount_bs}`);

    // Mostrar modal para ingresar OTP
    document.getElementById('otp-modal').style.display = 'block';
  } else {
    alert('Error: ' + result.error);
  }
});

// Paso 2: Verificar OTP
async function verifyOTP() {
  const otp_code = document.getElementById('otp-code').value;

  const response = await fetch('/api/verificar-otp', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ otp_code })
  });

  const result = await response.json();

  if (result.success) {
    alert('✅ Pago exitoso! Referencia: ' + result.transaction.bank_reference);
    window.location.href = '/success';
  } else {
    alert('❌ ' + result.message);
  }
}
```

---

### 🔄 Diagrama de Flujo

```
┌─────────────────────────────────────────────────────┐
│  1. USUARIO INGRESA DATOS EN TU INTERFAZ             │
│     - Monto: $50                                    │
│     - Banco, Cédula, Teléfono                       │
└─────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│  2. TU BACKEND → BuhoPago API                       │
│     POST /direct-payment/generate-otp               │
└─────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│  3. BUHOPAGO ENVÍA SMS CON OTP                      │
│     SMS → "Tu código OTP es: 123456"                │
└─────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│  4. USUARIO INGRESA OTP EN TU INTERFAZ              │
│     Input: "123456"                                 │
└─────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│  5. TU BACKEND → BuhoPago API                       │
│     POST /direct-payment/verify-otp                 │
└─────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│  6. BUHOPAGO PROCESA DÉBITO BANCARIO                │
│     Conexión con banco → Débito inmediato           │
└─────────────────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│  7. TU INTERFAZ MUESTRA RESULTADO                   │
│     ✅ "Pago exitoso"  o  ❌ "Pago rechazado"       │
└─────────────────────────────────────────────────────┘
```

---

### ⚠️ Consideraciones Importantes

1. **Seguridad:**
   - NUNCA expongas tu API Key en el frontend
   - Todas las llamadas a BuhoPago deben hacerse desde tu backend
   - Valida los datos del pagador antes de enviarlos

2. **Experiencia de Usuario:**
   - Muestra el monto en bolívares después del primer paso
   - Indica claramente que se enviará un SMS
   - Maneja todos los estados posibles (success, failed, pending)

3. **Manejo de Errores:**
   - OTP inválido → Permitir reintentar
   - Timeout → Ofrecer generar nuevo OTP
   - Fondos insuficientes → Mostrar mensaje claro

4. **Testing:**
   - Usa API keys de prueba (`bp_test_xxx`) en desarrollo
   - Prueba con diferentes bancos y montos
   - Verifica el manejo de errores

---

## Modelos de Datos

### Payment Link

```typescript
interface PaymentLink {
  id: string;                    // UUID único
  slug: string;                  // Identificador corto para la URL
  url: string;                   // URL completa del payment link
  amount: number;                // Monto del pago
  currency: string;              // USD, EUR, o VES
  description: string;           // Descripción del pago
  status: string;                // "active", "expired", "used"
  qr_url?: string;               // URL del código QR
  reference_id?: string;         // ID de referencia externa
  metadata?: object;             // Datos personalizados
  created_at: string;            // ISO 8601 timestamp
  expires_at?: string;           // ISO 8601 timestamp
  payment_count: number;         // Número de pagos recibidos
  total_collected: number;       // Total recaudado
}
```

### Transaction

```typescript
interface Transaction {
  id: string;                    // ID de la transacción
  payment_link_slug: string;     // Slug del payment link
  amount: number;                // Monto pagado
  status: string;                // "pending", "completed", "failed"
  payer_email?: string;          // Email del pagador
  completed_at?: string;         // ISO 8601 timestamp
  bank_reference?: string;       // Referencia bancaria
}
```

### Bank Account

```typescript
interface BankAccount {
  id: string;                    // UUID único
  user_id: string;               // ID del usuario propietario
  bank: string;                  // Nombre del banco
  account_number: string;        // Número de cuenta
  account_holder: string;        // Titular de la cuenta
  rif: string;                   // RIF del titular
  account_type?: string;         // Tipo de cuenta: "Corriente", "Ahorro", etc.
  active: boolean;               // Estado de la cuenta
  created_at: string;            // ISO 8601 timestamp
  updated_at: string;            // ISO 8601 timestamp
}
```

---

## Códigos de Respuesta

### Códigos de Éxito

| Código | Descripción |
|--------|-------------|
| `200 OK` | Petición exitosa |
| `201 Created` | Recurso creado exitosamente |
| `204 No Content` | Operación exitosa sin contenido de respuesta |

### Códigos de Error

| Código | Descripción | Solución |
|--------|-------------|----------|
| `400 Bad Request` | Datos inválidos en la petición | Verifica los parámetros enviados |
| `401 Unauthorized` | API key inválida o faltante | Verifica tu API key |
| `403 Forbidden` | No tienes permiso para esta acción | Verifica que tengas cuenta bancaria activa |
| `404 Not Found` | Recurso no encontrado | Verifica el ID del recurso |
| `500 Internal Server Error` | Error del servidor | Contacta soporte |

### Formato de Error

```json
{
  "detail": "Descripción del error",
  "error": "codigo_error",
  "message": "Mensaje más detallado",
  "action": "accion_sugerida"
}
```

**Ejemplo 1 - Falta cuenta bancaria:**
```json
{
  "detail": "No tienes ninguna cuenta bancaria activa. Configura una cuenta antes de crear payment links."
}
```

**Ejemplo 2 - Error de validación:**
```json
{
  "detail": [
    {
      "loc": ["body", "amount"],
      "msg": "ensure this value is greater than 0",
      "type": "value_error.number.not_gt"
    }
  ]
}
```

---

## Webhooks

Los webhooks te permiten recibir notificaciones automáticas cuando ocurren eventos importantes (como un pago completado).

### Configuración

Incluye el parámetro `webhook_url` al crear un payment link:

```json
{
  "amount": 50.00,
  "currency": "USD",
  "description": "Mi producto",
  "webhook_url": "https://tusitio.com/webhooks/buhopago"
}
```

### Payload del Webhook

Cuando un pago se complete, recibirás un POST request a tu `webhook_url`:

```json
{
  "event": "payment.completed",
  "timestamp": "2024-01-15T11:45:00Z",
  "data": {
    "payment_link_id": "550e8400-e29b-41d4-a716-446655440000",
    "payment_link_slug": "pay_abc123def",
    "transaction_id": "txn_abc123",
    "amount": 50.00,
    "currency": "USD",
    "status": "completed",
    "payer_email": "cliente@example.com",
    "reference_id": "order_12345",
    "metadata": {
      "customer_id": "cus_abc123"
    },
    "completed_at": "2024-01-15T11:45:00Z"
  }
}
```

### Verificar Webhook Endpoint

Puedes verificar que tu webhook endpoint esté accesible:

```http
GET /public-api/webhooks/verify
```

### Mejores Prácticas para Webhooks

1. **Responde rápido**: Tu endpoint debe responder con `200 OK` en menos de 5 segundos
2. **Procesa asíncronamente**: Guarda los datos y procésalos en background
3. **Valida los datos**: Verifica que el `payment_link_id` exista en tu base de datos
4. **Maneja duplicados**: Un webhook puede enviarse más de una vez (idempotencia)
5. **Usa HTTPS**: Tu webhook URL debe usar HTTPS en producción

**Ejemplo de handler (Node.js/Express):**
```javascript
app.post('/webhooks/buhopago', async (req, res) => {
  // Responde rápido
  res.status(200).send('OK');

  // Procesa asíncronamente
  const { event, data } = req.body;

  if (event === 'payment.completed') {
    // Actualiza tu base de datos
    await processPayment(data);
  }
});
```

---

## Ejemplos de Integración

### JavaScript/Node.js

```javascript
const axios = require('axios');

const BUHOPAGO_API_KEY = process.env.BUHOPAGO_API_KEY;
const BASE_URL = 'https://points0.com/public-api';

// Verificar y crear cuenta bancaria si es necesario
async function ensureBankAccount() {
  try {
    // Verificar si ya existe una cuenta bancaria
    const response = await axios.get(
      `${BASE_URL}/bank-accounts`,
      {
        headers: {
          'Authorization': `Bearer ${BUHOPAGO_API_KEY}`
        }
      }
    );

    if (response.data.length > 0) {
      console.log('✅ Cuenta bancaria ya existe');
      return response.data[0];
    }

    // Crear cuenta bancaria si no existe
    const newAccount = await axios.post(
      `${BASE_URL}/bank-accounts`,
      {
        bank: 'Banco de Venezuela',
        account_number: '01020123456789012345',
        account_holder: 'Mi Empresa S.A.',
        rif: 'J-12345678-9',
        account_type: 'Corriente'
      },
      {
        headers: {
          'Authorization': `Bearer ${BUHOPAGO_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );

    console.log('✅ Cuenta bancaria creada');
    return newAccount.data;

  } catch (error) {
    console.error('Error con cuenta bancaria:', error.response?.data);
    throw error;
  }
}

// Crear un payment link
async function createPaymentLink(orderData) {
  try {
    const response = await axios.post(
      `${BASE_URL}/payment-links`,
      {
        amount: orderData.total,
        currency: 'USD',
        description: `Orden #${orderData.id}`,
        reference_id: orderData.id,
        webhook_url: 'https://tusitio.com/webhooks/buhopago',
        return_url: `https://tusitio.com/orders/${orderData.id}/success`,
        metadata: {
          customer_id: orderData.customerId,
          items: orderData.items
        }
      },
      {
        headers: {
          'Authorization': `Bearer ${BUHOPAGO_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );

    console.log('Payment link creado:', response.data.url);
    return response.data;

  } catch (error) {
    console.error('Error:', error.response.data);
    throw error;
  }
}

// Consultar estado de un payment link
async function checkPaymentStatus(linkId) {
  try {
    const response = await axios.get(
      `${BASE_URL}/payment-links/${linkId}`,
      {
        headers: {
          'Authorization': `Bearer ${BUHOPAGO_API_KEY}`
        }
      }
    );

    return response.data;

  } catch (error) {
    console.error('Error:', error.response.data);
    throw error;
  }
}

// Uso
async function processOrder() {
  // 1. Asegurar que existe una cuenta bancaria
  await ensureBankAccount();

  // 2. Crear el payment link
  const order = {
    id: '12345',
    total: 99.99,
    customerId: 'cus_abc123',
    items: ['item1', 'item2']
  };

  const paymentLink = await createPaymentLink(order);
  console.log('Redirige al usuario a:', paymentLink.url);
}

processOrder().catch(console.error);
```

### Python

```python
import requests
import os

BUHOPAGO_API_KEY = os.getenv('BUHOPAGO_API_KEY')
BASE_URL = 'https://points0.com/public-api'

def ensure_bank_account():
    """Verifica y crea cuenta bancaria si es necesario"""

    headers = {
        'Authorization': f'Bearer {BUHOPAGO_API_KEY}'
    }

    # Verificar si ya existe una cuenta bancaria
    response = requests.get(f'{BASE_URL}/bank-accounts', headers=headers)
    response.raise_for_status()

    accounts = response.json()

    if len(accounts) > 0:
        print('✅ Cuenta bancaria ya existe')
        return accounts[0]

    # Crear cuenta bancaria si no existe
    bank_data = {
        'bank': 'Banco de Venezuela',
        'account_number': '01020123456789012345',
        'account_holder': 'Mi Empresa S.A.',
        'rif': 'J-12345678-9',
        'account_type': 'Corriente'
    }

    headers['Content-Type'] = 'application/json'
    response = requests.post(
        f'{BASE_URL}/bank-accounts',
        json=bank_data,
        headers=headers
    )
    response.raise_for_status()

    print('✅ Cuenta bancaria creada')
    return response.json()

def create_payment_link(order_data):
    """Crea un payment link para una orden"""

    headers = {
        'Authorization': f'Bearer {BUHOPAGO_API_KEY}',
        'Content-Type': 'application/json'
    }

    payload = {
        'amount': order_data['total'],
        'currency': 'USD',
        'description': f"Orden #{order_data['id']}",
        'reference_id': order_data['id'],
        'webhook_url': 'https://tusitio.com/webhooks/buhopago',
        'return_url': f"https://tusitio.com/orders/{order_data['id']}/success",
        'metadata': {
            'customer_id': order_data['customer_id'],
            'items': order_data['items']
        }
    }

    response = requests.post(
        f'{BASE_URL}/payment-links',
        json=payload,
        headers=headers
    )

    response.raise_for_status()
    return response.json()

def check_payment_status(link_id):
    """Consulta el estado de un payment link"""

    headers = {
        'Authorization': f'Bearer {BUHOPAGO_API_KEY}'
    }

    response = requests.get(
        f'{BASE_URL}/payment-links/{link_id}',
        headers=headers
    )

    response.raise_for_status()
    return response.json()

# Uso
if __name__ == '__main__':
    # 1. Asegurar que existe una cuenta bancaria
    ensure_bank_account()

    # 2. Crear el payment link
    order = {
        'id': '12345',
        'total': 99.99,
        'customer_id': 'cus_abc123',
        'items': ['item1', 'item2']
    }

    payment_link = create_payment_link(order)
    print(f"Redirige al usuario a: {payment_link['url']}")
```

### PHP

```php
<?php

class BuhoPagoClient {
    private $apiKey;
    private $baseUrl = 'https://api.buhopago.com/public-api';

    public function __construct($apiKey) {
        $this->apiKey = $apiKey;
    }

    public function createPaymentLink($data) {
        $ch = curl_init($this->baseUrl . '/payment-links');

        curl_setopt_array($ch, [
            CURLOPT_POST => true,
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => [
                'Authorization: Bearer ' . $this->apiKey,
                'Content-Type: application/json'
            ],
            CURLOPT_POSTFIELDS => json_encode($data)
        ]);

        $response = curl_exec($ch);
        $httpCode = curl_getinfo($ch, CURLINFO_HTTP_CODE);
        curl_close($ch);

        if ($httpCode !== 201) {
            throw new Exception('Error creating payment link: ' . $response);
        }

        return json_decode($response, true);
    }

    public function getPaymentLink($linkId) {
        $ch = curl_init($this->baseUrl . '/payment-links/' . $linkId);

        curl_setopt_array($ch, [
            CURLOPT_RETURNTRANSFER => true,
            CURLOPT_HTTPHEADER => [
                'Authorization: Bearer ' . $this->apiKey
            ]
        ]);

        $response = curl_exec($ch);
        curl_close($ch);

        return json_decode($response, true);
    }
}

// Uso
$client = new BuhoPagoClient(getenv('BUHOPAGO_API_KEY'));

$paymentLink = $client->createPaymentLink([
    'amount' => 99.99,
    'currency' => 'USD',
    'description' => 'Orden #12345',
    'reference_id' => '12345',
    'webhook_url' => 'https://tusitio.com/webhooks/buhopago',
    'metadata' => [
        'customer_id' => 'cus_abc123'
    ]
]);

echo "Redirige al usuario a: " . $paymentLink['url'];
?>
```

### cURL

```bash
# 1. Verificar si tienes cuentas bancarias
curl -X GET https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_xxx"

# 2. Crear cuenta bancaria (si no tienes ninguna)
curl -X POST https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Mi Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente"
  }'

# 3. Crear payment link
curl -X POST https://points0.com/public-api/payment-links \
  -H "Authorization: Bearer bp_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "currency": "USD",
    "description": "Producto de prueba",
    "reference_id": "order_12345",
    "webhook_url": "https://tusitio.com/webhooks/buhopago"
  }'

# Consultar payment link
curl -X GET https://points0.com/public-api/payment-links/{link_id} \
  -H "Authorization: Bearer bp_live_xxx"

# Listar payment links
curl -X GET "https://points0.com/public-api/payment-links?status=active&page=1" \
  -H "Authorization: Bearer bp_live_xxx"

# Actualizar cuenta bancaria
curl -X PUT https://points0.com/public-api/bank-accounts/{account_id} \
  -H "Authorization: Bearer bp_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "account_type": "Ahorro"
  }'

# Eliminar cuenta bancaria (soft delete)
curl -X DELETE https://points0.com/public-api/bank-accounts/{account_id} \
  -H "Authorization: Bearer bp_live_xxx"
```

---

## Mejores Prácticas

### Seguridad

1. **Nunca expongas tu API key**
   - Guárdala en variables de entorno
   - No la commits en Git
   - Usa servicios de gestión de secretos (AWS Secrets Manager, HashiCorp Vault, etc.)

2. **Usa HTTPS siempre**
   - Todas las peticiones deben usar HTTPS
   - Especialmente importante para webhooks

3. **Valida los webhooks**
   - Verifica que los datos recibidos correspondan a payment links que creaste
   - Implementa idempotencia para manejar duplicados

### Performance

1. **Cachea cuando sea apropiado**
   - No necesitas consultar el estado de cada payment link en cada request
   - Usa los webhooks para recibir actualizaciones en tiempo real

2. **Maneja errores apropiadamente**
   - Implementa retry logic con exponential backoff
   - Loggea los errores para debugging

3. **Usa paginación**
   - Al listar payment links, usa `page_size` adecuado
   - No intentes cargar miles de links en una sola request

### Flujo Recomendado

```
1. [SETUP INICIAL] Verificar que tienes al menos una cuenta bancaria
   ↓
2. Usuario inicia checkout en tu aplicación
   ↓
3. Tu backend crea un payment link vía API
   ↓
4. Rediriges al usuario a la URL del payment link
   ↓
5. Usuario completa el pago en BuhoPago
   ↓
6. BuhoPago envía webhook a tu servidor
   ↓
7. Tu servidor actualiza el estado de la orden
   ↓
8. Usuario es redirigido a tu return_url
   ↓
9. Muestras página de confirmación
```

**Setup Inicial (solo una vez):**

```bash
# Verificar cuentas bancarias
curl https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_xxx"

# Si el array está vacío, crear una cuenta
curl -X POST https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Mi Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente"
  }'
```

### Testing

1. **Usa API keys de test**
   - Crea API keys con `environment: "test"` para desarrollo
   - Los payment links de test no procesarán pagos reales

2. **Configura cuenta bancaria de prueba**
   - Crea al menos una cuenta bancaria antes de probar payment links
   - Usa datos ficticios para testing (ej: RIF J-00000000-0)

3. **Verifica tu webhook endpoint**
   - Usa herramientas como ngrok para exponer localhost
   - Prueba con diferentes escenarios (éxito, fallo, timeout)

4. **Maneja edge cases**
   - Payment link expirado
   - Usuario cancela el pago
   - Pago duplicado
   - Network timeouts
   - Falta de cuenta bancaria
   - Cuenta bancaria desactivada

5. **Script de pruebas incluido**
   - Usa el script `pruebaApi.py` incluido en el repositorio
   - Ejecuta tests completos de todos los endpoints
   ```bash
   cd backend-buho
   python pruebaApi.py
   ```

---

## Limitaciones y Cuotas

### Rate Limits

- **Máximo 100 requests por minuto** por API key
- **Máximo 10,000 requests por día** por API key

Si excedes estos límites, recibirás un error `429 Too Many Requests`.

### Límites de API Keys

- **Máximo 10 API keys activas** por usuario
- Las API keys pueden expirar si se configura `expires_in_days`

### Límites de Payment Links

- **Expiración máxima**: 8760 horas (1 año)
- **Descripción**: Entre 3 y 200 caracteres
- **Metadata**: Máximo 10KB por payment link
- **Requisito**: Al menos 1 cuenta bancaria activa

### Límites de Bank Accounts

- **Máximo cuentas activas**: Sin límite
- **Campos requeridos**: bank, account_number, account_holder, rif
- **Número de cuenta**: Máximo 20 caracteres
- **RIF**: Máximo 20 caracteres (formato: J-12345678-9)
- **Unicidad**: No se permiten cuentas duplicadas por usuario

### Monedas Soportadas

BuhoPago soporta las siguientes monedas para payment links y pagos directos:

- **USD** (Dólar estadounidense)
- **EUR** (Euro)
- **VES** (Bolívar venezolano)

#### Conversión Automática a VES

Todos los pagos se procesan en **Bolívares (VES)**, que es la moneda nacional de Venezuela. Si creas un payment link o pago directo en USD o EUR:

1. **Conversión automática**: El monto se convierte a VES usando la tasa de cambio actual
2. **Tasa guardada**: La tasa de cambio se guarda en la transacción para auditoría
3. **Transparencia**: El pagador ve ambos montos (original y en VES)

**Ejemplo de conversión:**
```json
// Request: Crear payment link de $50 USD
{
  "amount": 50.00,
  "currency": "USD",
  "description": "Suscripción mensual"
}

// Al momento del pago, el sistema:
// 1. Obtiene tasa actual: 1 USD = 66.00 VES
// 2. Calcula: 50 * 66.00 = 3,300.00 VES
// 3. Guarda ambos valores en la transacción

// Transacción guardada:
{
  "amount": 3300.00,              // Monto procesado en VES
  "amount_original": 50.00,       // Monto original
  "currency_original": "USD",     // Moneda original
  "exchange_rate": 66.00          // Tasa usada
}
```

#### Pagos Directos en VES

Si creas un pago directamente en VES:
- **No hay conversión**: El monto se procesa tal cual
- **Tasa de cambio**: Se guarda como 1.0 (sin conversión)
- **Procesamiento directo**: Más rápido y sin cálculos adicionales

```json
{
  "amount": 3300.00,
  "currency": "VES",
  "description": "Pago en bolívares"
}
// No se aplica conversión, se procesa directamente
```

#### Tracking de Tasas de Cambio

Todas las transacciones guardan información completa sobre la conversión:
- Moneda original del pago
- Monto original antes de conversión
- Tasa de cambio aplicada
- Monto final en VES procesado

Esto te permite:
- ✅ Auditoría completa de tasas aplicadas
- ✅ Reconciliación de pagos multi-moneda
- ✅ Reportes detallados por moneda original
- ✅ Trazabilidad de conversiones históricas

---

## Soporte

### Documentación Adicional

- **API Reference**: https://docs.buhopago.com/api
- **Changelog**: https://docs.buhopago.com/changelog
- **Status Page**: https://status.buhopago.com

### Contacto

- **Email**: api-support@buhopago.com
- **Discord**: https://discord.gg/buhopago
- **GitHub Issues**: https://github.com/buhopago/api-examples

### Reportar Bugs

Si encuentras un bug o problema:

1. Verifica que estés usando la última versión de la API
2. Revisa la documentación y ejemplos
3. Contacta a soporte con:
   - API key prefix (solo los primeros caracteres)
   - Request ID (si está disponible)
   - Logs de error completos
   - Pasos para reproducir el problema

---

## Changelog

### Versión 1.1.1 (2026-02-04)

- 🔧 **Correcciones Técnicas**:
  - **BREAKING CHANGE**: `transaction_id` ahora es `integer` en lugar de `string` para Direct Payments
    - Endpoint afectado: `POST /direct-payment/generate-otp` (respuesta)
    - Endpoint afectado: `POST /direct-payment/verify-otp` (request)
    - **Acción requerida**: Actualiza tu código para manejar `transaction_id` como número entero
  - Corrección en el servicio de OTP para soporte multi-tenant mejorado
  - Optimización de imports y dependencias internas
- 📝 **Formato de Cédula**:
  - Ahora acepta formato sin prefijo (ej: `"30552028"`) además del formato con prefijo (ej: `"V-30552028"`)
  - Se recomienda enviar la cédula sin prefijo para evitar problemas de formato
- 🏦 **Códigos de Banco**:
  - Asegúrate de usar el código correcto de 4 dígitos (ej: `"0105"` para Mercantil, `"0102"` para Banco de Venezuela)
- ✅ **Estabilidad**:
  - Mejoras en la confiabilidad del envío de OTP
  - Mejor manejo de errores y validaciones

**Migración de `transaction_id`:**

```javascript
// ❌ Antes (v1.1.0)
const response = await fetch('/api/verificar-otp', {
  body: JSON.stringify({
    otp_code: "123456",
    transaction_id: "95"  // ❌ String
  })
});

// ✅ Ahora (v1.1.1)
const response = await fetch('/api/verificar-otp', {
  body: JSON.stringify({
    otp_code: "123456",
    transaction_id: 95  // ✅ Number
  })
});
```

### Versión 1.1.0 (2026-02-03)

- ✨ **Nuevos endpoints de Bank Accounts**
  - POST `/bank-accounts` - Crear cuenta bancaria
  - GET `/bank-accounts` - Listar cuentas bancarias
  - GET `/bank-accounts/{id}` - Obtener cuenta específica
  - PUT `/bank-accounts/{id}` - Actualizar cuenta bancaria
  - DELETE `/bank-accounts/{id}` - Eliminar cuenta bancaria (soft/hard delete)
- 💳 **Nuevos endpoints de Direct Payments**
  - POST `/direct-payment/generate-otp` - Generar OTP para pago directo
  - POST `/direct-payment/verify-otp` - Verificar OTP y procesar pago
- 🎯 **Caso de uso completo**: Ejemplo de tienda online con captura de datos en interfaz propia del merchant
- 📋 **Requisito obligatorio**: Al menos una cuenta bancaria activa para crear payment links
- ⚡ **Optimización**: QR codes deshabilitados para links creados vía API (reducción de costos)
- 📝 Script de pruebas completo (`pruebaApi.py`) con 22 tests
- 📚 Documentación ampliada con ejemplos de integración en HTML, JavaScript y Python Flask

### Versión 1.0.0 (2024-01-15)

- Lanzamiento inicial de la API pública
- Endpoints para crear, consultar, listar y desactivar payment links
- Soporte para webhooks
- Autenticación con API keys
- Soporte para USD, EUR y VES

---

**¿Listo para empezar?** Obtén tu API key y empieza a integrar pagos en minutos. 🚀
