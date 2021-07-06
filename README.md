# SistemasEmbebidos1C2021
# Trabajo Práctico N°1
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