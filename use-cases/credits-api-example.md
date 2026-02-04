📋 Endpoints Disponibles
BASE_URL = 'https://points0.com/public-api'
1. Consultar Capacidad de Créditos
GET /credits/capacity

Consulta el volumen disponible en tu cuenta para procesar créditos inmediatos.

Headers requeridos:
http
Authorization: Bearer {tu_api_key}
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
403 Forbidden: Tu API key no tiene el scope credits:execute

2. Ejecutar Crédito Inmediato
POST /credits/execute

Envía un crédito inmediato a una cuenta bancaria.

Headers requeridos:
http
Authorization: Bearer {tu_api_key}
Content-Type: application/json
Scopes requeridos:
json
{
  "required_scope": "credits:execute"
}
📝 Ejemplos de Uso
Ejemplo 1: Modo Tradicional (Cuenta Registrada)
python
import requests

API_KEY = 'tu_api_key_aqui'
BASE_URL = 'https://points0.com/public-api'

headers = {
    'Authorization': f'Bearer {API_KEY}',
    'Content-Type': 'application/json'
}

# 1. Primero consulta tu capacidad disponible
response = requests.get(
    f'{BASE_URL}/credits/capacity',
    headers=headers
)

if response.status_code == 200:
    capacidad = response.json()
    disponible = capacidad['capacity']['volume_available']
    print(f"💰 Tienes ${disponible} disponible para créditos")

# 2. Ejecutar crédito a una cuenta registrada
payload = {
    'bank_account_id': 'ID_DE_TU_CUENTA_REGISTRADA',
    'amount': 100.00,
    'concept': 'Pago a proveedor'
}

response = requests.post(
    f'{BASE_URL}/credits/execute',
    json=payload,
    headers=headers
)

if response.status_code == 200:
    resultado = response.json()
    print(f"✅ Crédito ejecutado: {resultado}")
Ejemplo 2: Modo Manual (Datos Directos)
python
# Enviar crédito sin tener la cuenta registrada
payload = {
    'cedula_manual': 'V30552028',  # Cédula/RIF del beneficiario
    'account_number_manual': '01050123456789012345',  # Número de cuenta
    'bank_code_manual': '0105',  # Código del banco (ej: 0105 = Mercantil)
    'amount': 50.00,
    'concept': 'Reembolso de gastos'
}

response = requests.post(
    f'{BASE_URL}/credits/execute',
    json=payload,
    headers=headers
)

if response.status_code == 200:
    resultado = response.json()
    print(f"✅ Crédito manual ejecutado: {resultado}")
🎯 Casos de Uso Comunes para Merchants
1. Pagos a Proveedores
python
def pagar_proveedor(proveedor_id, monto, concepto):
    """Pagar a un proveedor usando cuenta registrada"""
    payload = {
        'bank_account_id': obtener_cuenta_proveedor(proveedor_id),
        'amount': monto,
        'concept': f"Pago {concepto} - {datetime.now().strftime('%Y%m%d')}"
    }
    
    return ejecutar_credito(payload)
2. Reembolsos a Clientes
python
def reembolsar_cliente(cliente_email, monto, datos_bancarios):
    """Reembolsar a cliente con datos bancarios manuales"""
    payload = {
        'cedula_manual': datos_bancarios['cedula'],
        'account_number_manual': datos_bancarios['cuenta'],
        'bank_code_manual': datos_bancarios['banco'],
        'amount': monto,
        'concept': f"Reembolso - Cliente: {cliente_email}"
    }
    
    return ejecutar_credito(payload)
3. Nómina de Empleados
python
def procesar_nomina(empleados):
    """Procesar nómina usando modo tradicional o manual"""
    resultados = []
    
    for empleado in empleados:
        if empleado['tiene_cuenta_registrada']:
            payload = {
                'bank_account_id': empleado['cuenta_id'],
                'amount': empleado['salario'],
                'concept': f"Nómina {empleado['nombre']}"
            }
        else:
            payload = {
                'cedula_manual': empleado['cedula'],
                'account_number_manual': empleado['cuenta'],
                'bank_code_manual': empleado['banco'],
                'amount': empleado['salario'],
                'concept': f"Nómina {empleado['nombre']}"
            }
        
        resultado = ejecutar_credito(payload)
        resultados.append(resultado)
    
    return resultados