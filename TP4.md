* Trabajo Práctico N'4


** Ejercicio 7

**** Enunciado del ejercicio

En el conector CONN_20x2 de la edu-ciaa, armar un loopback a través de puerto serie Uart3, a través de un resistor de valor igual o superior a 1k.:

**Pin 88 {P2_4} bornera 23 [RX]* --/\/\/\-- *[TX] 25 bornera {P2₃} Pin 87**

a. Modificar el programa del ejercicio 2 para redireccionar el envió del carácter de la UART2 a la UART3.
b. Modificar el programa del ejercicio 3 para redireccionar la recepción del carácter de la UART2 a la UART3.
c. Testear que cuando esté conectado el resistor de loopback, se puedan prender o apagar 2 de los leds con los 4 pulsadores.

**PULSADOR -> LED -> UART3.TX -> RESISTOR -> UART3.RX -> LED**

**** Programa original

Para poder afrotar el presente ejercicio, me basé en el ejercicio 3, este aporta la estructura general del nuevo código.

**** Modificaciones realizadas

Incorporé la habilitación del transmisor UART 3 tomada del archivo tx_rx_interrupt_bridge.c ubicada en el directorio firmware_v3/examples/c/sapi/uart/tx_rx_interrupt_bridge/src. 

```c
   uartCallbackSet(UART_USB, UART_TRANSMITER_FREE, uartUsbSendCallback, NULL);
```
De este mismo archivo, utilicé el criterio de inicialización de terminales propuesto, de manera tal de que ambas sean reconocidas al realizar el debugging.

```c
   uartWriteByte(UART_USB, 'u');
   uartWriteByte(UART_232, '2');
```

**** Código final

```c
#include "sapi.h"

static char c;
static char lectura;
static bool_t int_flag = FALSE;


void onRx_USB( void *noUsado ){
   c = uartRxRead(UART_USB);
   printf( "Recibimos <<%c>> por UART\r\n", c );
}

void onRx_232( void *unused ){
   lectura = uartRxRead(UART_232);
   int_flag = TRUE;
}

int main(void)
{
   /* Inicializar la placa */
   boardConfig();

   /* Inicializar la UART_232 */
   uartConfig(UART_232, 115200);
   uartCallbackSet(UART_232, UART_RECEIVE, onRx_232, NULL);
   uartInterrupt(UART_232, true);

   /* Inicializar la UART_USB */
   uartConfig(UART_USB, 115200);
   uartCallbackSet(UART_USB, UART_RECEIVE, onRx_USB, NULL);
   uartInterrupt(UART_USB, true);


   // Identificación de terminal
   uartWriteByte(UART_USB, 'u');
   uartWriteByte(UART_232, '2');

   bool_t valor;

   while(1) {
  
		  //tecla 1
	      valor = !gpioRead( TEC1 ); // Envia 'H'
	      if (valor == 1){
	    	  uartWriteByte(UART_USB,'H');
	    	  uartWriteByte(UART_232,'H');
	    	  gpioToggle(LEDB);
	      }

		  //tecla 2
	      valor = !gpioRead( TEC2 );  // Envia 'o'
	      if (valor == 1){
	    	  uartWriteByte( UART_USB,'o');
	    	  uartWriteByte( UART_232,'o');
	    	  gpioToggle(LED1);
	      }


		  //tecla 3
	      valor = !gpioRead( TEC3 ); // Envia 'l'
	      if (valor == 1){
	    	  uartWriteByte( UART_USB, 'l' );
	    	  uartWriteByte( UART_232, 'l' );
	    	  gpioToggle(LED2);
	      }


		  //tecla 4
	      valor = !gpioRead( TEC4 ); // Envia 'a'
	      if (valor == 1){
	    	  uartWriteByte(UART_USB, 'a' );
	    	  uartWriteByte(UART_232, 'a' );
	    	  gpioToggle(LED3);
	      }

	      if(int_flag == TRUE )
	      {
	    	  if (lectura == 'H') gpioToggle( LED1);
	    	  else if (lectura == 'o') gpioToggle( LED2);
	    	  else if (lectura == 'l') gpioToggle( LED3);
	    	  else if (lectura == 'a') gpioToggle( LEDB);
	    	  int_flag = FALSE;
	      }

	      delay(200);

   }
   return 0 ;
}
```

