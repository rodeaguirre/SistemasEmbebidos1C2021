# SistemasEmbebidos1C2021
# Trabajo Práctico N°3


> ### Indice
> 1. Ejercicio 2
> 2. Ejercicio 3
> 3. Ejercicio 4
> 4. Ejercicio 5
> 5. Ejercicio 6
> 6. Ejercicio 7
> 7. Ejercicio 8

----------------------------------------------------------

## Ejercicio 2

Programar una aplicación que realice lo siguiente:
cuando se presione (o se suelte) uno de los pulsadores se deberá prender (o apagar) un led de la placa y enviar un carácter a
través de la terminal serie implementada con el puerto USB de debug. Dado que hay 4 pulsadores en la placa, asignar un led y un caracter
distinto a cada pulsador y mantener la consistencia entre ellos.

Para la realización se utilizó el programa /*switches_leds*/ como base de partida, cuya ubicación es:
firmware_v3/examples/c/sapi/gpio/switches_leds/src/**swtiches_leds.c**

Se modificó dentro de cada ciclo condicional /*if*/ para enviae un caracter por la /*uart*/ en el caso de que la condición sea verdadera.

A continuación se muestra la implementación del programa.

```c
int main(void){

   /* ------------- INICIALIZACIONES ------------- */

   /* Inicializar la placa */
   boardConfig();

   // Inicializar UART_USB a 115200 baudios
   uartConfig( UART_USB, 115200 );

   /* Variable para almacenar el valor de tecla leido */
   bool_t valor;

   /* ------------- REPETIR POR SIEMPRE ------------- */
   while(1) {
	  //tecla 1
      valor = !gpioRead( TEC1 );
      if(valor == 1){
    	  uartWriteByte( UART_USB, 'H' );  // Envia 'H'
      }
      gpioWrite( LEDB, valor );

	  //tecla 2
      valor = !gpioRead( TEC2 );
      if(valor == 1){
    	  uartWriteByte( UART_USB, 'o' );  // Envia 'o'
      }
      gpioWrite( LED1, valor );

	  //tecla 3
      valor = !gpioRead( TEC3 );
      if(valor == 1){
    	  uartWriteByte( UART_USB, 'l' );  // Envia 'l'
      }
      gpioWrite( LED2, valor );

	  //tecla 4
      valor = !gpioRead( TEC4 );
      if(valor == 1){
    	  uartWriteByte( UART_USB, 'a' );  // Envia 'a'
      }
      gpioWrite( LED3, valor );

      delay(200);
   }

   /* NO DEBE LLEGAR NUNCA AQUI, debido a que a este programa no es llamado
      por ningun S.O. */
	return 0 ;
}
```



## Ejercicio 3

Programar una aplicación que realice lo siguiente:

por medio de la terminal serie implementada con el puerto USB de debug, y basándose en el ejemplo /*uart.c*/, asignar 3 caracteres para prender cada uno de los 3 led y 3 caracteres para apagar cada uno de los 3 led. Implementarlo por interrupciones, basandose en el ejemplo /*rx_interrupt*/. Por último, complementarlo con el /*ejercicio 2*/.

Para la realización se utilizó el programa /*rx_interrupt*/ como base de partida, cuya ubicación es:
firmware_v3/examples/c/sapi/uart/rx?interrupt/src/**rx_interrupt.c**

Se incorporó al /*main*/ el código del /*ejercicio 2*/. Por otro lado, al producirse la interrupción se llama a la función /*onRx()*/ donde se lee y guarda el caracter enviado y se setea un flag. Este flag permite ingresar dentro del /*ciclo if*/ donde se discierne qué caracter se ingresó y que led debe prenderse o apagarse.
```c
#include "sapi.h"

static char c;
static bool_t int_flag = FALSE;

void onRx( void *noUsado )
{
   c = uartRxRead( UART_USB );
   int_flag = TRUE;
}

int main(void)
{
   /* Inicializar la placa */
   boardConfig();

   /* Inicializar la UART_USB junto con las interrupciones de Tx y Rx */
   uartConfig(UART_USB, 115200);
   // Seteo un callback al evento de recepcion y habilito su interrupcion
   uartCallbackSet(UART_USB, UART_RECEIVE, onRx, NULL);
   // Habilito todas las interrupciones de UART_USB
   uartInterrupt(UART_USB, true);

   /* Variable para almacenar el valor de tecla leido */
   bool_t valor;

   while(1) {

		  //tecla 1
	      valor = !gpioRead( TEC1 ); // Envia 'H'
	      if (valor == 1) uartWriteByte( UART_USB, 'H' );

		  //tecla 2
	      valor = !gpioRead( TEC2 );  // Envia 'o'
	      if (valor == 1) uartWriteByte( UART_USB, 'o' );

		  //tecla 3
	      valor = !gpioRead( TEC3 ); // Envia 'l'
	      if (valor == 1) uartWriteByte( UART_USB, 'l' );

		  //tecla 4
	      valor = !gpioRead( TEC4 ); // Envia 'a'
	      if (valor == 1) uartWriteByte( UART_USB, 'a' );


	      if(int_flag == TRUE )
	      {
	      	  printf( "Recibimos <<%c>> por UART\r\n", c );
	    	  if (c == 'a') gpioToggle( LED1);
	    	  else if (c == 'b') gpioToggle( LED2);
	    	  else if (c == 'c') gpioToggle( LED3);
	    	  int_flag = FALSE;
	      }

	      delay(200);

   }

   /* NO DEBE LLEGAR NUNCA AQUI, debido a que a este programa no es llamado
      por ningun S.O. */
   return 0 ;
}

```

A continuación se presentan las funciones relacionadas con la generación de interrupciones.

##### uartCallbackSet()

* **Definición:** Se habilita la interrupción por evento de la /*uart*/ y se setea la llamada a función.
* **Argumentos**
	* **uart:** uart correspondiente.
	* **event:** evento de interrupción (UART_RECEIVE o UART_TRANSMITER_FREE)
	* **callbackFunc:** función a ejecutar al generarse la interrupción.
	* **callbackFunc:** Null.
```c
// UART Interrupt event Enable and set a callback
void uartCallbackSet( uartMap_t uart, uartEvents_t event, 
                      callBackFuncPtr_t callbackFunc, void* callbackParam )
{   
   uint32_t intMask;

   switch(event){

      case UART_RECEIVE:
         // Enable UART Receiver Buffer Register Interrupt
         //intMask = UART_IER_RBRINT;
         
         // Enable UART Receiver Buffer Register Interrupt and Enable UART line
         //status interrupt. LPC43xx User manual page 1118
         intMask = UART_IER_RBRINT | UART_IER_RLSINT;
         
         if( callbackFunc != 0 ) {
            // Set callback
            if( (uart == UART_GPIO) || (uart == UART_485) ){
               rxIsrCallbackUART0 = callbackFunc;
               rxIsrCallbackUART0Params = callbackParam;
            }
            if( (uart == UART_USB) || (uart == UART_ENET) ){
               rxIsrCallbackUART2 = callbackFunc;
               rxIsrCallbackUART2Params = callbackParam;
            }            
            if( uart == UART_232 ){
               rxIsrCallbackUART3 = callbackFunc;
               rxIsrCallbackUART3Params = callbackParam;
            }
         } else{
            return;
         }
      break;
      case UART_TRANSMITER_FREE:
      ...

```

##### uartInterrupt()

* **Definición:** Se habilita las interrupciones globales de la /*uart*/.
* **Argumentos**
	* **uart:** uart correspondiente.
	* **enable:** habilitado o deshabilitado (TRUE o FALSE).

```c
// UART Global Interrupt Enable/Disable
void uartInterrupt( uartMap_t uart, bool_t enable )
{
   if( enable ) {
      // Interrupt Priority for UART channel
      NVIC_SetPriority( lpcUarts[uart].uartIrqAddr, 5 ); // FreeRTOS Requiere prioridad >= 5 (numero mas alto, mas baja prioridad)
      // Enable Interrupt for UART channel
      NVIC_EnableIRQ( lpcUarts[uart].uartIrqAddr );
   } else {
      // Disable Interrupt for UART channel
      NVIC_DisableIRQ( lpcUarts[uart].uartIrqAddr );
   }
}
```