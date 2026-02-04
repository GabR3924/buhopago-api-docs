# ⚡ Inicio Rápido

Empieza a recibir pagos en menos de 5 minutos.

## 1️⃣ Obtén tu API Key

1. Inicia sesión en [BuhoPago](https://buhopago.com)
2. Ve a **Configuración → API Keys**
3. Crea una nueva API Key
4. Guarda tu key de forma segura: `bp_live_xxxxx...`

⚠️ **Importante**: Nunca compartas tu API Key ni la incluyas en código público.

## 2️⃣ Crea una Cuenta Bancaria

Antes de recibir pagos, necesitas configurar al menos una cuenta bancaria.

```bash
curl -X POST https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer bp_live_tu_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Tu Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente"
  }'
```

**Respuesta:**
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "bank": "Banco de Venezuela",
  "account_number": "01020123456789012345",
  "active": true,
  "created_at": "2026-02-03T10:30:00Z"
}
```

## 3️⃣ Crea tu Primer Payment Link

Ahora puedes crear links de pago que tus clientes pueden usar para pagarte.

```bash
curl -X POST https://points0.com/public-api/payment-links \
  -H "Authorization: Bearer bp_live_tu_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "currency": "USD",
    "description": "Pago de Producto Premium",
    "webhook_url": "https://tu-sitio.com/webhook",
    "return_url": "https://tu-sitio.com/success"
  }'
```

**Respuesta:**
```json
{
  "id": "link_abc123",
  "slug": "pay_xyz789",
  "url": "https://points0.com/pay/pay_xyz789",
  "amount": 50.00,
  "currency": "USD",
  "status": "active",
  "created_at": "2026-02-03T11:00:00Z"
}
```

🎉 **¡Listo!** Comparte el `url` con tu cliente.

## 4️⃣ Recibe Notificaciones (Opcional)

Configura webhooks para recibir notificaciones cuando un pago se complete.

```python
# Tu servidor recibe:
{
  "event": "payment.completed",
  "payment_link_id": "link_abc123",
  "amount": 50.00,
  "currency": "USD",
  "status": "completed",
  "payer_email": "cliente@example.com",
  "completed_at": "2026-02-03T11:30:00Z"
}
```

---

## 🎯 Próximos Pasos

### Para Payment Links
- [Documentación Completa de Payment Links](api-documentation.md#payment-links)
- [Personalizar Payment Links](api-documentation.md#crear-payment-link)
- [Configurar Webhooks](api-documentation.md#webhooks)

### Para Direct Payments
Si quieres integrar el flujo de pago directamente en tu interfaz:
- [Guía de Direct Payments](api-documentation.md#direct-payments-pagos-directos)
- [Caso de Uso: Tienda Online](use-cases/direct-payment-example.md)

### Gestión de Cuentas
- [Gestionar Bank Accounts](bank-accounts.md)
- [Listar y Actualizar Cuentas](bank-accounts.md#listar-cuentas-bancarias)

---

## 💡 Consejos

1. **Usa el entorno de pruebas** con `bp_test_` keys antes de producción
2. **Configura webhooks** para automatizar el procesamiento de pagos
3. **Implementa reintentos** para manejar errores de red
4. **Valida signatures** de webhooks para seguridad

## 🆘 ¿Necesitas Ayuda?

- **Email**: soporte@buhopago.com
- **Documentación**: [API Reference](api-documentation.md)
- **Ejemplos**: [Casos de Uso](use-cases/direct-payment-example.md)
