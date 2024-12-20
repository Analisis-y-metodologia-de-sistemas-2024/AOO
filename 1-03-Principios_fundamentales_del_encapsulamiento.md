# Encapsulamiento en Python: Guía Práctica

## 1. Fundamentos del Encapsulamiento

El encapsulamiento en Python se implementa mediante convenciones y decoradores:
- Atributos privados: prefijo `__`
- Atributos protegidos: prefijo `_`
- Properties para control de acceso
- Métodos de acceso controlado

## 2. Implementación Práctica

### 2.1 Sistema Bancario

```python
from typing import Optional, List
from datetime import datetime
from decimal import Decimal

class CuentaBancaria:
    def __init__(self, numero: str, titular: str, saldo_inicial: Decimal = Decimal('0.0')):
        self.__numero = numero
        self.__titular = titular
        self.__saldo = saldo_inicial
        self.__transacciones: List[dict] = []
    
    @property
    def saldo(self) -> Decimal:
        """Obtiene el saldo actual de la cuenta."""
        return self.__saldo
    
    @property
    def titular(self) -> str:
        """Obtiene el nombre del titular."""
        return self.__titular
    
    def depositar(self, monto: Decimal) -> bool:
        """Realiza un depósito en la cuenta."""
        if monto <= Decimal('0'):
            raise ValueError("El monto debe ser positivo")
        
        self.__saldo += monto
        self.__registrar_transaccion("depósito", monto)
        return True
    
    def retirar(self, monto: Decimal) -> bool:
        """Realiza un retiro de la cuenta."""
        if monto <= Decimal('0'):
            raise ValueError("El monto debe ser positivo")
        
        if monto > self.__saldo:
            raise ValueError("Saldo insuficiente")
        
        self.__saldo -= monto
        self.__registrar_transaccion("retiro", monto)
        return True
    
    def __registrar_transaccion(self, tipo: str, monto: Decimal) -> None:
        """Método privado para registrar transacciones."""
        self.__transacciones.append({
            'fecha': datetime.now(),
            'tipo': tipo,
            'monto': monto,
            'saldo_resultante': self.__saldo
        })
    
    def obtener_historial(self) -> List[dict]:
        """Obtiene una copia del historial de transacciones."""
        return self.__transacciones.copy()

```

### 2.2 Sistema de Gestión de Empleados

```python
from dataclasses import dataclass
from datetime import date
from decimal import Decimal
from typing import Optional

@dataclass
class DatosPersonales:
    """Clase para encapsular datos personales sensibles."""
    __slots__ = ['__nombre', '__fecha_nacimiento', '__dni']
    
    def __init__(self, nombre: str, fecha_nacimiento: date, dni: str):
        self.__nombre = nombre
        self.__fecha_nacimiento = fecha_nacimiento
        self.__dni = dni
    
    @property
    def nombre(self) -> str:
        return self.__nombre
    
    @property
    def edad(self) -> int:
        hoy = date.today()
        return hoy.year - self.__fecha_nacimiento.year - (
            (hoy.month, hoy.day) < 
            (self.__fecha_nacimiento.month, self.__fecha_nacimiento.day)
        )

class Empleado:
    def __init__(self, datos: DatosPersonales, salario: Decimal):
        self.__datos = datos
        self.__salario = salario
        self.__evaluaciones: List[dict] = []
    
    @property
    def nombre(self) -> str:
        return self.__datos.nombre
    
    @property
    def salario(self) -> Decimal:
        return self.__salario
    
    def ajustar_salario(self, nuevo_salario: Decimal, motivo: str) -> None:
        """Ajusta el salario del empleado con justificación."""
        if nuevo_salario <= Decimal('0'):
            raise ValueError("El salario debe ser positivo")
        
        self.__registrar_cambio_salario(self.__salario, nuevo_salario, motivo)
        self.__salario = nuevo_salario
    
    def __registrar_cambio_salario(
        self, 
        anterior: Decimal, 
        nuevo: Decimal, 
        motivo: str
    ) -> None:
        """Registro privado de cambios salariales."""
        self.__evaluaciones.append({
            'fecha': date.today(),
            'salario_anterior': anterior,
            'salario_nuevo': nuevo,
            'motivo': motivo
        })
```

## 3. Pruebas Unitarias

```python
import unittest
from decimal import Decimal
from datetime import date

class TestCuentaBancaria(unittest.TestCase):
    def setUp(self):
        self.cuenta = CuentaBancaria("123", "Juan Pérez", Decimal('1000.0'))
    
    def test_deposito_valido(self):
        self.assertTrue(self.cuenta.depositar(Decimal('500.0')))
        self.assertEqual(self.cuenta.saldo, Decimal('1500.0'))
    
    def test_retiro_valido(self):
        self.assertTrue(self.cuenta.retirar(Decimal('500.0')))
        self.assertEqual(self.cuenta.saldo, Decimal('500.0'))
    
    def test_retiro_invalido(self):
        with self.assertRaises(ValueError):
            self.cuenta.retirar(Decimal('1500.0'))

class TestEmpleado(unittest.TestCase):
    def setUp(self):
        datos = DatosPersonales("Ana López", date(1990, 5, 15), "12345678")
        self.empleado = Empleado(datos, Decimal('50000.0'))
    
    def test_ajuste_salario(self):
        self.empleado.ajustar_salario(Decimal('55000.0'), "Promoción")
        self.assertEqual(self.empleado.salario, Decimal('55000.0'))

```

## 4. Mejores Prácticas

1. **Diseño**:
   - Usar properties en lugar de getters/setters tradicionales
   - Minimizar la exposición de atributos
   - Separar datos sensibles en clases específicas

2. **Implementación**:
   - Validar datos en setters y constructores
   - Usar type hints para mejor documentación
   - Emplear dataclasses cuando sea apropiado

3. **Seguridad**:
   - Proteger datos sensibles con atributos privados
   - Controlar acceso mediante métodos específicos
   - Validar todos los inputs

## 5. Ejercicios Propuestos

1. **Sistema de Expedientes Médicos**:
   - Implementar una clase para gestionar historiales médicos
   - Asegurar privacidad de datos sensibles
   - Permitir acceso controlado según roles

2. **Gestor de Contraseñas**:
   - Crear sistema para almacenar credenciales
   - Implementar encriptación de datos
   - Gestionar acceso mediante autenticación

## Conclusión

El encapsulamiento en Python, aunque menos estricto que en otros lenguajes, es fundamental para crear código seguro y mantenible. La clave está en usar las convenciones y herramientas disponibles de manera efectiva.