Documentación de Credits API - BúhoPago
📋 Versión Actualizada (v1.3.0)
Base URL: https://points0.com/public-api

🚀 Caso de Uso: Exchange Platform con Credits API
Características Nuevas (v1.3.0):
Modo Manual: Envío directo con datos bancarios sin cuenta registrada

📋 Endpoints Disponibles

1. Consultar Capacidad de Créditos
   GET /credits/capacity

Consulta el volumen disponible para procesamiento de créditos inmediatos.

Headers requeridos:
http
Authorization: Bearer {api_key}
Scopes requeridos:
json
{
"required_scope": "credits:execute"
}
Respuesta Exitosa (200 OK):
json
{
"success": true,
"capacity": {
"volume_processed_total": 15000.00,
"volume_credited_total": 14500.00,
"volume_available": 500.00,
"last_transaction_at": "2024-01-15T10:30:00Z"
}
}
Errores posibles:
403 Forbidden: API key no tiene el scope credits:execute

500 Internal Server Error: Error del servidor

2. Ejecutar Crédito Inmediato
   POST /credits/execute

Ejecuta un crédito inmediato a una cuenta bancaria. Soporta dos modos.

Headers requeridos:
http
Authorization: Bearer {api_key}
Content-Type: application/json
Scopes requeridos:
json
{
"required_scope": "credits:execute"
}
2.1 Modo Tradicional (Cuenta Registrada)
Request Body:
json
{
"bank_account_id": "bank_acc_123456789",
"amount": 100.00,
"concept": "Pago por servicios"
}
Parámetros:
Campo Tipo Requerido Descripción
bank_account_id string ✅ Sí ID de la cuenta bancaria registrada
amount decimal ✅ Sí Monto a acreditar (mínimo: 1.00)
concept string ✅ Sí Concepto de la transacción
2.2 Modo Manual (Datos Directos) - NUEVO en v1.3.0
Request Body:
json
{
"cedula_manual": "V30552028",
"account_number_manual": "01050123456789012345",
"bank_code_manual": "0105",
"amount": 50.00,
"concept": "Reembolso manual"
}
Parámetros:
Campo Tipo Requerido Descripción
cedula_manual string ✅ Sí Cédula/RIF del beneficiario
account_number_manual string ✅ Sí Número de cuenta bancaria
bank_code_manual string ✅ Sí Código del banco (ej: 0105)
amount decimal ✅ Sí Monto a acreditar
concept string ✅ Sí Concepto de la transacción
Nota: En modo manual, TODOS los campos \*\_manual son requeridos si no se proporciona bank_account_id.

Respuesta Exitosa (200 OK):
json
{
"success": true,
"status": "processing",
"transaction_id": "cred_abc123xyz789",
"capacity_remaining": 450.00,
"message": "Crédito en proceso"
}
Errores posibles:
Código Error Descripción
400 Bad Request Campos inválidos o faltantes
403 Insufficient Capacity Volumen disponible insuficiente
403 Scope Required API key sin permisos credits:execute
404 Bank Account Not Found Cuenta bancaria no existe
422 Validation Error Datos manuales incompletos
Ejemplo de Error por Capacidad Insuficiente:
json
{
"detail": {
"error": "insufficient_capacity",
"available_capacity": 25.00,
"required_amount": 100.00,
"message": "Volumen disponible insuficiente para la transacción"
}
}
💡 Implementación Actualizada

1. Cliente Mejorado (Soporta ambos modos)
   python
   import requests
   from typing import Dict, Optional

class BuhoPagoCreditsClient:
def **init**(self, api_key: str, base_url: str = "https://points0.com/public-api"):
self.api_key = api_key
self.base_url = base_url
self.headers = {
"Authorization": f"Bearer {api_key}",
"Content-Type": "application/json"
}

    def get_capacity(self) -> Dict:
        """Obtener capacidad disponible (REQUIERE scope: credits:execute)"""
        response = requests.get(
            f"{self.base_url}/credits/capacity",
            headers=self.headers
        )

        if response.status_code == 200:
            return response.json()
        elif response.status_code == 403:
            raise PermissionError("API key no tiene scope 'credits:execute'")
        else:
            response.raise_for_status()

    def execute_credit(
        self,
        # Parámetros tradicionales
        bank_account_id: Optional[str] = None,
        # Parámetros modo manual (NUEVO en v1.3.0)
        cedula_manual: Optional[str] = None,
        account_number_manual: Optional[str] = None,
        bank_code_manual: Optional[str] = None,
        # Parámetros comunes
        amount: float,
        concept: str
    ) -> Dict:
        """
        Ejecutar crédito inmediato

        Modos disponibles:
        1. Tradicional: bank_account_id (cuenta pre-registrada)
        2. Manual: cedula_manual + account_number_manual + bank_code_manual
        """

        # Construir payload según modo
        payload = {"amount": amount, "concept": concept}

        if bank_account_id:
            # Modo tradicional
            payload["bank_account_id"] = bank_account_id
        elif cedula_manual and account_number_manual and bank_code_manual:
            # Modo manual (NUEVO)
            payload.update({
                "cedula_manual": cedula_manual,
                "account_number_manual": account_number_manual,
                "bank_code_manual": bank_code_manual
            })
        else:
            raise ValueError(
                "Debe proporcionar:\n"
                "1. bank_account_id (modo tradicional) O\n"
                "2. cedula_manual + account_number_manual + bank_code_manual (modo manual)"
            )

        response = requests.post(
            f"{self.base_url}/credits/execute",
            headers=self.headers,
            json=payload
        )

        return self._handle_credit_response(response)

    def _handle_credit_response(self, response) -> Dict:
        """Manejar respuesta con errores específicos"""
        if response.status_code == 200:
            return response.json()

        # Errores específicos
        elif response.status_code == 403:
            error_data = response.json()
            detail = error_data.get('detail', {})

            # Capacidad insuficiente
            if isinstance(detail, dict) and detail.get('error') == 'insufficient_capacity':
                raise InsufficientCapacityError(
                    f"Capacidad insuficiente: Disponible ${detail.get('available_capacity')}, "
                    f"Requiere ${detail.get('required_amount')}"
                )
            # Scope faltante
            elif isinstance(detail, str) and 'no tiene permisos' in detail:
                raise PermissionError("API key no tiene scope 'credits:execute'")
            else:
                response.raise_for_status()

        # Validación de campos
        elif response.status_code in [400, 422]:
            error_data = response.json()
            raise ValidationError(f"Error de validación: {error_data}")

        else:
            response.raise_for_status()

class InsufficientCapacityError(Exception):
"""Excepción para capacidad insuficiente"""
pass

class ValidationError(Exception):
"""Excepción para errores de validación"""
pass 2. Sistema de Retiro Inteligente (Con Fallback)
python
class IntelligentWithdrawalSystem:
def **init**(self, buhopago_client: BuhoPagoCreditsClient):
self.client = buhopago_client
self.primary_account_id = "your-primary-bank-account-id"
self.min_withdrawal = 100.00 # Mínimo $100

    def withdraw(self, amount: float, concept: str,
                 fallback_data: Optional[Dict] = None) -> Dict:
        """
        Intenta retiro con fallback automático

        Args:
            amount: Monto a retirar
            concept: Concepto de la transacción
            fallback_data: Datos para modo manual si falla tradicional
                {
                    "cedula": "V30552028",
                    "account_number": "01050123456789012345",
                    "bank_code": "0105"
                }
        """

        # 1. Verificar capacidad
        try:
            capacity = self.client.get_capacity()
            available = capacity['capacity']['volume_available']

            if available < amount:
                raise InsufficientCapacityError(
                    f"Capacidad insuficiente: ${available} disponible, "
                    f"se requieren ${amount}"
                )

        except PermissionError:
            print("⚠️ API key sin scope 'credits:execute'. "
                  "Solicita el scope a support@buhopago.com")
            return {"success": False, "error": "scope_required"}

        # 2. Intento primario: Modo tradicional
        try:
            print("🔄 Intentando retiro en modo tradicional...")
            result = self.client.execute_credit(
                bank_account_id=self.primary_account_id,
                amount=amount,
                concept=concept
            )

            if result.get('success'):
                print(f"✅ Retiro exitoso: {result.get('transaction_id')}")
                return {
                    "success": True,
                    "mode": "traditional",
                    "transaction_id": result.get('transaction_id'),
                    "status": result.get('status')
                }

        except Exception as e:
            print(f"⚠️ Modo tradicional falló: {e}")

            # 3. Fallback: Modo manual (si hay datos)
            if fallback_data:
                try:
                    print("🔄 Intentando retiro en modo manual...")
                    result = self.client.execute_credit(
                        cedula_manual=fallback_data["cedula"],
                        account_number_manual=fallback_data["account_number"],
                        bank_code_manual=fallback_data["bank_code"],
                        amount=amount,
                        concept=concept
                    )

                    if result.get('success'):
                        print(f"✅ Retiro manual exitoso: {result.get('transaction_id')}")
                        return {
                            "success": True,
                            "mode": "manual",
                            "transaction_id": result.get('transaction_id'),
                            "status": result.get('status')
                        }

                except ValidationError as ve:
                    print(f"❌ Error validación modo manual: {ve}")
                    return {"success": False, "error": "validation_failed", "details": str(ve)}

            return {"success": False, "error": "withdrawal_failed", "details": str(e)}

3. Webhook Handler Mejorado
   python
   from flask import Flask, request, jsonify
   import hmac
   import hashlib

app = Flask(**name**)

class EnhancedWebhookHandler:
def **init**(self, withdrawal_system: IntelligentWithdrawalSystem):
self.withdrawal_system = withdrawal_system

    def handle_payment_completed(self, transaction_data: Dict):
        """Manejar pago completado y ejecutar retiro automático"""

        # Extraer datos relevantes
        amount = transaction_data['amount']
        customer_data = transaction_data.get('customer', {})

        # Preparar datos para retiro
        withdrawal_payload = {
            "amount": amount,
            "concept": f"Retiro auto - Transacción {transaction_data['id']}",
            "fallback_data": None
        }

        # Si el cliente proporcionó datos bancarios, usarlos como fallback
        if customer_data.get('bank_details'):
            withdrawal_payload["fallback_data"] = {
                "cedula": customer_data.get('document_id'),
                "account_number": customer_data['bank_details'].get('account_number'),
                "bank_code": customer_data['bank_details'].get('bank_code')
            }

        # Ejecutar retiro
        result = self.withdrawal_system.withdraw(**withdrawal_payload)

        # Registrar resultado
        self.log_withdrawal_result(transaction_data['id'], result)

        return result

    def log_withdrawal_result(self, transaction_id: str, result: Dict):
        """Registrar resultado del retiro para auditoría"""
        log_entry = {
            "timestamp": datetime.now().isoformat(),
            "transaction_id": transaction_id,
            "result": result
        }

        # Guardar en base de datos o archivo
        print(f"📝 Log retiro: {log_entry}")

@app.route('/webhook/payment-completed', methods=['POST'])
def handle_webhook():
"""Endpoint para webhook de pagos completados"""

    # Verificar firma (omitido por brevedad)
    # ...

    data = request.json

    if data['event'] == 'payment.completed':
        handler = EnhancedWebhookHandler(withdrawal_system)
        result = handler.handle_payment_completed(data['data'])

        return jsonify({
            "status": "processed",
            "withdrawal_result": result
        }), 200

    return jsonify({"status": "ignored"}), 200

4. Ejemplo Completo de Uso
   python

# Configuración

API_KEY = "bp_live_your_api_key_here"
BASE_URL = "https://points0.com/public-api"

# Inicializar cliente

client = BuhoPagoCreditsClient(api_key=API_KEY, base_url=BASE_URL)
withdrawal_system = IntelligentWithdrawalSystem(client)

# Escenario 1: Cliente con cuenta registrada

def scenario_traditional():
"""Cliente que ya tiene cuenta bancaria registrada"""
try: # Verificar capacidad
capacity = client.get_capacity()
print(f"💰 Capacidad disponible: ${capacity['capacity']['volume_available']}")

        # Ejecutar retiro tradicional
        result = client.execute_credit(
            bank_account_id="bank_acc_123456789",
            amount=500.00,
            concept="Pago por servicios de intercambio"
        )

        print(f"✅ Retiro exitoso: {result}")

    except InsufficientCapacityError as e:
        print(f"❌ {e}")
    except PermissionError as e:
        print(f"🔒 {e} - Contacta a support@buhopago.com")
    except ValidationError as e:
        print(f"📝 Error de validación: {e}")

# Escenario 2: Cliente nuevo (modo manual)

def scenario_manual():
"""Cliente que no tiene cuenta registrada"""
try:
result = client.execute_credit(
cedula_manual="V30552028",
account_number_manual="01050123456789012345",
bank_code_manual="0105",
amount=250.00,
concept="Primer retiro - Datos manuales"
)

        print(f"✅ Retiro manual exitoso: {result}")

    except ValidationError as e:
        print(f"❌ Datos inválidos: {e}")
    except Exception as e:
        print(f"⚠️ Error: {e}")

# Escenario 3: Sistema automático con fallback

def scenario_automatic_with_fallback():
"""Sistema que intenta tradicional primero, luego manual"""

    # Datos del cliente
    customer = {
        "id": "cust_123",
        "name": "Juan Pérez",
        "has_registered_account": False,  # No tiene cuenta registrada
        "bank_details": {
            "cedula": "V30552028",
            "account_number": "01050123456789012345",
            "bank_code": "0105"
        }
    }

    # Intentar retiro
    withdrawal_payload = {
        "amount": 300.00,
        "concept": f"Retiro para {customer['name']}",
        "fallback_data": customer['bank_details'] if not customer['has_registered_account'] else None
    }

    result = withdrawal_system.withdraw(**withdrawal_payload)

    if result["success"]:
        print(f"✅ Retiro {result['mode']} exitoso. ID: {result['transaction_id']}")

        # Proceder con conversión a cripto
        if result['status'] == 'APROBADA':
            convert_to_crypto(amount=300.00, customer_id=customer['id'])
    else:
        print(f"❌ Falló retiro: {result.get('error')}")

def convert_to_crypto(amount: float, customer_id: str):
"""Convertir a criptomoneda después de retiro exitoso"""
print(f"💱 Convirtiendo ${amount} a USDT para cliente {customer_id}") # Tu lógica de conversión aquí # exchange_rate = get_exchange_rate() # usdt_amount = amount / exchange_rate # send_to_wallet(customer_id, usdt_amount) 5. Tests de Validación (Basados en los tests proporcionados)
python
class CreditsAPITester:
def **init**(self, api_key: str, base_url: str = "https://points0.com/public-api"):
self.api_key = api_key
self.base_url = base_url
self.headers = {"Authorization": f"Bearer {api_key}"}

    def test_get_credits_capacity(self):
        """Test 31: Consultar capacidad de procesamiento disponible"""
        try:
            response = requests.get(
                f"{self.base_url}/credits/capacity",
                headers=self.headers
            )

            if response.status_code == 200:
                data = response.json()
                if data.get('success') and data.get('capacity'):
                    capacity = data['capacity']
                    print("✅ Capacidad obtenida exitosamente")
                    return capacity
                else:
                    print("❌ Respuesta sin datos de capacidad")
                    return None
            elif response.status_code == 403:
                print("⚠️ API key sin scope 'credits:execute' (esperado)")
                return None
            else:
                print(f"❌ Status code: {response.status_code}")
                return None

        except Exception as e:
            print(f"❌ Exception: {str(e)}")
            return None

    def test_execute_credit_with_manual_data(self):
        """Test 34: Ejecutar crédito con datos manuales (sin cuenta registrada)"""
        try:
            payload = {
                'cedula_manual': 'V30552028',
                'account_number_manual': '01050123456789012345',
                'bank_code_manual': '0105',
                'amount': 50.00,
                'concept': 'Test credito manual'
            }

            response = requests.post(
                f"{self.base_url}/credits/execute",
                json=payload,
                headers=self.headers
            )

            if response.status_code == 200:
                data = response.json()
                if data.get('success'):
                    print(f"✅ Crédito manual ejecutado - Status: {data.get('status')}")
                    return data
                else:
                    print("❌ Crédito no exitoso")
                    return None
            elif response.status_code == 403:
                error_detail = response.json().get('detail', {})
                if isinstance(error_detail, dict) and error_detail.get('error') == 'insufficient_capacity':
                    print("⚠️ Capacidad insuficiente (esperado si no hay volumen)")
                    return None
                elif isinstance(error_detail, str) and 'no tiene permisos' in error_detail:
                    print("⚠️ API key sin scope 'credits:execute'")
                    return None
                else:
                    print(f"❌ Error 403: {error_detail}")
                    return None
            elif response.status_code == 400:
                print("❌ Bad Request")
                return None
            else:
                print(f"❌ Status code: {response.status_code}")
                return None

        except Exception as e:
            print(f"❌ Exception: {str(e)}")
            return None

    def test_execute_credit_missing_fields(self):
        """Test 35: Intentar ejecutar crédito sin proporcionar ni bank_account_id ni datos manuales"""
        try:
            payload = {
                'amount': 50.00,
                'concept': 'Test sin datos'
            }

            response = requests.post(
                f"{self.base_url}/credits/execute",
                json=payload,
                headers=self.headers
            )

            # 400/422 = Validación de campos
            # 403 = Sin permisos (scope) - La validación de scope ocurre antes
            if response.status_code in [400, 422]:
                print("✅ Correctamente rechazado por falta de campos requeridos")
                return True
            elif response.status_code == 403:
                print("✅ API key sin scope 'credits:execute' (no se pudo validar campos)")
                return True
            else:
                print(f"❌ Esperaba 400/422/403, obtuve {response.status_code}")
                return False

        except Exception as e:
            print(f"❌ Exception: {str(e)}")
            return False

    def test_execute_credit_partial_manual_data(self):
        """Test 36: Intentar ejecutar crédito con datos manuales incompletos"""
        try:
            # Solo proporcionar cedula_manual y account_number, falta bank_code
            payload = {
                'cedula_manual': 'V30552028',
                'account_number_manual': '01050123456789012345',
                'amount': 50.00,
                'concept': 'Test datos incompletos'
            }

            response = requests.post(
                f"{self.base_url}/credits/execute",
                json=payload,
                headers=self.headers
            )

            # 400/422 = Validación de campos
            # 403 = Sin permisos (scope) - La validación de scope ocurre antes
            if response.status_code in [400, 422]:
                print("✅ Correctamente rechazado por datos manuales incompletos")
                return True
            elif response.status_code == 403:
                print("✅ API key sin scope 'credits:execute' (no se pudo validar campos)")
                return True
            else:
                print(f"❌ Esperaba 400/422/403, obtuve {response.status_code}")
                return False

        except Exception as e:
            print(f"❌ Exception: {str(e)}")
            return False

📊 Reglas de Validación
Validaciones Comunes:
Scope obligatorio: credits:execute requerido para ambos endpoints

Formato de montos: Decimal positivo con 2 decimales

Concepto: Máximo 255 caracteres

Validaciones Específicas:
Para Modo Tradicional:
bank_account_id debe existir y estar activo

Cuenta bancaria debe pertenecer al usuario autenticado

Para Modo Manual:
Grupo completo: Si se proporciona cualquier campo \*\_manual, todos son requeridos

Formato cédula: V/E/J + números (ej: V30552028, J-12345678-1)

Código banco: 4 dígitos numéricos válidos

Número cuenta: Entre 15-20 dígitos

🔒 Seguridad y Mejores Prácticas
Requisitos de Scope:
python

# La API key DEBE tener el scope:

REQUIRED_SCOPES = ["credits:execute"]

# Verificación en dashboard:

# 1. Ve a https://dashboard.buhopago.com/api-keys

# 2. Edita tu API key

# 3. Marca el scope "credits:execute"

Manejo de Errores Recomendado:
python
ERROR_HANDLING = {
403: {
"insufficient_capacity": "Esperar o reducir monto",
"scope_required": "Solicitar scope credits:execute"
},
400: "Verificar formato de datos",
422: "Datos manuales incompletos",
404: "Cuenta bancaria no encontrada"
}
Límites y Validaciones:
Monto mínimo: $1.00

Datos manuales: Grupo completo requerido (cedula + cuenta + banco)

Formato cédula: V/E/J seguido de números

Formato cuenta: 15-20 dígitos

Código banco: 4 dígitos numéricos

📊 Métricas y Monitoreo
python
class CreditsAPIMonitor:
def **init**(self, client: BuhoPagoCreditsClient):
self.client = client
self.metrics = {
"successful_withdrawals": 0,
"failed_withdrawals": 0,
"total_volume": 0.0,
"mode_usage": {"traditional": 0, "manual": 0}
}

    def track_withdrawal(self, result: Dict):
        """Registrar métricas de retiro"""
        if result.get("success"):
            self.metrics["successful_withdrawals"] += 1
            self.metrics["mode_usage"][result.get("mode", "unknown")] += 1
        else:
            self.metrics["failed_withdrawals"] += 1

        # Enviar a sistema de monitoreo
        self.send_to_monitoring_system(self.metrics)

    def print_dashboard(self):
        """Mostrar dashboard en consola"""
        total = self.metrics["successful_withdrawals"] + self.metrics["failed_withdrawals"]
        success_rate = (self.metrics["successful_withdrawals"] / total * 100) if total > 0 else 0

        print("=" * 50)
        print("📊 DASHBOARD CREDITS API")
        print("=" * 50)
        print(f"✅ Retiros exitosos: {self.metrics['successful_withdrawals']}")
        print(f"❌ Retiros fallidos: {self.metrics['failed_withdrawals']}")
        print(f"📈 Tasa de éxito: {success_rate:.1f}%")
        print(f"🏦 Modo tradicional: {self.metrics['mode_usage']['traditional']}")
        print(f"🆕 Modo manual: {self.metrics['mode_usage']['manual']}")
        print(f"💰 Volumen total: ${self.metrics['total_volume']:,.2f}")
        print("=" * 50)

🚨 Solución de Problemas
Problema Común 1: "API key no tiene permisos"
python

# Solución:

# 1. Verificar que la API key tenga scope "credits:execute"

# 2. En dashboard: Editar API key → Marcar checkbox

# 3. Si no ves la opción, contacta a support@buhopago.com

Problema Común 2: "Datos manuales incompletos"
python

# Error: Solo proporcionaste cedula_manual y account_number_manual

# Solución: Proveer TODOS los campos del grupo manual:

correct_payload = {
"cedula_manual": "V30552028",
"account_number_manual": "01050123456789012345",
"bank_code_manual": "0105", # ⬅️ NO OLVIDAR ESTE
"amount": 100.00,
"concept": "Test"
}
Problema Común 3: "Capacidad insuficiente"
python

# Soluciones:

# 1. Verificar capacity disponible: GET /credits/capacity

# 2. Reducir monto solicitado

# 3. Esperar a que se procesen más pagos entrantes

# 4. Contactar soporte para aumentar límites

📞 Soporte y Recursos
Documentación oficial: https://docs.buhopago.com

Soporte técnico: support@buhopago.com

API Status: https://status.buhopago.com

Ejemplos de código: https://github.com/buhopago/examples

Última actualización: Enero 2024
Versión API: 1.3.0
Base URL: https://points0.com/public-api
Estado: Producción ✅
