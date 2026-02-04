# 💳 Credits API - Créditos Inmediatos

## Introducción

La API de Créditos Inmediatos permite a usuarios empresariales con permisos especiales retirar fondos procesados a través de la plataforma. Este sistema está diseñado para negocios que:

- Reciben transacciones de terceros
- Necesitan convertir fondos a stablecoins u otras divisas
- Requieren liquidez inmediata basada en su volumen procesado

## ⚠️ Requisitos Previos

Para usar esta API necesitas:

1. **API Key con permisos especiales**: Tu API key debe tener el scope `credits:execute`
2. **KYC Aprobado**: Tu cuenta debe estar verificada
3. **Volumen procesado**: Solo puedes retirar fondos hasta el monto que hayas procesado en transacciones

> 📧 **¿Cómo obtener acceso?** Contacta a soporte en support@buhopago.com para solicitar el scope `credits:execute` en tu API key.

---

## 📊 Cómo Funciona

### Sistema de Capacidad de Procesamiento

```
┌─────────────────────────────────────────────────────────────┐
│  1. Recibes transacciones → Incrementa "volumen procesado"  │
│  2. Ejecutas crédito      → Reduce "volumen disponible"     │
│  3. Volumen disponible    = Procesado - Retirado            │
└─────────────────────────────────────────────────────────────┘
```

**Ejemplo:**
- Procesaste $10,000 en transacciones → Volumen disponible: $10,000
- Retiras $3,000 vía crédito → Volumen disponible: $7,000
- Recibes $5,000 más → Volumen disponible: $12,000

---

## 🔐 Autenticación

Todas las peticiones requieren tu API key con el scope `credits:execute`:

```http
Authorization: Bearer bp_live_abc123def456...
```

---

## 📡 Endpoints Disponibles

### 1. Ejecutar Crédito Inmediato

Retira fondos de tu volumen disponible a una cuenta bancaria.

**Endpoint:**
```
POST /api/v1/credits/execute
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
Content-Type: application/json
```

**Request Body:**
```json
{
  "bank_account_id": "550e8400-e29b-41d4-a716-446655440000",
  "amount": 1000.50,
  "concept": "Retiro para conversion"
}
```

**Parámetros:**

| Campo | Tipo | Requerido | Descripción |
|-------|------|-----------|-------------|
| `bank_account_id` | string (UUID) | ✅ | ID de tu cuenta bancaria registrada |
| `amount` | number | ✅ | Monto a retirar (debe ser > 0) |
| `concept` | string | ✅ | Concepto del retiro (máx 30 caracteres) |

**Respuesta Exitosa (200):**
```json
{
  "success": true,
  "message": "Crédito aprobado",
  "status": "APROBADA",
  "transaction_id": "OP-2024-12345678",
  "bank_response": {
    "id": "OP-2024-12345678",
    "reference": "REF-987654321",
    "message": "Operación exitosa"
  },
  "capacity_remaining": 8999.50
}
```

**Respuesta - Crédito Pendiente (200):**
```json
{
  "success": true,
  "message": "Crédito en espera",
  "status": "ESPERA",
  "transaction_id": "OP-2024-12345678",
  "bank_response": {
    "id": "OP-2024-12345678",
    "message": "Transacción en proceso"
  },
  "capacity_remaining": null
}
```

**Estados Posibles:**
- `APROBADA` - Crédito ejecutado exitosamente
- `ESPERA` - Crédito en proceso (consultar luego)
- `RECHAZADA` - Crédito rechazado por el banco

---

### 2. Consultar Capacidad Disponible

Verifica cuánto volumen tienes disponible para retirar.

**Endpoint:**
```
GET /api/v1/credits/capacity
```

**Headers:**
```http
Authorization: Bearer bp_live_xxx
```

**Respuesta (200):**
```json
{
  "success": true,
  "capacity": {
    "volume_processed_total": 50000.00,
    "volume_credited_total": 15000.00,
    "volume_available": 35000.00,
    "last_transaction_at": "2024-01-15T14:30:00Z",
    "updated_at": "2024-01-15T14:30:00Z"
  }
}
```

**Campos de Respuesta:**

| Campo | Descripción |
|-------|-------------|
| `volume_processed_total` | Total histórico procesado en transacciones |
| `volume_credited_total` | Total retirado mediante créditos inmediatos |
| `volume_available` | **Disponible actualmente para retirar** |
| `last_transaction_at` | Fecha de la última transacción procesada |
| `updated_at` | Última actualización del registro |

---

## 🚨 Códigos de Error

### 403 Forbidden - Capacidad Insuficiente

```json
{
  "detail": {
    "error": "insufficient_capacity",
    "message": "Capacidad insuficiente. Disponible: 500.00, Requerido: 1000.00",
    "available_capacity": 500.00,
    "required_amount": 1000.00
  }
}
```

**Solución:** Espera a recibir más transacciones o reduce el monto a retirar.

---

### 403 Forbidden - Sin Permisos

```json
{
  "detail": "API key no tiene permisos para ejecutar créditos inmediatos. Contacta a soporte para habilitar este permiso."
}
```

**Solución:** Contacta a soporte para que agreguen el scope `credits:execute` a tu API key.

---

### 400 Bad Request - Usuario sin KYC

```json
{
  "detail": "Usuario no tiene cédula registrada. Completa tu KYC primero."
}
```

**Solución:** Completa el proceso de verificación KYC en tu dashboard.

---

### 404 Not Found - Cuenta Bancaria

```json
{
  "detail": "Cuenta bancaria no encontrada o no pertenece al usuario"
}
```

**Solución:** Verifica que el `bank_account_id` sea correcto y que la cuenta te pertenezca.

---

## 💡 Ejemplos de Uso

### Ejemplo con cURL

```bash
# 1. Consultar capacidad disponible
curl https://api.buhopago.com/api/v1/credits/capacity \
  -H "Authorization: Bearer bp_live_xxx"

# 2. Ejecutar crédito inmediato
curl -X POST https://api.buhopago.com/api/v1/credits/execute \
  -H "Authorization: Bearer bp_live_xxx" \
  -H "Content-Type: application/json" \
  -d '{
    "bank_account_id": "550e8400-e29b-41d4-a716-446655440000",
    "amount": 1000,
    "concept": "Retiro para operacion"
  }'
```

### Ejemplo con Python

```python
import requests

API_KEY = "bp_live_xxx"
BASE_URL = "https://api.buhopago.com/api/v1"

headers = {
    "Authorization": f"Bearer {API_KEY}",
    "Content-Type": "application/json"
}

# 1. Consultar capacidad
response = requests.get(
    f"{BASE_URL}/credits/capacity",
    headers=headers
)
capacity = response.json()
print(f"Disponible: ${capacity['capacity']['volume_available']}")

# 2. Ejecutar crédito si hay capacidad suficiente
if capacity['capacity']['volume_available'] >= 1000:
    credit_data = {
        "bank_account_id": "550e8400-e29b-41d4-a716-446655440000",
        "amount": 1000,
        "concept": "Retiro automatizado"
    }

    response = requests.post(
        f"{BASE_URL}/credits/execute",
        headers=headers,
        json=credit_data
    )

    result = response.json()
    if result['success'] and result['status'] == 'APROBADA':
        print(f"✅ Crédito aprobado: {result['transaction_id']}")
        print(f"💰 Capacidad restante: ${result['capacity_remaining']}")
    else:
        print(f"⏳ Crédito en proceso: {result['status']}")
else:
    print("❌ Capacidad insuficiente")
```

### Ejemplo con Node.js

```javascript
const axios = require('axios');

const API_KEY = 'bp_live_xxx';
const BASE_URL = 'https://api.buhopago.com/api/v1';

const headers = {
  'Authorization': `Bearer ${API_KEY}`,
  'Content-Type': 'application/json'
};

// 1. Consultar capacidad
async function checkCapacity() {
  const response = await axios.get(`${BASE_URL}/credits/capacity`, { headers });
  return response.data;
}

// 2. Ejecutar crédito
async function executeCredit(bankAccountId, amount, concept) {
  try {
    const response = await axios.post(
      `${BASE_URL}/credits/execute`,
      {
        bank_account_id: bankAccountId,
        amount: amount,
        concept: concept
      },
      { headers }
    );

    return response.data;
  } catch (error) {
    if (error.response?.status === 403) {
      console.error('❌ Capacidad insuficiente o sin permisos');
      console.error(error.response.data);
    }
    throw error;
  }
}

// Uso
async function main() {
  // Verificar capacidad
  const capacity = await checkCapacity();
  console.log(`Disponible: $${capacity.capacity.volume_available}`);

  // Ejecutar crédito
  if (capacity.capacity.volume_available >= 1000) {
    const result = await executeCredit(
      '550e8400-e29b-41d4-a716-446655440000',
      1000,
      'Retiro automatizado'
    );

    if (result.success && result.status === 'APROBADA') {
      console.log(`✅ Crédito aprobado: ${result.transaction_id}`);
      console.log(`💰 Restante: $${result.capacity_remaining}`);
    }
  }
}

main();
```

---

## 🔄 Flujo Recomendado

### Para Retiros Automáticos

```
1. Recibir notificación de transacción completada (webhook)
   ↓
2. Consultar capacidad disponible
   ↓
3. Si capacidad >= monto deseado:
   ├─ Ejecutar crédito inmediato
   └─ Guardar transaction_id para tracking
   ↓
4. Si status = "APROBADA":
   ├─ Proceder con conversión a stablecoin
   └─ Actualizar registros internos
   ↓
5. Si status = "ESPERA":
   └─ Implementar polling o esperar notificación
```

---

## 📝 Mejores Prácticas

### ✅ Hacer

1. **Verificar capacidad antes de ejecutar**
   ```python
   capacity = get_capacity()
   if capacity['volume_available'] >= amount:
       execute_credit(amount)
   ```

2. **Guardar transaction_id para tracking**
   ```python
   result = execute_credit(1000)
   db.save_transaction_id(result['transaction_id'])
   ```

3. **Implementar reintentos con backoff**
   ```python
   import time
   for attempt in range(3):
       try:
           result = execute_credit(amount)
           break
       except Exception as e:
           if attempt < 2:
               time.sleep(2 ** attempt)
   ```

4. **Monitorear volumen disponible**
   - Consulta periódicamente tu capacidad
   - Configura alertas cuando esté bajo

5. **Usar conceptos descriptivos**
   - ✅ "Retiro para conversion BTC"
   - ❌ "Retiro"

### ❌ No Hacer

1. **No ejecutar sin verificar capacidad primero**
2. **No ignorar errores de capacidad insuficiente**
3. **No hacer múltiples retiros simultáneos** (puede causar problemas de concurrencia)
4. **No asumir que ESPERA = error** (puede procesarse después)

---

## 🔒 Seguridad

### Protege tu API Key

```bash
# ✅ Usar variables de entorno
export BUHOPAGO_API_KEY="bp_live_xxx"

# ❌ NO hardcodear en el código
api_key = "bp_live_xxx"  # ¡NUNCA!
```

### Rate Limiting

- Máximo 100 solicitudes por minuto
- Máximo 10 créditos simultáneos en proceso

### IP Whitelisting

Puedes restringir tu API key a IPs específicas contactando a soporte.

---

## 🆘 Soporte

¿Necesitas ayuda?

- 📧 Email: support@buhopago.com
- 💬 Telegram: @BuhoPagoSupport
- 🌐 Docs: https://docs.buhopago.com

Para solicitar acceso a la Credits API, envía un email con:
- Nombre de tu empresa
- Caso de uso
- Volumen mensual estimado
