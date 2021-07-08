# SistemasEmbebidos1C2021
# Trabajo Práctico N°1

# GPIO

1. REGISTROS DEL MICRO CONTROLADOR

IO

Los registros que controlan la GPIO van desde la posición 0x400F 4000 a la hasta la 0x400F 6320. En este sector se encuentran 9 grupos: B, W, DIR, MASK, PIN, MPIN, SET, CLR, NOT.

En el caso de los grupos B y W, Modificar el registro BX modifica el pin Z del puerto Y donde X = Y\*32 + Z. Para todos los otros registros Modificar el bit Z del registro Y modifica el pin Z del puerto Y.

B: Son registros de 8 bits. Hay 256 de estos, cada uno controla un pin. De cada uno de estos registros solo se van a leer o escribir el bit menos significativo. Un 1 en ese bit corresponde a estado alto. El offset va desde 0x0000 hasta 0x00FF.

W: Son registros de 32 bits. Hay 256 de estos, cada uno controla un pin. En la lectura se lee 0x FFFF FFFF si esta en alto y 0 si está en bajo. En la escritura, se escribe 0 para que esté en estado bajo y cualquier otro valor para que está en alto. El offset va desde 0x1000 hasta 0x13FC.

DIR: Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Escribir un 1 en un bit pone a un pin como salida y si se escribe un 0 se establece como entrada. El offset va desde 0x2000 hasta 0x201C.

MASK: Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Poner un 1 en el bit X del registro MASKY hace que no se pueda leer ni escribir en el bit X del registro MPINY. El offset va desde 0x2080 hasta 0x209C.

PIN: Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Se puede leer o escribir el estado de un pin. Un 1 corresponde a estado alto. El offset va desde 0x2100 hasta 0x211C.

MPIN: Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Si El bit correspondiente de MASK está en 1, entonces el bit no se puede escribir y siempre se lee como 0. Si el bit de MASK está en 0 entonces funciona como PIN. El offset va desde 0x2180 hasta 0x219C.

SET: Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Escribir un 1 setea el pin y escribir un 0 no genera cambios. No se puede leer estos registros. El offset va desde 0x2200 hasta 0x221C.

CLR: Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Al leer se lee el estado del pin. Escribir un 1 pone el pin en bajo y escribir un 0 no genera cambios. El offset va desde 0x2280 hasta 0x229C.

NOT: Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Escribir un 1 togglea el estado del pin y escribir un 0 no genera cambios. No se puede leer estos registros. El offset va desde 0x2300 hasta 0x231C.

INTERRUPCIONES POR PIN

Existen dos tipos de interrupciones que se pueden generar por GPIO, las interrupciones asociadas a pin, análogas a INT0 e INT1 en el atmega328p, y las interrupciones asociadas por grupo, análogas PCINT0 a PCINT2 en el atmega328p.

Existen 8 interrupciones de pines de GPIO, PIN\_INT0 a PIN\_INT8. Al contrario de lo que ocurre con el atmega328p uno puede elegir que pines de GPIO son los que van a formar parte de este tipo de interrupción.

Las interrupciones por pin estás asociadas a 2 grupos de registro, uno asociado a SCU y otro a GPIO.

Los registros asociados al SCU permiten asociar el numero de interrupción al pin de GPIO y existen 2 registros: PINTSEL0 y PINTSEL1. La dirección base de estos registros es 0x4008 6000 y el offset de estos registros son 0xE00 y 0xE04 respectivamente.

Tabla 1.1:

| Registro | Bits | Función |
| --- | --- | --- |
| PINTSEL0 | 4:0 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT0. |
|  | 7:5 | Selecciona el número de port de GPIO para la interrupción PIN\_INT0. |
|  | 8:12 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT1. |
|  | 13:15 | Selecciona el número de port de GPIO para la interrupción PIN\_INT1. |
|  | 16:20 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT2. |
|  | 21:23 | Selecciona el número de port de GPIO para la interrupción PIN\_INT2. |
|  | 24:28 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT3. |
|  | 29:31 | Selecciona el número de port de GPIO para la interrupción PIN\_INT3. |
| PINTSEL1 | 4:0 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT4. |
|  | 7:5 | Selecciona el número de port de GPIO para la interrupción PIN\_INT4. |
|  | 8:12 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT5. |
|  | 13:15 | Selecciona el número de port de GPIO para la interrupción PIN\_INT5. |
|  | 16:20 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT6. |
|  | 21:23 | Selecciona el número de port de GPIO para la interrupción PIN\_INT6. |
|  | 24:28 | Selecciona el número de pin de GPIO para la interrupción PIN\_INT7. |
|  | 29:31 | Selecciona el número de port de GPIO para la interrupción PIN\_INT7. |

Los registros asociados al GPIO lo que haces es definir las características de las interrupciones. Estos registros son registros de 32 bits en donde el bit n de cada registro está asociada a una característica de la interrupción PIN\_INTn. La dirección base de estos registros es 0x4008 7000.

Tabla 1.2:

| Registro | Tipo | offset | Función (valor del bit n) |
| --- | --- | --- | --- |
| ISEL | R/W | 0x00 | 0 activa interrupción por flanco, 1 activa por nivel. |
| IENR | R/W | 0x04 | Si es por nivel: 0 deshabilita interrupción, 1 habilita interrupción.Si es por flanco: 0 deshabilita interrupción por flanco ascendente, 1 habilita interrupción por flanco ascendente. |
| SIENR | WO | 0x08 | 0 no genera cambio.Si es por nivel: 1 habilita interrupción.Si es por flanco: 1 habilita interrupción por flanco ascendente. |
| CIENR | WO | 0x0C | 0 no genera cambio.Si es por nivel: 1 deshabilita interrupción.Si es por flanco: 1 deshabilita interrupción por flanco ascendente. |
| IENF | R/W | 0x10 | Si es por nivel: 0 setea el nivel de activación en LOW, setea el nivel de activación en HIGH.Si es por flanco: 0 deshabilita interrupción por flanco descendente, 1 habilita interrupción por flanco descendente. |
| SIENF | WO | 0x14 | 0 no genera cambio.Si es por nivel: 1 setea el nivel de activación en LOW.Si es por flanco: 1 habilita interrupción por flanco descendente. |
| CIENF | WO | 0x18 | 0 no genera cambio.Si es por nivel: 1 setea el nivel de activación en HIGH.Si es por flanco: 1 deshabilita interrupción por flanco descendente. |
| RISE | R/W | 0x1C | Lectura: 0 no se detectó ningún flanco ascendente desde que se limpió el bit, 1 se detectó un flanco ascendente.Escritura: 0 no genera cambio, 1 limpia el bit del registro. |
| FALL | R/W | 0x20 | Lectura: 0 no se detectó ningún flanco descendente desde que se limpió el bit, 1 se detectó un flanco descendente.Escritura: 0 no genera cambio, 1 limpia el bit del registro. |
| IST | R/W | 0x24 | Lectura: 0 no se espera una interrupción, 1 se espera interrupciónEscritura: 0 no genera cambios, 1 si es por nivel, cambia el nivel de activación (togglea), si es por flaco limpia los bits de los registros RISE y FALL. |

INTERRUPCIONES POR GRUPO

Las interrupciones por grupo se generan cuando una o todos los pines de un grupo, dependiendo de la configuración, envían la señal adecuada. Existen 2 interrupciones de este tipo: GINT0, GINT1. Ambas interrupciones presentan la misma estructura de registros. La diferencia radica en que la dirección base de GINT0 es 0x4008 8000 mientras que la de GINT1 es 0x4008 9000.

Tabla 1.3:

| Registro | offset | Función |
| --- | --- | --- |
| CTRL | 0x00 | Bit 0: leer un 0 indica que no hay interrupción pendiente mientras que leer un 1 indica que si la hay. Escribir un 0 no genera cambios y escribir un 1 pone en 0 el bitBit 1: 0 función OR, indica que uno de los pines del grupo tiene que cumplir la condición de trigger para que se genere la interrupción. 1 función AND, indica que todos los pines tienen que cumplir la condición.Bir 2: 0 la interrupción se genera por flanco, 1 la interrupción se genera por nivel |
| PORT\_POLn | 0x20 +n\*0x4 | Bit m: 0 el pin m del gpio port n se activa en LOW, 1 se activa en HIGH. |
| PORT\_ENAn | 0x40 +n\*0x4 | Bit m: 0 el pin m del gpio port n no contribuye a la interrupción, 1el pin contribuye a la interrupción. |

2. Estructura de datos que &quot;matchean&quot; el GPIO

IO

La estructura que marche los registros de GPIO con su número de puerto y pin es ```LPC_GPIO_T```, que se muestra en la figura 2.1.

![Figura GPIO 2.1](/GPIO/LPC_GPIO_T.jpg)

Figura 2.1: Estructura que representa los registros GPIO.

En la figura 2.1 se puede observar que IO corresponde a los registros de lectura y escritura y O corresponde a los de solo escritura. También se puede observar que cada grupo se agregan elementos de más para que cada agrupo posea el offset correcto.

También existe una etiqueta ```LPC_GPIO_PORT_BASE``` a la que se le asigna el valor 0x400F4000, que corresponde a la dirección de inicio de los registros GPIO. Además, se encuentra la línea ```#define LPC_GPIO_PORT ((LPC_GPIO_T *) LPC_GPIO_PORT_BASE)``` que genera un puntero a ```LPC_GPIO_T```con el valor de la dirección de inicio.

Como es necesario saber el numero de puerto y pin para poder gestionar el GPIO, existen otras 3 estructuras que sirven para obtener el numero de pin y puerto a partir de la etiqueta que le asigna la EDU-CIAA a cada pin del microcontrolador.

La primera estructura es un ```enum gpioMap\_t```, que se encuentra en el archivo sapi\_peripheral\_map.h. enum. Lo que hace es asignarle a cada etiqueta de la EDU-CIAA un valor de posición. La segunda estructura es un vector de ```pinInitGpioLpc4337_t```, llamado gpioPinsInit, que se encuentra en el archivo sapi\_gpio.c. Lo que hace es obtener un ```pinInitGpioLpc4337_t```, a partir de la posición asignada a cada etiquta. Por último, el ```pinInitGpioLpc4337_t``` contiene 5 ```int8_t```, correspondientes al puerto físico, pin físico, número de función asignada a gpio, puerto gpio y pin gpio. En conjunto estas 3 estructuras lo que hacen en conjunto es apartir de una estiqueta poder manejar la SCU para poner la función gpio y poder manejar la gpio. En las figuras 2.2 a 2.4 se puede observar la estructura ```de pinInitGpioLpc4337_t```.

![Figura GPIO 2.2](/GPIO/pinInitGpioLpc4337_t.jpg)

Figura 2.2: Estructura de pinInitGpioLpc4337\_t

![Figura GPIO 2.3](/GPIO/gpioInitLpc4337_t.jpg)

Figura 2.3: estructura de gpioInitLpc4337\_t

![Figura GPIO 2.4](/GPIO/pinInitLpc4337_t.jpg)

Figura 2.4: estructura de pinInitLpc4337\_t

INTERRUPCIONES POR PIN

La estructura que representa los registros de la SCU es ```LPC_SCU_T``` que .se muestra en la figura 2.5, donde lo importante para las interrupciones es el atributo ``PINTSEL``.

![Figura GPIO 2.5](/GPIO/LPC_SCU_T.jpg)

Figura 2.5: estructura que representa los registros en SCU.

Para poder acceder a estos registros se encuentra la etiqueta ``LPC_SCU_BASE`` que posee el valor 0x40086000 que es la dirección base de dichos registros. También existe la linea  ``#define LPC_SCU ((LPC_SCU_T*) LPC_SCU_BASE)``  que genera un puntero a ```LPC_SCU_T``` con el valor de la dirección de inicio.

La estructura que representa los registros de GPIO para el manejo de interrupción por pin es ``LPC_PIN_INT_T``, que se muestra en la figura 2.6.

![Figura GPIO 2.6](/GPIO/LPC_PIN_INT_T.jpg)

Figura 2.6: estructura que representa los registros de interrupción por pin.

Las líneas
```c 
#define LPC_PIN_INT_BASE 0x40087000 
``` 
y
``` c
#define LPC_GPIO_PIN_INT ((LPC_PIN_INT_T*) LPC_PIN_INT_BASE)
``` 
Generan un puntero a la dirección inicial de dichos registros.

INTERRUPCIONES POR GRUPO

La estructura que representa los registros de GPIO para el manejo de interrupción por grupo es LPC\_GPIOGROUPINT\_T, que se muestra en la figura 2.7.

![Figura GPIO 2.7](/GPIO/LPC_GPIOGROUPINT_T.jpg)

Figura 2.7: estructura que representa los registros de interrupción por grupo.

En el caso del grupo 0, las etiquetas

```c
#define LPC_GPIO_GROUP_INT0_BASE 0x40088000
``` 
y
```c
#define LPC_GPIOGROUP ((LPC_GPIOGROUPINT_T*) LPC_GPIO_GROUP_INT0_BASE)
```

generan un puntero a la dirección inicial de dichos registros.

En el caso del grupo 1 está la etiqueta
```c
#define LPC_GPIO_GROUP_INT1_BASE 0x40089000
```

Que marca el inicio de los registros de este grupo.

3. Primitivas

IO: Nivel Chip (NXP)
```c
void Chip_GPIO_Init(LPC_GPIO_T *pGPIO)
```
Función: inicializa el bloque GPIO. pGPIO debe contener la dirección de inicio de los registros de gpio.

Archivo: gpio\_18xx\_43xx.h

```c
void Chip_GPIO_DeInit(LPC_GPIO_T *pGPIO)
```
Función: des inicializa el bloque GPIO.

Archivo: gpio\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIO_SetDir(LPC_GPIO_T *pGPIO, uint8_t portNum, uint32_t bitValue, uint8_t out)
```
Función: En base a los bits que están en 1 en bitvalue pone los pines como entrada (out = 0), o salida (out = 1).

Archivo: gpio\_18xx\_43xx.h

```c
STATIC INLINE void Chip_GPIO_SetPinState(LPC_GPIO_T *pGPIO, uint8_t port, uint8_t pin,bool setting)
```
Función: Escribe el estado del pin modificando el grupo B.

Archivo: gpio\_18xx\_43xx.h
```c
STATIC INLINE bool Chip_GPIO_ReadPortBit(LPC_GPIO_T *pGPIO, uint32_t port, uint8_t pin)
```
Función: Lee el estado del pin a partir del grupo B de registros.

Archivo: gpio\_18xx\_43xx.h

Funciones Útiles:
```c
STATIC INLINE void Chip_GPIO_SetPinToggle(LPC_GPIO_T *pGPIO, uint8_t port, uint8_t pin)
```
Función: Togglea el valor de un pin.

Archivo: gpio\_18xx\_43xx.h

IO: Nivel Sapi
```c
static void gpioObtainPinInit ( gpioMap_t pin, int8_t *pinNamePort, int8_t *pinNamePin, int8_t *func, int8_t *gpioPort, int8_t *gpioPin )
```
Función: obtiene el valor de pin físico, puerto físico, función gpio, pin gpio y puerto gpio a partir de la etiqueta de la EDU-CIAA.

Archivo: sapi\_gpio.c

De uso común:
```c
bool_t gpioInit( gpioMap_t pin, gpioInit_t config )
```
Función: Si se envía &quot;ENABLE&quot; en config, activa la función de gpio del microcontrolador. En otro caso establece la función gpio del pin seleccionado, en base a config lo puede poner como salida o como entrada. Si es entrada se puede poner con o sin resistencia de pullup y/o pulldown. Devuelve false si si pasan VCC o GND como pin o si la configuración no es correcta.

Archivo: sapi\_gpio.c
```c
bool_t gpioWrite( gpioMap_t pin, bool_t value )
```
Función: Escribe el valor de un pin. Se debe haber puesto como salida con **gpioInit** antes de usarla.

Archivo: sapi\_gpio.c
```c
bool_t gpioToggle( gpioMap_t pin )
```
Función: Alterna el valor del pin si está como salida. Se debe haber puesto como salida con **gpioInit** antes de usarla.

Archivo: sapi\_gpio.c
```c
bool_t gpioRead ( gpioMap_t pin )
```
Función: Lee el valor del pin. Se debe haber inicializado con **gpioInit** antes de usarla.

Archivo: sapi\_gpio.c

INTERRUPCIONES POR PIN: Nivel Chip (NXP)
```c
STATIC INLINE void Chip_SCU_GPIOIntPinSel(uint8_t PortSel, uint8_t PortNum, uint8_t PinNum)
```
Función: Asocia una interrupción por pin a un pin de la GPIO.

Archivo: scu\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_Init(LPC_PIN_INT_T *pPININT)
```
Función: Inicializa el bloque de interrupciones por pin. Se debe usar despué de la función Chip\_GPIO\_Init.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_DeInit(LPC_PIN_INT_T *pPININT)
```
Función: des inicializa el bloque de interrupciones por pin.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_SetPinModeEdge(LPC_PIN_INT_T *pPININT, uint32_t pins)
```
Función: setea el trigger por flanco.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_SetPinModeLevel(LPC_PIN_INT_T *pPININT, uint32_t pins)
```
Función: setea el trigger por nivel.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_EnableIntHigh(LPC_PIN_INT_T *pPININT, uint32_t pins)
```
Función: habilita interrupción por flanco ascendente o por nivel.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_DisableIntHigh(LPC_PIN_INT_T *pPININT, uint32_t pins)
```
Función: deshabilita interrupción por flanco ascendente o por nivel.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_EnableIntLow(LPC_PIN_INT_T *pPININT, uint32_t pins)
```
Función: habilita interrupción por flanco descendente o pone el nivel en LOW para interrupción por nivel.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_PININT_DisableIntLow(LPC_PIN_INT_T *pPININT, uint32_t pins)
```
Función: deshabilita interrupción por flanco descendente o pone el nivel en HIGH para interrupción por nivel.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE uint32_t Chip_PININT_GetIntStatus(LPC_PIN_INT_T *pPININT)
```
Función: obtiene el estado de la interrupción.

Archivo: pinint\_18xx\_43xx.h

INTERRUPCIONES POR GRUPO: Nivel Chip (NXP)
```c
STATIC INLINE bool Chip_GPIOGP_GetIntStatus(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group)
```
Función: obtiene el estado de la interrupción.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_SelectOrMode(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group)
```
Función: pone la interruption en modo OR.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_SelectAndMode(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group)
```
Función: pone la interruption en modo AND.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_SelectEdgeMode(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group)
```
Función: pone el modo de activación en flanco.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_SelectLevelMode(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group)
```
Función: pone el modo de activación en nivel.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_SelectLowLevel(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group, uint8_t port, uint32_t pinMask)
```
Función: setea el modo de activación en estado LOW.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_SelectHighLevel(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group, uint8_t port, uint32_t pinMask)
```
Función: setea el modo de activación en estado HIGH.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_DisableGroupPins(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group, uint8_t port, uint32_t pinMask)
```
Función: Desasocia un pin a la interrupción.

Archivo: pinint\_18xx\_43xx.h
```c
STATIC INLINE void Chip_GPIOGP_EnableGroupPins(LPC_GPIOGROUPINT_T *pGPIOGPINT, uint8_t group, uint8_t port, uint32_t pinMask)
```
Función: Asocia un pin a la interrupción.

Archivo: pinint\_18xx\_43xx.h

4. Diagrama

![Figura GPIO 4.1](/GPIO/GPIO_resumen.jpg)

---
# Index

- [TIMERS](#TIMERS)
- [CLOCK](#CLOCK)
- [DELAY](#DELAY)

---

# TIMERS

## General

Cada Timer/Counter está diseñado para contar ciclos del clock periférico (PCLK) o de un clock externo. Pueden opcionalmente generar interrupciones y otras acciones en valores del timer específicos, basados en los match registers. También incluye 4 entradas de capturas para “capturar” el valor del timer cuando ocurre una transición de la señal de entrada, y opcionalmente generar una interrupción.

## Características

* 4 Timers/Counters de 32-bits con un Prescaler programable de 32-bits cada uno.
* 4 32-bits Capture channel por cada Timer que pueden tomar un snapshot del valor del timer con una transición de la señal de entrada (input signal). Opcionalmente puede generar una interrupción.
* 4 32-bits Match Registers que pueden permitir:
    * Generar interrupción
    * Detener el Timer y opcionalmente generar interrupción
    * Resetear el Timer y opcionalmente generar interrupción.
* 4 Salidas externas (External outputs) correspondientes a cada Match Register con la posibilidad con setear en 0, 1 y togglear la salida externa en caso de suceder el match. 


## Registros Base de los Timers

| TIMER         | REGISTRO BASE |
| ------------- |:-------------:|
| TIMER0        | 0x 4008 4000  |
| TIMER1        | 0x 4008 5000  |
| TIMER2        | 0X 400C 3000  |
| TIMER3        | 0x 400C 4000  |


## Registros de los Timers

| SIGLA | NOMBRE	       	       | OFFSET  | DESCRIPCIÓN	 |
| -----	|:------------------------:|:-------:|-------------|
| IR    | Interrupt Register       | 0x 000  | Interrupt flags para los match y captura de los timers. 1 = interrupción generada. |
| TCR   | Time Control Register    | 0x 004  | Activa y desactiva las operaciones de los Timer/Counter. |
| TC    | Timer Counter            | 0x 008  | Registro de 32-bits. Incrementa cuando el prescale counter llega a su máximo valor. Este evento no genera una interrupción. Genera un overflow. |
| PR    | Prescale Register        | 0x 00C  | Registro de 32-bits. Especifica el máximo valor del Prescale Counter. |
| PC    | Prescale Counter         | 0x 010  | Registro de 32-bits. Controla la división del PCLK por una constante antes de ser aplicado al Timer Counter. |
| MCR   | Match Control Register   | 0x 014  | Controla qué operaciones son realizadas cuando uno de los Match Registers metchea el Timer Counter (estas son: Generación de interrupción MR y Reset del TC). |
| MR0   | Match Register 0         | 0x 018  | Registro de 32-bits. Especifica el valor para matchear el Timer Counter |
| MR1   | Match Register 1         | 0x 01C  | Idem |
| MR2   | Match Register 2         | 0x 020  | Idem |
| MR3   | Match Register 3         | 0x 024  | Idem |
| CCR   | Capture Control Register | 0x 028  | Controla si uno de las 4 capturas es cargado con el valor del Timer Counter cuando el capture event ocurre y si una interrupción es generada por el capture evento. |
| CR0   | Capture Register 0       | 0x 02C  | Valor del timer counter capture |
| CR1   | Capture Register 1       | 0x 030  | Idem |
| CR2   | Capture Register 2       | 0x 034  | Idem |
| CR3   | Capture Register 3       | 0x 038  | Idem |
| EMR   | External Match Register  | 0x 03C  | Controla y muestra el estado de los external match pins. |
| CTCR  | Count Control Register   | 0x 070  | Usado para seleccionar entre modo Timer o modo Counter (en este modo se selecciona el pin y el flanco para contar). Timer Mode (Cuenta cada flanco asc. Del PCLK). Counter Mode rising/falling/both Edge. |

## Diagrama de bloques


## Primitivas y structs - Timer - Nivel Sapi

**Ubicación:** firmware_v3/libs/sapi/sapi_v0.6.2/soc/peripherals/src/**sapi_timer.c**

---
##### Timer_Init()
```c
void Timer_Init( uint8_t timerNumber , uint32_t ticks, callBackFuncPtr_t voidFunctionPointer ){
/* Enable timer clock  
and reset it */
  	Chip_TIMER_Init();
   	Chip_RGU_TriggerReset();
/* Update the defalut function pointer name of the Compare match 0*/
   	timer_dd[timerNumber].timerCompareMatchFunctionPointer

/* Initialize compare match with the specified ticks (number of counts needed to clear the match counter) */
  	Chip_TIMER_MatchEnableInt( TIMERCOMPAREMATCH0 );
  	Chip_TIMER_SetMatch(TIMERCOMPAREMATCH0, ticks );

/* Makes Timer Match 0 period the timer period*/
   	Chip_TIMER_ResetOnMatchEnable(TIMERCOMPAREMATCH0 );

/*Enable timer*/
  	Chip_TIMER_Enable();

/* Enable timer interrupt */
  	NVIC_SetPriority();
  	NVIC_EnableIRQ();
  	NVIC_ClearPendingIRQ();
}	
```

* **Definición:** Inicializa el Timer periférico.
* **timerNumber:** Corresponde al Timer (0, 1, 2 o 3).
* **ticks:** Números de ticks necesarios para finalizar el ciclo.
* **voidFunctionPointer:** Ptr a función a ejecutar al final del ciclo.

---
##### Timer_DeInit()
```c
void Timer_DeInit( uint8_t timerNumber ){
	NVIC_DisableIRQ(timer_sd[timerNumber].IRQn);
   	Chip_TIMER_Disable(timer_sd[timerNumber].name);
   	Chip_TIMER_DeInit(timer_sd[timerNumber].name);

}
```

* **Definición:** desinicializa el Timer periférico.
* **timerNumber:** Corresponde al Timer (0, 1, 2 o 3).

---
##### Timer_microsecondsToTicks()
```c
uint32_t Timer_microsecondsToTicks( uint32_t uS ){
	uS*(LPC4337_MAX_FREC/1000000);
}
```

* **Definición:** Convierte el argumento en microsegundos a número de ticks. 
* **Us:** microsegundos a convertir en ticks.
* Puede ser usado en el segundo parámetro de la función Timer_Init().

---
##### Timer_EnableCompareMatch()
```c
void Timer_EnableCompareMatch( uint8_t timerNumber, uint8_t compareMatchNumber, uint32_t ticks, callBackFuncPtr_t voidFunctionPointer ){
	Chip_TIMER_ClearMatch();
  	Chip_TIMER_MatchDisableInt();
  }
```

-	**Definición:** Habilita un compare match in un timer.
-	**compareMatchNumber:** Compare match (number: 1 2 o 3).
-	**ticks:** valor de comparación en número de ticks.

---
##### Timer_DisableCompareMatch()
```c
void Timer_DisableCompareMatch( uint8_t timerNumber, uint8_t compareMatchNumber ){
	Chip_TIMER_ClearMatch();
  	Chip_TIMER_MatchDisableInt();
}
```

*	**Definición:** Desabilita  el compare match (number).
*	**compareMatchNumber:** Compare match (number: 1 2 o 3).


## Primitivas y structs - Timer - Nivel Chip

**Ubicación:** firmware_v3/libs/lpc_open/lpc_chip_43xx/src/**timer_18xx_43xx.c**

##### LPC_TIMER_T
```c
typedef struct {					/*!< TIMERn Structure       */
__IO uint32_t IR;				
__IO uint32_t TCR;			
__IO uint32_t TC;
__IO uint32_t PR;
__IO uint32_t PC;			/*!< Prescale Counter. The 32 bit PC is a counter which is incremented to the value stored in PR. When the value in PR is reached, the TC is incremented and the PC is cleared. The PC is observable and controllable through the bus interface. */
__IO uint32_t MCR;			/*!< Match Control Register. The MCR is used to control if an interrupt is generated and if the TC is reset when a Match occurs. */
	__IO uint32_t MR[4];			
	__IO uint32_t CCR;	
	__IO uint32_t CR[4];
	__IO uint32_t EMR;	
	__I  uint32_t RESERVED0[12];
	__IO uint32_t CTCR;			
} LPC_TIMER_T;

```

---
##### Chip_TIMER_Init()
```c
void Chip_TIMER_Init(LPC_TIMER_T *pTMR)
{
  Chip_Clock_Enable(Chip_TIMER_GetClockIndex(pTMR));
}
```

---
##### Chip_TIMER_DeInit()
```c
void Chip_TIMER_DeInit(LPC_TIMER_T *pTMR)
{
  Chip_Clock_Disable(Chip_TIMER_GetClockIndex(pTMR));
}
```

---
##### Chip_TIMER_Reset()
```c
void Chip_TIMER_Reset(LPC_TIMER_T *pTMR)
{
  uint32_t reg;

  /* Disable timer, set terminal count to non-0 */
  reg = pTMR->TCR;
  pTMR->TCR = 0;
  pTMR->TC = 1;

  /* Reset timer counter */
  pTMR->TCR = TIMER_RESET;

  /* Wait for terminal count to clear */
  while (pTMR->TC != 0) {}

  /* Restore timer state */
  pTMR->TCR = reg;
}
```

---
##### Chip_Chip_TIMER_GetClockIndex()
```c
/* Returns clock index for the peripheral block */
STATIC CHIP_CCU_CLK_T Chip_TIMER_GetClockIndex(LPC_TIMER_T *pTMR)
{
  CHIP_CCU_CLK_T clkTMR;

  if (pTMR == LPC_TIMER3) {
    clkTMR = CLK_MX_TIMER3;
  }
    else if (pTMR == LPC_TIMER2) {
    clkTMR = CLK_MX_TIMER2;
  }
    else if (pTMR == LPC_TIMER1) {
    clkTMR = CLK_MX_TIMER1;
  }
  else {
    clkTMR = CLK_MX_TIMER0;
  }

  return clkTMR;
}
```

# CLOCK

### Clock Generation Unit

| Base clock    | Branch clock  | operation frecuency |
|:-------------:|:-------------:|:-------------------:|
| BASE_M4_CLK   | CLK_M4_BUS    | up to 204 Mhz       |

Cada *CGU Base Clock* tiene diversos *clock branches*.

El *CCU* prende o apaga el clock a periféricos individuales.

### Clock Control Unit

Cada *clock branch* pueden ser activados o desactivados por los registros *CCU1* y *CCU2*.

| CCU           | REGISTRO BASE |
|:-------------:|:-------------:|
| CCU1          | 0x 4005 1000  |
| CCU2          | 0x 4005 2000  |

A continuación se presetan los *clock branches* del *CCU1*:

| BASE CLOCK    | BRANCH CLOCK  |
|:-------------:|:-------------:|
| BASE_M4_CLK   | CLK_M4_TIMER0 |
| BASE_M4_CLK   | CLK_M4_TIMER1 |
| BASE_M4_CLK   | CLK_M4_TIMER2 |
| BASE_M4_CLK   | CLK_M4_TIMER3 |

A continuación se presetan los resgistros *status* y *configuration* de cada *clock timer branch*:

| REGISTRO          | ADDRESS OFFSET|
|:-----------------:|:-------------:|
| CLK_M4_TIMER0_CFG | 0x 520        |
| CLK_M4_TIMER0_STAT| 0x 524        |
| CLK_M4_TIMER1_CFG | 0x 528        |
| CLK_M4_TIMER1_STAT| 0x 52C        |
| CLK_M4_TIMER2_CFG | 0x 618        |
| CLK_M4_TIMER2_STAT| 0x 61C        |
| CLK_M4_TIMER3_CFG | 0x 620        |
| CLK_M4_TIMER3_STAT| 0x 224        |


## Primitivas y structs - Clock - Nivel Chip

**Ubicación:** firmware_v3/libs/lpc_open/lpc_chip_43xx/src/**clock_18xx_43xx.c**

##### Chip_Clock_Enable()
```c
/* Enables a peripheral clock */
void Chip_Clock_Enable(CHIP_CCU_CLK_T clk)
{
  /* Start peripheral clock running */
  if (clk >= CLK_CCU2_START) {
    LPC_CCU2->CLKCCU[clk - CLK_CCU2_START].CFG |= 1;
  }
  else {
    LPC_CCU1->CLKCCU[clk].CFG |= 1;
  }
}

```

---
##### Chip_Clock_Disable()
```c
/* Disables a peripheral clock */
void Chip_Clock_Disable(CHIP_CCU_CLK_T clk)
{
  /* Stop peripheral clock */
  if (clk >= CLK_CCU2_START) {
    LPC_CCU2->CLKCCU[clk - CLK_CCU2_START].CFG &= ~1;
  }
  else {
    LPC_CCU1->CLKCCU[clk].CFG &= ~1;
  }
}
```

# DELAY

## Primitivas y structs - Delay - Nivel Sapi

**Ubicación:** firmware_v3/libs/sapi/sapi_v0.6.2/abstract_modules/src/**sapi_delay.c**

##### delay_t
```c
typedef struct{
   tick_t startTime;
   tick_t duration;
   bool_t running;
} delay_t;
```
**startime:** donde comienza el Delay. Utilizado por la función *delayRead()*.
**duration:** duración del Delay.
**running:** establece si el Delay está activado o no.

### Funciones Bloqueantes

##### delay()
```c
void delay( tick_t duration_ms )
{
   tick_t startTime = tickRead();
   while ( (tick_t)(tickRead() - startTime) < duration_ms/tickRateMS );
}
```

Inicializa el Delay en *duration_ms*.

### Funciones No Bloqueantes


##### delayInit()
```c
void delayInit( delay_t * delay, tick_t duration );
```
Inicializa el struct *delay* con el *duration* correspondiente. Se activa el Delay.

---
##### delayRead()
```c
bool_t delayRead( delay_t * delay )
```
Devuelve un 0 (*TRUE*) en caso de haber llegado al tiempo de Delay.

---
##### delayWrite()
```c
void delayWrite( delay_t * delay, tick_t duration );
```
Sobreescribe el atributo *duration*.

## Primitivas y structs - Tick - Nivel Sapi

**Ubicación:** firmware_v3/libs/sapi/sapi_v0.6.2/abstract_modules/src/**sapi_tick.c**

```c
typedef uint64_t tick_t;
```
```c
static tick_t tickCounter; //VARIABLE GLOBAL QUE CONTIENE EL TICK COUNT
```

---
##### tickInit()
```c
bool_t tickInit( tick_t tickRateMSvalue )
{
  SysTick_Config( SystemCoreClock * tickRateMSvalue / 1000 );
}
```
Inicializa el *tick Counter* y configura el *rate* entre 1 a 50 ms.

---
##### tickRead()
```c
tick_t tickRead( void );
```
Lee el *tick Counter*.

---
##### tickWrite()
```c
void tickWrite( tick_t ticks );
```
Sobreescribe el *tick counter*.

---
##### tickCallbackSet()
```c
bool_t tickCallbackSet( callBackFuncPtr_t tickCallback, void* tickCallbackParams );
```

---
##### tickPowerSet()
```c
void tickPowerSet( bool_t power );
```
Activa o desactiva el clock periférico que alimenta al Systick. Utilizada por la función *tickInit()*.

---
##### tickerCallback()
```c
void tickerCallback( void );
```
Llamada de la interrupción. Incremento del *tickCounter*. Ejecuta la función (ptr a función) en caso de estar configurado.

## Primitivas y structs - Systick - Nivel Chip

**Ubicación:** firmware_v3/libs/cmsis_core/inc/**core_cm0.h**

##### SysTick_Config()
```c
__STATIC_INLINE uint32_t SysTick_Config(uint32_t ticks);
```
Configuración del *System Tick*. Inicializa el *System Timer* y sus interrupciones y comienza el *System Tick Timer*. Counter está en *free running mode* para generar interrupciones periódicas.
**ticks:** número de ticks entre 2 interrupciones. Es utilizada por la función *tickInit()*.
