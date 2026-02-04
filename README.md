# 🦉 BuhoPago API - Documentación

Bienvenido a la documentación oficial de la API pública de BuhoPago. Acepta pagos móviles en Venezuela de forma segura y rápida.

## 🚀 Inicio Rápido

1. **Obtén tu API Key** en el panel de BuhoPago
2. **Crea una cuenta bancaria** para recibir pagos
3. **Empieza a recibir pagos** con Payment Links o Direct Payments

```bash
# Ejemplo rápido
curl -X POST https://points0.com/public-api/payment-links \
  -H "Authorization: Bearer bp_live_tu_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "currency": "USD",
    "description": "Pago de prueba"
  }'
```

## 📚 Características

- **Payment Links**: Crea links de pago personalizados
- **Direct Payments**: Integra pagos directamente en tu interfaz
- **Bank Accounts**: Gestiona tus cuentas bancarias
- **Webhooks**: Recibe notificaciones en tiempo real
- **Múltiples Monedas**: USD, EUR, VES

## 🔑 Autenticación

Todos los endpoints requieren autenticación mediante API Key:

```http
Authorization: Bearer bp_live_your_api_key_here
```

## 🌐 Base URL

```
https://points0.com/public-api
```

## 💡 Casos de Uso

### Payment Links
Ideal para:
- E-commerce
- Facturas por email
- Pagos únicos
- Enlaces compartibles

### Direct Payments
Ideal para:
- Integraciones personalizadas
- Flujos de pago en tu interfaz
- Control total del UX
- Apps móviles

## 📖 Documentación Completa

- [**API Reference**](api-documentation.md) - Referencia completa de endpoints
- [**Bank Accounts**](bank-accounts.md) - Gestión de cuentas bancarias
- [**Casos de Uso**](use-cases/direct-payment-example.md) - Ejemplos prácticos
- [**Changelog**](changelog.md) - Historial de versiones

## 🆘 Soporte

- **Email**: soporte@buhopago.com
- **Documentación**: https://docs.buhopago.com
- **Base URL**: https://points0.com/public-api

## 📦 Versión Actual

**v1.1.0** - Última actualización: 2026-02-03

---

**¿Listo para empezar?** Lee el [Inicio Rápido](quick-start.md) o ve directamente a la [Documentación Completa](api-documentation.md).
