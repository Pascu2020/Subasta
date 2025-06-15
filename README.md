# 🏆 Smart Contract de Subastas

## 📋 Descripción General

Este proyecto implementa un smart contract en Solidity para realizar subastas tradicionales (incrementando el precio por al menos el 5%) con reembolsos parciales, extensiones automáticas de tiempo (10 minutos) y cobro de comisiones del 2%.

## 🚀 Características Principales

- ✅ **Subasta**: Precio ascendente hasta finalización
- ✅ **Incremento Mínimo**: 5% sobre la mejor oferta actual
- ✅ **Extensión Automática**: +10 minutos si oferta en últimos 10 minutos
- ✅ **Reembolsos Parciales**: Durante subasta activa
- ✅ **Sistema de Comisiones**: 2% sobre reembolsos no ganadores
- ✅ **Gestión de Propietario**: Control completo del ciclo de subasta

## 🏗️ Arquitectura del Contrato

### Estados de la Subasta
```solidity
enum EstadoSubasta { 
    Inactiva,    // Subasta no iniciada
    Activa,      // Subasta en curso
    Finalizada   // Subasta terminada
}
```

### Variables Principales

| Variable | Tipo | Descripción |
|----------|------|-------------|
| `owner` | address | Propietario del contrato |
| `precioMinimo` | uint256 | Precio mínimo de inicio |
| `mejorOferta` | uint256 | Mayor oferta actual |
| `mejorOferente` | address | Dirección del mejor oferente |
| `estadoSubasta` | EstadoSubasta | Estado actual de la subasta |
| `depositosTotales` | mapping | Depósitos por usuario |

## ⚙️ Funciones Principales

### 🎮 Funciones del Owner

#### `iniciarSubasta()`
- **Descripción**: Inicia la subasta desde estado Inactiva
- **Acceso**: Solo owner
- **Eventos**: `SubastaIniciada`

```solidity
function iniciarSubasta() external onlyOwner subastaInactiva
```

#### `finalizarSubasta()`
- **Descripción**: Finaliza la subasta manualmente
- **Acceso**: Solo owner
- **Eventos**: `SubastaFinalizada`

#### `transferirGanador(address _destinatario)`
- **Descripción**: Envía la oferta ganadora al destinatario especificado
- **Parámetros**: `_destinatario` - Dirección receptora
- **Acceso**: Solo owner
- **Eventos**: `TransferenciaGanador`

#### `retirarComisiones()`
- **Descripción**: Retira comisiones acumuladas (2% de reembolsos)
- **Acceso**: Solo owner
- **Eventos**: `ComisionRetirada`

#### `cambiarPropietario(address _nuevoOwner)`
- **Descripción**: Transfiere la propiedad del contrato
- **Parámetros**: `_nuevoOwner` - Nueva dirección propietaria
- **Acceso**: Solo owner
- **Eventos**: `PropietarioCambiado`

### 💰 Funciones de Usuarios

#### `ofertar()`
- **Descripción**: Realiza una oferta en la subasta
- **Payable**: Sí (requiere envío de ETH)
- **Validaciones**: 
  - Subasta activa
  - Depósito total ≥ (mejor_oferta × 1.05)
- **Eventos**: `NuevaOferta`, `ExtensionTiempo` (si aplica)

```solidity
function ofertar() external payable subastaActiva
```

#### `retirarExceso(uint256 _monto)`
- **Descripción**: Retira exceso de depósito durante subasta activa
- **Parámetros**: `_monto` - Cantidad a retirar
- **Validaciones**: 
  - Subasta activa
  - Monto ≤ exceso disponible
- **Eventos**: `ReembolsoRealizado`

#### `procesarReembolsos()`
- **Descripción**: Procesa reembolsos a oferentes no ganadores
- **Acceso**: Cualquiera (después de finalización)
- **Comisión**: 2% descontado automáticamente
- **Eventos**: `ReembolsoRealizado`

### 📊 Funciones de Consulta

#### `obtenerGanador()`
```solidity
function obtenerGanador() external view returns (address ganador, uint256 montoGanador)
```

#### `obtenerOfertas()`
```solidity
function obtenerOfertas() external view returns (address[] memory oferentes, uint256[] memory montos)
```

#### `obtenerEstadoSubasta()`
```solidity
function obtenerEstadoSubasta() external view returns (EstadoSubasta estado, uint256 tiempoRestante)
```

#### `obtenerDepositoDisponible(address _usuario)`
```solidity
function obtenerDepositoDisponible(address _usuario) external view returns (uint256 disponible)
```

#### `obtenerInfoCompleta()`
```solidity
function obtenerInfoCompleta() external view returns (
    EstadoSubasta estado,
    uint256 tiempoRestante,
    uint256 mejorOfertaActual,
    address mejorOferenteActual,
    uint256 totalOfertasRealizadas,
    uint256 comisiones
)
```

## 📡 Eventos

### `NuevaOferta`
```solidity
event NuevaOferta(
    address indexed oferente, 
    uint256 monto, 
    uint256 timestamp,
    uint256 numeroOferta
);
```

### `SubastaIniciada`
```solidity
event SubastaIniciada(
    uint256 duracion, 
    uint256 precioMinimo,
    uint256 tiempoFinal
);
```

### `SubastaFinalizada`
```solidity
event SubastaFinalizada(
    address indexed ganador, 
    uint256 montoGanador, 
    bool conOfertas,
    uint256 timestamp
);
```

### `ReembolsoRealizado`
```solidity
event ReembolsoRealizado(
    address indexed oferente, 
    uint256 monto,
    bool esComision
);
```

### `ExtensionTiempo`
```solidity
event ExtensionTiempo(
    uint256 nuevoTiempoFinal,
    address causante
);
```

## 🔒 Modificadores de Seguridad

### `onlyOwner`
- Restringe acceso solo al propietario del contrato

### `subastaActiva`
- Verifica que la subasta esté en estado Activa
- Verifica que no haya expirado el tiempo límite

### `subastaFinalizada`
- Verifica que la subasta haya finalizado
- Permite ejecución si ha expirado automáticamente

### `noReentrant`
- Previene ataques de reentrada en funciones críticas

## 🧮 Lógica de Negocio

### Cálculo de Ofertas Válidas
```
Oferta Mínima Requerida = Mejor Oferta Actual × 1.05
```

### Sistema de Depósitos
- Los usuarios pueden depositar más ETH del mínimo requerido
- Pueden retirar el exceso durante la subasta (si no son el mejor oferente)
- El mejor oferente puede retirar solo el exceso sobre su oferta actual

### Extensión Automática
- Si una oferta válida se realiza en los últimos 10 minutos
- La subasta se extiende automáticamente 10 minutos más
- Sin límite en el número de extensiones

### Sistema de Comisiones
- 2% sobre todos los reembolsos a oferentes no ganadores
- Las comisiones se acumulan en el contrato
- Solo el owner puede retirar las comisiones

## 🛡️ Seguridad

### Validaciones Implementadas
- Verificación de estados de subasta
- Validación de montos y direcciones
- Protección contra reentrada
- Verificación de permisos de owner



