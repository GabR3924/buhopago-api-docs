# 🆘 Soporte y Ayuda

¿Necesitas ayuda con la API de BuhoPago? Estamos aquí para ayudarte.

---

## 📧 Contacto

### Soporte Técnico
- **Email**: soporte@buhopago.com
- **Respuesta**: 24-48 horas hábiles

### Ventas y Consultas Comerciales
- **Email**: ventas@buhopago.com
- **Teléfono**: +58 (XXX) XXX-XXXX

### Reportar Bugs
- **Email**: bugs@buhopago.com
- **Incluir**:
  - Descripción del problema
  - Request ID (si está disponible)
  - API Key prefix (primeros 10 caracteres)
  - Pasos para reproducir

---

## 🔍 Antes de Contactar

### Revisa la Documentación
1. [Inicio Rápido](quick-start.md) - Configuración básica
2. [API Reference](api-documentation.md) - Documentación completa
3. [Bank Accounts](bank-accounts.md) - Gestión de cuentas
4. [Casos de Uso](use-cases/direct-payment-example.md) - Ejemplos prácticos

### Problemas Comunes

#### "No tienes cuenta bancaria activa"
**Solución**: Debes crear al menos una cuenta bancaria antes de crear payment links.

```bash
curl -X POST https://points0.com/public-api/bank-accounts \
  -H "Authorization: Bearer tu_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "bank": "Banco de Venezuela",
    "account_number": "01020123456789012345",
    "account_holder": "Tu Empresa S.A.",
    "rif": "J-12345678-9",
    "account_type": "Corriente"
  }'
```

#### "401 Unauthorized"
**Solución**: Verifica que:
- Tu API Key esté correcta
- Estés usando el formato: `Bearer bp_live_xxxxx`
- La API Key no haya expirado

#### "422 Validation Error"
**Solución**: Revisa que todos los campos requeridos estén presentes y tengan el formato correcto.

#### Webhooks no se reciben
**Solución**:
- Verifica que tu webhook URL sea accesible públicamente
- Usa HTTPS (no HTTP)
- Responde con 200 OK en menos de 5 segundos
- Verifica que no haya firewall bloqueando las IPs de BuhoPago

---

## 📚 Recursos Adicionales

### Documentación
- [Documentación Completa](api-documentation.md)
- [Changelog](changelog.md)
- [Ejemplos de Código](use-cases/direct-payment-example.md)

### Herramientas
- [Postman Collection](#) - Próximamente
- [SDK de Python](#) - Próximamente
- [SDK de JavaScript](#) - Próximamente

### Testing
- [Script de Pruebas](https://github.com/buhopago/api-tests) - Próximamente
- Ambiente de Sandbox - Próximamente

---

## 🐛 Reportar Problemas

Si encuentras un bug o problema:

1. **Verifica** que estés usando la última versión de la API
2. **Revisa** la documentación y ejemplos
3. **Contacta** a soporte con:
   - API key prefix (solo los primeros caracteres)
   - Request ID (si está disponible)
   - Logs de error completos
   - Pasos para reproducir el problema

**Email**: bugs@buhopago.com

---

## 💡 Sugerencias y Feedback

¿Tienes ideas para mejorar la API?

- **Email**: feedback@buhopago.com
- **Incluir**:
  - Descripción de la feature
  - Caso de uso
  - Beneficios esperados

---

## 🔐 Seguridad

Si descubres una vulnerabilidad de seguridad:

⚠️ **NO** la reportes públicamente

**Email**: security@buhopago.com

Trabajamos con investigadores de seguridad de forma responsable y agradecemos los reportes.

---

## 📊 Estado del Servicio

### Uptime
- **Objetivo**: 99.9% uptime
- **Actual**: Consulta en tiempo real en [status.buhopago.com](#)

### Mantenimiento Programado
Los mantenimientos se anuncian con 48 horas de anticipación vía:
- Email a usuarios registrados
- [Status Page](#)
- Twitter: [@BuhoPago](#)

---

## 🌎 Horarios de Soporte

### Soporte Técnico
- **Lunes a Viernes**: 9:00 AM - 6:00 PM (Venezuela, GMT-4)
- **Sábados**: 10:00 AM - 2:00 PM
- **Domingos y Feriados**: Cerrado (emergencias vía email)

### Respuesta Esperada
- **Crítico**: 2-4 horas
- **Alto**: 24 horas
- **Medio**: 48 horas
- **Bajo**: 72 horas

---

## 📝 Términos de Servicio

- [Términos y Condiciones](https://buhopago.com/terms)
- [Política de Privacidad](https://buhopago.com/privacy)
- [SLA](https://buhopago.com/sla)

---

**¿Listo para empezar?** Regresa al [Inicio](README.md) o comienza con la [Guía Rápida](quick-start.md).
