# 🏦 Bank Accounts API - Documentación

## Descripción

Los endpoints de Bank Accounts permiten a los merchants gestionar sus cuentas bancarias a través de la API pública de BuhoPago. **Debes tener al menos una cuenta bancaria configurada para poder crear payment links.**

## Autenticación

Todos los endpoints requieren autenticación mediante API Key en el header:

```http
Authorization: Bearer bp_live_your_api_key_here
```

## Base URL

```
https://points0.com/public-api
```

---

## Endpoints

### 1. Crear Cuenta Bancaria

Crea una nueva cuenta bancaria para recibir pagos.

**Endpoint:** `POST /public-api/bank-accounts`

**Request Body:**

```json
{
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "account_holder": "Merchant Test S.A.",
  "rif": "J-12345678-9",
  "account_type": "Corriente"
}
```

**Campos:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `bank` | string | Sí | Nombre del banco (máx 100 caracteres) |
| `account_number` | string | Sí | Número de cuenta (máx 20 caracteres) |
| `account_holder` | string | Sí | Titular de la cuenta (máx 200 caracteres) |
| `rif` | string | Sí | RIF del titular (máx 20 caracteres) |
| `account_type` | string | No | Tipo de cuenta: "Corriente", "Ahorro", etc. |

**Response (201 Created):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "account_holder": "Merchant Test S.A.",
  "rif": "J-12345678-9",
  "account_type": "Corriente",
  "active": true,
  "created_at": "2026-02-03T10:30:00Z",
  "updated_at": "2026-02-03T10:30:00Z"
}
```

**Ejemplo con cURL:**

```bash
curl -X POST https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Merchant Test S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente"
  }'
```

**Ejemplo con Python:**

```python
import requests

API_KEY = 'bp_live_your_api_key'
BASE_URL = 'https://points0.com/public-api'

headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}

payload = {
    'bank': 'Banco de Venezuela',
    'account_number': '01020123456789012345',
    'account_holder': 'Merchant Test S.A.',
    'rif': 'J-12345678-9',
    'account_type': 'Corriente'
}

response = requests.post(
    f'{BASE_URL}/bank-accounts',
    json=payload,
    headers=headers
)

print(response.json())
```

---

### 2. Listar Cuentas Bancarias

Obtiene todas las cuentas bancarias configuradas.

**Endpoint:** `GET /public-api/bank-accounts`

**Query Parameters:**

| Parámetro | Tipo | Requerido | Default | Descripción |
|-----------|------|-----------|---------|-------------|
| `active_only` | boolean | No | `true` | Mostrar solo cuentas activas |

**Response (200 OK):**

```json
[
  {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "user_id": "123e4567-e89b-12d3-a456-426614174000",
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Merchant Test S.A.",
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
    "account_holder": "Merchant Test S.A.",
    "rif": "J-12345678-9",
    "account_type": "Ahorro",
    "active": true,
    "created_at": "2026-02-03T11:00:00Z",
    "updated_at": "2026-02-03T11:00:00Z"
  }
]
```

**Ejemplo con cURL:**

```bash
# Listar solo cuentas activas (default)
curl https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_your_api_key"

# Listar todas las cuentas (incluidas inactivas)
curl "https://points0.com/public-api/bank-accounts?active_only=false" \
  -H "Authorization: Bearer bp_live_your_api_key"
```

---

### 3. Obtener Cuenta Específica

Consulta información de una cuenta bancaria específica.

**Endpoint:** `GET /public-api/bank-accounts/{bank_account_id}`

**Response (200 OK):**

```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "user_id": "123e4567-e89b-12d3-a456-426614174000",
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "account_holder": "Merchant Test S.A.",
  "rif": "J-12345678-9",
  "account_type": "Corriente",
  "active": true,
  "created_at": "2026-02-03T10:30:00Z",
  "updated_at": "2026-02-03T10:30:00Z"
}
```

**Errores:**

- `404 Not Found`: Cuenta bancaria no encontrada
- `401 Unauthorized`: API Key inválida

**Ejemplo con cURL:**

```bash
curl https://points0.com/public-api/bank-accounts/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer bp_live_your_api_key"
```

---

### 4. Actualizar Cuenta Bancaria

Actualiza información de una cuenta bancaria existente.

**Endpoint:** `PUT /public-api/bank-accounts/{bank_account_id}`

**Request Body (todos los campos son opcionales):**

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

**Response (200 OK):**

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

**Ejemplo con cURL:**

```bash
curl -X PUT https://points0.com/public-api/bank-accounts/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer bp_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "account_type": "Ahorro"
  }'
```

---

### 5. Eliminar Cuenta Bancaria

Elimina (desactiva) una cuenta bancaria.

**Endpoint:** `DELETE /public-api/bank-accounts/{bank_account_id}`

**Query Parameters:**

| Parámetro | Tipo | Requerido | Default | Descripción |
|-----------|------|-----------|---------|-------------|
| `hard_delete` | boolean | No | `false` | Si es `true`, elimina permanentemente. Si es `false`, solo desactiva (soft delete) |

**Response (204 No Content):** No retorna contenido.

**Comportamiento:**

- **Soft Delete** (`hard_delete=false`): La cuenta se marca como `active=false` pero se preserva en la base de datos. Los payment links asociados permanecen intactos.
- **Hard Delete** (`hard_delete=true`): La cuenta se elimina permanentemente junto con todos sus payment links asociados. **⚠️ Acción irreversible.**

**Ejemplo con cURL:**

```bash
# Soft delete (recomendado)
curl -X DELETE https://points0.com/public-api/bank-accounts/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: Bearer bp_live_your_api_key"

# Hard delete (elimina permanentemente)
curl -X DELETE "https://points0.com/public-api/bank-accounts/550e8400-e29b-41d4-a716-446655440000?hard_delete=true" \
  -H "Authorization: Bearer bp_live_your_api_key"
```

---

## Flujo de Uso Típico

### 1. Verificar si tienes cuentas bancarias

```bash
curl https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_your_api_key"
```

Si la respuesta es `[]` (array vacío), necesitas crear una cuenta bancaria.

### 2. Crear tu primera cuenta bancaria

```bash
curl -X POST https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Mi Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente"
  }'
```

### 3. Crear un payment link

Ahora puedes crear payment links usando `/public-api/payment-links`. El sistema automáticamente usará tu cuenta bancaria.

```bash
curl -X POST https://points0.com/public-api/payment-links \
  -H "Authorization: Bearer bp_live_your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "currency": "USD",
    "description": "Pago de prueba"
  }'
```

---

## Códigos de Error

| Código | Descripción |
|--------|-------------|
| `201` | Creado exitosamente |
| `200` | OK |
| `204` | No Content (eliminación exitosa) |
| `400` | Bad Request (datos inválidos) |
| `401` | Unauthorized (API Key inválida) |
| `404` | Not Found (cuenta no encontrada) |
| `500` | Internal Server Error |

---

## Notas Importantes

1. **Cuenta Bancaria Requerida**: Debes tener al menos una cuenta bancaria activa para crear payment links.
2. **Primera Cuenta**: Cuando creas payment links vía API, el sistema usa automáticamente la primera cuenta bancaria creada (la más antigua).
3. **Unicidad**: No puedes tener dos cuentas con el mismo número de cuenta para el mismo usuario.
4. **Soft Delete**: Por defecto, las eliminaciones son "soft delete" para preservar el historial.
5. **RIF**: El RIF debe incluir el formato completo (ej: `J-12345678-9`).

---

## Ejemplo Completo en Python

```python
import requests
import time

API_KEY = 'bp_live_your_api_key'
BASE_URL = 'https://points0.com/public-api'

headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}

# 1. Verificar cuentas existentes
response = requests.get(f'{BASE_URL}/bank-accounts', headers=headers)
accounts = response.json()

if len(accounts) == 0:
    print("No hay cuentas bancarias. Creando una...")

    # 2. Crear cuenta bancaria
    bank_data = {
        'bank': 'Banco de Venezuela',
        'account_number': '01020123456789012345',
        'account_holder': 'Mi Empresa S.A.',
        'rif': 'J-12345678-9',
        'account_type': 'Corriente'
    }

    response = requests.post(
        f'{BASE_URL}/bank-accounts',
        json=bank_data,
        headers=headers
    )

    if response.status_code == 201:
        print("✅ Cuenta bancaria creada exitosamente")
        account = response.json()
        print(f"   Bank: {account['bank']}")
        print(f"   Account: {account['account_number']}")
    else:
        print(f"❌ Error: {response.json()}")
        exit(1)

time.sleep(0.5)

# 3. Crear payment link
link_data = {
    'amount': 99.99,
    'currency': 'USD',
    'description': 'Pago de Producto Premium',
    'webhook_url': 'https://mi-sitio.com/webhook',
    'return_url': 'https://mi-sitio.com/success',
    'reference_id': 'ORDER-12345'
}

response = requests.post(
    f'{BASE_URL}/payment-links',
    json=link_data,
    headers=headers
)

if response.status_code == 201:
    link = response.json()
    print("✅ Payment link creado exitosamente")
    print(f"   URL: {link['url']}")
    print(f"   Slug: {link['slug']}")
else:
    print(f"❌ Error: {response.json()}")
```

---

## Soporte

Para más información o reportar problemas:
- Email: soporte@buhopago.com
- Documentación completa: https://docs.buhopago.com
