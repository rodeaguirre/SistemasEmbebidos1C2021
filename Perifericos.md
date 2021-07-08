# Sistemas embebidos - 1C2021

> ### Indice
> 1. SCU (Martín Anus)
>   - Introducción al esquemático de la CIAA
>   - Registros del SCU
>   - Estructura de datos que "matchean" el SCU
>   - Estratos de las primitivas
> 2. GPIO (Francisco Cufré)
>   - Registros del GPIO
>   - Estructura de datos que "matchean" el GPIO
>   - Estratos de las primitivas
> 3. Timer (Matias Viñas)
>   - Registros del GPIO
>   - Estructura de datos que "matchean" el GPIO
>   - Estratos de las primitivas
> 4. Puerto Serie (Rocío de Aguirre)
>   - Registros del GPIO
>   - Estructura de datos que "matchean" el GPIO
>   - Estratos de las primitivas


# 4. Puerto Serie 

## Registros del GPIO


## Estructura de datos que "matchean" el GPIO

Se introduce a continuación una versión simplificada de la función main() dentro del archivo **uart.c**.

```c
int main(void){
   boardConfig(); 
   uartConfig( UART_USB, 115200 );
   
   uint8_t dato  = 0;
   uint8_t dato1 = 1;
   uint8_t dato2 = 78;
   int32_t dato3 = 1234;

   static char uartBuff[10]; 

   uartWriteByte( UART_USB, 'h' - 32 ); 
   uartWriteByte( UART_USB, '\n' ); 

   uartWriteString( UART_USB, "\r\n" ); 
   uartWriteString( UART_USB, "Chau\r\n" );

   char miTexto[] = "Hola de nuevo\r\n";
   uartWriteString( UART_USB, miTexto );

   itoa( dato3, uartBuff, 10 );
   uartWriteString( UART_USB, uartBuff );

   while(1) {
      if(  uartReadByte( UART_USB, &dato ) ){
         uartWriteByte( UART_USB, dato );
      }
   }
   return 0 ;
}
```

## Estrato de las primitivas
Dentro del main() se pueden observar múltiples funciones a las que se hace referencia, estas se enlistan a continuación. 

1. boardConfig()
2. uartWriteByte()
3. uartWriteString()
4. itoa()
5. uartReadByte()
6. uartConfig()

### 1. boardConfig()
Se define la función boardConfig() a partir de la función boardInit() explicitada en firmware_v3/libs/sapi/sapi_v.6.2/board/inc/sapi_board.h
Incluimos a continuación un extracto de la función. Se extraen los comentarios y las partes de código que no refieren a la EDU-CIAA-NXP.

```c
#define boardConfig boardInit

void boardInit(void)
{
  SystemCoreClockUpdate();
  cyclesCounterInit( SystemCoreClock );
  tickInit( 1 );  
  gpioInit( 0, GPIO_ENABLE );

  gpioInit( TEC1, GPIO_INPUT );
...
  gpioInit( TEC4, GPIO_INPUT );

  gpioInit( LEDR, GPIO_OUTPUT );
...
  gpioInit( LED3, GPIO_OUTPUT );
}

```

Esta función se encarga de inicializar el GPIO. Comienza por setear los clocks y actualizar la variable SystemCoreClock. Luego, inicializa el conteo de Ticks con resolucion de 1ms. Por último, configure los pines del GPIO:
- Inicializa GPIOs
- Configura de pines de entrada para Teclas de la EDU-CIAA-NXP
- Configura de pines de salida para Leds de la EDU-CIAA-NXP


### 2. uartWriteByte()

```c
void uartWriteByte( uartMap_t uart, const uint8_t value ){
   while( uartTxReady( uart ) == FALSE );
   uartTxWrite( uart, value );
}
```
Esta función comienza por esperar un spacio en FIFO y luego envía un byte.
### 3. uartWriteString()

```c
void uartWriteString( uartMap_t uart, const char* str ){
   while( *str != 0 ) {
      uartWriteByte( uart, (uint8_t)*str );
      str++;
   }
}
```
Esta función envía un string.

### 4. itoa()

```c
char* itoa(int value, char* result, int base) {
   if (base < 2 || base > 36) { *result = '\0'; return result; }
   char* ptr = result, *ptr1 = result, tmp_char;
   int tmp_value;
   do {
      tmp_value = value;
      value /= base;
      *ptr++ = "zyxwvutsrqponmlkjihgfedcba9876543210123456789abcdefghijklmnopqrstuvwxyz" [35 + (tmp_value - value * base)];
   } while ( value );
   if (tmp_value < 0) *ptr++ = '-';
   *ptr-- = '\0';
   while(ptr1 < ptr) {
      tmp_char = *ptr;
      *ptr--= *ptr1;
      *ptr1++ = tmp_char;
   }
   return result;
}
```
Esta función comienza por validar la base ingresada y luego aplica un señal negativa. 
   
### 5. uartReadByte()

      

### 6. uartConfig()
Se define la función uartConfig() a partir de la función uartInit() explicitada en
(firmware_v3/libs/sapi/sapi_v0.6.2/soc/peropherals/inc/sapi_uart.h)

```c
#define uartConfig uartInit 

void uartInit( uartMap_t uart, uint32_t baudRate )
{
   Chip_UART_Init( lpcUarts[uart].uartAddr );
   Chip_UART_SetBaud( lpcUarts[uart].uartAddr, baudRate );
   Chip_UART_SetupFIFOS( lpcUarts[uart].uartAddr,
                         UART_FCR_FIFO_EN |
                         UART_FCR_TX_RS   |
                         UART_FCR_RX_RS   |
                         UART_FCR_TRG_LEV0 );
   Chip_UART_ReadByte( lpcUarts[uart].uartAddr );
   Chip_UART_TXEnable( lpcUarts[uart].uartAddr );
   Chip_SCU_PinMux( lpcUarts[uart].txPin.lpcScuPort,
                    lpcUarts[uart].txPin.lpcScuPin,
                    MD_PDN,
                    lpcUarts[uart].txPin.lpcScuFunc );
   Chip_SCU_PinMux( lpcUarts[uart].rxPin.lpcScuPort,
                    lpcUarts[uart].rxPin.lpcScuPin,
                    MD_PLN | MD_EZI | MD_ZI,
                    lpcUarts[uart].rxPin.lpcScuFunc );
   if( uart == UART_485 ) {
      Chip_UART_SetRS485Flags( LPC_USART0,
                               UART_RS485CTRL_DCTRL_EN |
                               UART_RS485CTRL_OINV_1     );
      Chip_SCU_PinMux( lpcUart485DirPin.lpcScuPort,
                       lpcUart485DirPin.lpcScuPin,
                       MD_PDN,
                       lpcUart485DirPin.lpcScuFunc );
   }
}
```
Esta función permite inicializar la UART. Comienza por setear el Baud rate, resetea FIFOS mediante FCR, FIFO Control Register. Luego, permite habilitar Enable y el contenido Reset, setear trigger level. A su vez, habilita la transmisión UART, configura el pin SCU UARTn_TXD y UARTn_RXD. Por último, configura RS485 especificando las Flags de RS485 y define un pin extra UARTn_DIR para RS485.

Esta incluye seis funciones y tres estructuras.    

1. Chip_UART_SetBaud
2. Chip_UART_SetupFIFOS 
3. Chip_UART_ReadByte
4. Chip_UART_TXEnable 
5. Chip_SCU_PinMux
6. Chip_UART_SetRS485Flags 

a. uartMap_t: typedef uartMap_t print_t;
b. lpcUart485DirPin
c. lpcUarts

#### Funciones: 

firmware_v3/libs/board/lpc_chip-43x/inc/uart_18xx_43xx.h: 

```c
uint32_t Chip_UART_SetBaud(LPC_USART_T *pUART, uint32_t baudrate){
	uint32_t div, divh, divl, clkin;	clkin = Chip_Clock_GetRate(UART_BClock[Chip_UART_GetIndex(pUART)]);
	div = clkin / (baudrate * 16);

	divh = div / 256;
	divl = div - (divh * 256);

	Chip_UART_EnableDivisorAccess(pUART);
	Chip_UART_SetDivisorLatches(pUART, divl, divh);
	Chip_UART_DisableDivisorAccess(pUART);

	return (clkin / div) >> 4;
}
```



```c
STATIC INLINE void Chip_UART_TXEnable(LPC_USART_T *pUART){
    pUART->TER2 = UART_TER2_TXEN;
}
```


```c
STATIC INLINE uint8_t Chip_UART_ReadByte(LPC_USART_T *pUART){
	return (uint8_t) (pUART->RBR & UART_RBR_MASKBIT);
}

```

```c
STATIC INLINE void Chip_UART_SetupFIFOS(LPC_USART_T *pUART, uint32_t fcr){
	pUART->FCR = fcr;
}

```

```c
STATIC INLINE void Chip_SCU_PinMuxSet(uint8_t port, uint8_t pin, uint16_t modefunc){
	LPC_SCU->SFSP[port][pin] = modefunc;
}
```

#### Estructuras

##### 1. Chip_UART_SetBaud

La estructura Chip_UART_SetBaud se encuentra definida en firmware_v3/libs/sapi/sapi_v0.6.2/board/inc/sapi_peripherical_map.h

```c
typedef enum {
   UART_GPIO = 0, // Hardware UART0 via GPIO1(TX), GPIO2(RX) pins on header P0
   UART_485  = 1, // Hardware UART0 via RS_485 A, B and GND Borns
                  // Hardware UART1 not routed
   UART_USB  = 3, // Hardware UART2 via USB DEBUG port
   UART_ENET = 4, // Hardware UART2 via ENET_RXD0(TX), ENET_CRS_DV(RX) pins on header P0
   UART_232  = 5, // Hardware UART3 via 232_RX and 232_tx pins on header P1
   UART_MAXNUM,
} uartMap_t;
```

##### 2. lpcUart485DirPin

La variable lpcUart485DirPin se encuentra definida en firmware_v3/libs/sapi/sapi_v0.6.2/soc/periphericals/src/sapi_uart.c. Se puede observar que esta se define a partir de la estructura lpc4337ScuPin_t especificada en firmware_v3/libs/sapi/sapi_v0.6.2/board/inc/sapi_peripherical_map.h.

```c
static const lpc4337ScuPin_t lpcUart485DirPin = {
   6, 2, FUNC2
};
```

```c
typedef struct {
   uint8_t lpcScuPort;
   uint8_t lpcScuPin;
   uint8_t lpcScuFunc;
} lpc4337ScuPin_t;
```


##### 3. lpcUarts

lpcUarts se trata de una constante estática del tipo uartLpcInit_t, que trata de una estructura que incorpora como campos ciertas estructuras. Por un lado, encontramos un puntero a LPC_USART_T que es una estructura de configuración de la UART. Por otro, vemos los periféricos tx y rx definidos a partir de los pines referenciados por una estructura lpc4337ScuPin_t, especificada antrteriormente. Por último, se define IRQn_Type como LPC18XX_IRQn_Type, una estructura que especifica la funcionalidad de todos los periféricos de micro. 

- lpcUarts: firmware_v3/libs/sapi/sapi_v0.6.2/soc/periphericals/src/sapi_uart.c
- uartLpcInit_t: firmware_v3/libs/sapi/sapi_v0.6.2/soc/periphericals/src/sapi_uart.c
- LPC_USART_T : firmware_v3/libs/board/lpc_chip-43x/inc/uart_18xx_43xx.h
- IRQn_Type: firmware_v3/libs/lpc_open/lpc_chip_43xx/inc/cmsis_18xx.h
- LPC18XX_IRQn_Type: firmware_v3/libs/lpc_open/lpc_chip_43xx/inc/cmsis_18xx.h

```c
static const uartLpcInit_t lpcUarts[] = {
	// { uartAddr, { txPort, txpin, txfunc }, { rxPort, rxpin, rxfunc }, uartIrqAddr  },
   // UART_GPIO (GPIO1 = U0_TXD, GPIO2 = U0_RXD)
   { LPC_USART0, { 6, 4, FUNC2 }, { 6, 5, FUNC2 }, USART0_IRQn }, // 0
   // UART_485 (RS485/Profibus)
   { LPC_USART0, { 9, 5, FUNC7 }, { 9, 6, FUNC7 }, USART0_IRQn }, // 1
   // UART not routed
   {  LPC_UART1, { 0, 0, 0     }, { 0, 0, 0     }, UART1_IRQn  }, // 2
   // UART_USB
   { LPC_USART2, { 7, 1, FUNC6 }, { 7, 2, FUNC6 }, USART2_IRQn }, // 3
   // UART_ENET
   { LPC_USART2, { 1,15, FUNC1 }, { 1,16, FUNC1 }, USART2_IRQn }, // 4
   // UART_232
   { LPC_USART3, { 2, 3, FUNC2 }, { 2, 4, FUNC2 }, USART3_IRQn }  // 5
};
```

```c
typedef struct {
   LPC_USART_T*      uartAddr;
   lpc4337ScuPin_t   txPin;
   lpc4337ScuPin_t   rxPin;
   IRQn_Type         uartIrqAddr;
} uartLpcInit_t;
```

```c
typedef struct {	/*!< USARTn Structure       */
	union {
		__IO uint32_t  DLL;			/*!< Divisor Latch LSB. Least significant byte of the baud rate divisor value. The full divisor is used to generate a baud rate from the fractional rate divider (DLAB = 1). */
		__O  uint32_t  THR;			/*!< Transmit Holding Register. The next character to be transmitted is written here (DLAB = 0). */
		__I  uint32_t  RBR;			/*!< Receiver Buffer Register. Contains the next received character to be read (DLAB = 0). */
	};

	union {
		__IO uint32_t IER;			/*!< Interrupt Enable Register. Contains individual interrupt enable bits for the 7 potential UART interrupts (DLAB = 0). */
		__IO uint32_t DLM;			/*!< Divisor Latch MSB. Most significant byte of the baud rate divisor value. The full divisor is used to generate a baud rate from the fractional rate divider (DLAB = 1). */
	};

	union {
		__O  uint32_t FCR;			/*!< FIFO Control Register. Controls UART FIFO usage and modes. */
		__I  uint32_t IIR;			/*!< Interrupt ID Register. Identifies which interrupt(s) are pending. */
	};

	__IO uint32_t LCR;				/*!< Line Control Register. Contains controls for frame formatting and break generation. */
	__IO uint32_t MCR;				/*!< Modem Control Register. Only present on USART ports with full modem support. */
	__I  uint32_t LSR;				/*!< Line Status Register. Contains flags for transmit and receive status, including line errors. */
	__I  uint32_t MSR;				/*!< Modem Status Register. Only present on USART ports with full modem support. */
	__IO uint32_t SCR;				/*!< Scratch Pad Register. Eight-bit temporary storage for software. */
	__IO uint32_t ACR;				/*!< Auto-baud Control Register. Contains controls for the auto-baud feature. */
	__IO uint32_t ICR;				/*!< IrDA control register (not all UARTS) */
	__IO uint32_t FDR;				/*!< Fractional Divider Register. Generates a clock input for the baud rate divider. */
	__IO uint32_t OSR;				/*!< Oversampling Register. Controls the degree of oversampling during each bit time. Only on some UARTS. */
	__IO uint32_t TER1;				/*!< Transmit Enable Register. Turns off USART transmitter for use with software flow control. */
	uint32_t  RESERVED0[3];
    __IO uint32_t HDEN;				/*!< Half-duplex enable Register- only on some UARTs */
	__I  uint32_t RESERVED1[1];
	__IO uint32_t SCICTRL;			/*!< Smart card interface control register- only on some UARTs */

	__IO uint32_t RS485CTRL;		/*!< RS-485/EIA-485 Control. Contains controls to configure various aspects of RS-485/EIA-485 modes. */
	__IO uint32_t RS485ADRMATCH;	/*!< RS-485/EIA-485 address match. Contains the address match value for RS-485/EIA-485 mode. */
	__IO uint32_t RS485DLY;			/*!< RS-485/EIA-485 direction control delay. */

	union {
		__IO uint32_t SYNCCTRL;		/*!< Synchronous mode control register. Only on USARTs. */
		__I  uint32_t FIFOLVL;		/*!< FIFO Level register. Provides the current fill levels of the transmit and receive FIFOs. */
	};

	__IO uint32_t TER2;				/*!< Transmit Enable Register. Only on LPC177X_8X UART4 and LPC18XX/43XX USART0/2/3. */
} LPC_USART_T;
```
```c
typedef LPC18XX_IRQn_Type IRQn_Type;
```

```c
typedef enum {
	/* -------------------------  Cortex-M3 Processor Exceptions Numbers  ----------------------------- */
	Reset_IRQn                        = -15,/*!<   1  Reset Vector, invoked on Power up and warm reset */
	NonMaskableInt_IRQn               = -14,/*!<   2  Non maskable Interrupt, cannot be stopped or preempted */
	HardFault_IRQn                    = -13,/*!<   3  Hard Fault, all classes of Fault */
	MemoryManagement_IRQn             = -12,/*!<   4  Memory Management, MPU mismatch, including Access Violation and No Match */
	BusFault_IRQn                     = -11,/*!<   5  Bus Fault, Pre-Fetch-, Memory Access Fault, other address/memory related Fault */
	UsageFault_IRQn                   = -10,/*!<   6  Usage Fault, i.e. Undef Instruction, Illegal State Transition */
	SVCall_IRQn                       = -5,	/*!<  11  System Service Call via SVC instruction */
	DebugMonitor_IRQn                 = -4,	/*!<  12  Debug Monitor                    */
	PendSV_IRQn                       = -2,	/*!<  14  Pendable request for system service */
	SysTick_IRQn                      = -1,	/*!<  15  System Tick Timer                */

	/* ---------------------------  LPC18xx/43xx Specific Interrupt Numbers  ------------------------------- */
	DAC_IRQn                          =   0,/*!<   0  DAC                              */
	RESERVED0_IRQn                    =   1,
	DMA_IRQn                          =   2,/*!<   2  DMA                              */
	RESERVED1_IRQn                    =   3,/*!<   3  EZH/EDM                          */
	RESERVED2_IRQn                    =   4,
	ETHERNET_IRQn                     =   5,/*!<   5  ETHERNET                         */
	SDIO_IRQn                         =   6,/*!<   6  SDIO                             */
	LCD_IRQn                          =   7,/*!<   7  LCD                              */
	USB0_IRQn                         =   8,/*!<   8  USB0                             */
	USB1_IRQn                         =   9,/*!<   9  USB1                             */
	SCT_IRQn                          =  10,/*!<  10  SCT                              */
	RITIMER_IRQn                      =  11,/*!<  11  RITIMER                          */
	TIMER0_IRQn                       =  12,/*!<  12  TIMER0                           */
	TIMER1_IRQn                       =  13,/*!<  13  TIMER1                           */
	TIMER2_IRQn                       =  14,/*!<  14  TIMER2                           */
	TIMER3_IRQn                       =  15,/*!<  15  TIMER3                           */
	MCPWM_IRQn                        =  16,/*!<  16  MCPWM                            */
	ADC0_IRQn                         =  17,/*!<  17  ADC0                             */
	I2C0_IRQn                         =  18,/*!<  18  I2C0                             */
	I2C1_IRQn                         =  19,/*!<  19  I2C1                             */
	RESERVED3_IRQn                    =  20,
	ADC1_IRQn                         =  21,/*!<  21  ADC1                             */
	SSP0_IRQn                         =  22,/*!<  22  SSP0                             */
	SSP1_IRQn                         =  23,/*!<  23  SSP1                             */
	USART0_IRQn                       =  24,/*!<  24  USART0                           */
	UART1_IRQn                        =  25,/*!<  25  UART1                            */
	USART2_IRQn                       =  26,/*!<  26  USART2                           */
	USART3_IRQn                       =  27,/*!<  27  USART3                           */
	I2S0_IRQn                         =  28,/*!<  28  I2S0                             */
	I2S1_IRQn                         =  29,/*!<  29  I2S1                             */
	RESERVED4_IRQn                    =  30,
	RESERVED5_IRQn                    =  31,
	PIN_INT0_IRQn                     =  32,/*!<  32  PIN_INT0                         */
	PIN_INT1_IRQn                     =  33,/*!<  33  PIN_INT1                         */
	PIN_INT2_IRQn                     =  34,/*!<  34  PIN_INT2                         */
	PIN_INT3_IRQn                     =  35,/*!<  35  PIN_INT3                         */
	PIN_INT4_IRQn                     =  36,/*!<  36  PIN_INT4                         */
	PIN_INT5_IRQn                     =  37,/*!<  37  PIN_INT5                         */
	PIN_INT6_IRQn                     =  38,/*!<  38  PIN_INT6                         */
	PIN_INT7_IRQn                     =  39,/*!<  39  PIN_INT7                         */
	GINT0_IRQn                        =  40,/*!<  40  GINT0                            */
	GINT1_IRQn                        =  41,/*!<  41  GINT1                            */
	EVENTROUTER_IRQn                  =  42,/*!<  42  EVENTROUTER                      */
	C_CAN1_IRQn                       =  43,/*!<  43  C_CAN1                           */
	RESERVED6_IRQn                    =  44,
	RESERVED7_IRQn                    =  45,/*!<                                       */
	ATIMER_IRQn                       =  46,/*!<  46  ATIMER                           */
	RTC_IRQn                          =  47,/*!<  47  RTC                              */
	RESERVED8_IRQn                    =  48,
	WWDT_IRQn                         =  49,/*!<  49  WWDT                             */
	RESERVED9_IRQn                    =  50,
	C_CAN0_IRQn                       =  51,/*!<  51  C_CAN0                           */
	QEI_IRQn                          =  52,/*!<  52  QEI                              */
} LPC18XX_IRQn_Type;
```

 
