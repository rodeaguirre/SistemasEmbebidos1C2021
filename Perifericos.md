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
Se pueden observar múltiples funciones a las que se hace referencia, las cuales se enlistarán a continuación. 

1. boardConfig()
2. uartConfig()
3. uartWriteByte()
4. uartWriteString()
5. itoa()
6. uartReadByte()

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



### 2. uartConfig()
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
// UART Initialization

   // Initialize UART

   // Set Baud rate
   
   //Chip_UART_ConfigData(LPC_UART, (UART_LCR_WLEN8 | UART_LCR_SBS_1BIT));
   
   // Restart FIFOS using FCR (FIFO Control Register).
   // Set Enable, Reset content, set trigger level
   
   // Dummy read
   
   // Enable UART Transmission
   
   // Configure SCU UARTn_TXD pin
                    
   // Configure SCU UARTn_RXD pin

   // Specific configurations for RS485
   
   IF 
    // Specific RS485 Flags
    // UARTn_DIR extra pin for RS485
      
- uartMap_t
- lpcUart485DirPin
- lpcUarts

- Chip_UART_SetBaud
- Chip_UART_SetupFIFOS - 
- Chip_UART_ReadByte -
- Chip_UART_TXEnable - 
- Chip_SCU_PinMux
- Chip_UART_SetRS485Flags - 

STATIC INLINE void Chip_UART_TXEnable(LPC_USART_T *pUART){
    pUART->TER2 = UART_TER2_TXEN;
}

STATIC INLINE uint8_t Chip_UART_ReadByte(LPC_USART_T *pUART){
	return (uint8_t) (pUART->RBR & UART_RBR_MASKBIT);
}

STATIC INLINE void Chip_UART_SetupFIFOS(LPC_USART_T *pUART, uint32_t fcr)
{
	pUART->FCR = fcr;
}


### 3. uartWriteByte()

```c
void uartWriteByte( uartMap_t uart, const uint8_t value )
{
   // Wait for space in FIFO (blocking)
   while( uartTxReady( uart ) == FALSE );
   // Send byte
   uartTxWrite( uart, value );
}
```

### 4. uartWriteString()

```c
void uartWriteString( uartMap_t uart, const char* str ){
   while( *str != 0 ) {
      uartWriteByte( uart, (uint8_t)*str );
      str++;
   }
}
```
// Blocking Send a string

### 5. itoa()

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

// check that the base if valid
// Apply negative sign
   
### 6. uartReadByte()

```c
void uartWriteByteArray( uartMap_t uart, const uint8_t* byteArray, uint32_t byteArrayLen ){
   uint32_t i = 0;
   for( i=0; i<byteArrayLen; i++ ) {
      uartWriteByte( uart, byteArray[i] );
   }
}
```
// Blocking, Send a Byte Array
      
## Estratos de las primitivas


 
