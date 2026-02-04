# 📝 Changelog

Historial de cambios y versiones de la API de BuhoPago.

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

### Planificado para v1.2.0
- Soporte para pagos recurrentes
- API de transacciones y reportes
- Webhooks con retry automático
- Soporte para refunds
- Dashboard analytics API

### En Consideración
- SDKs oficiales (Python, JavaScript, PHP)
- GraphQL API
- Soporte para más monedas
- Integración con plataformas de e-commerce

---

## 📢 Cómo Actualizar

### Breaking Changes
No hay breaking changes en esta versión. Todos los endpoints existentes permanecen compatibles.

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
