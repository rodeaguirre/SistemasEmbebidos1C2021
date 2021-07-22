* Trabajo Práctico N'4
EJERCICIO 5
5.1 ENUNCIADO
Este ejercicio propone utilizar el ADC para leer un valor analógico y mostrar luego este valor en la PC a través de la UART USB. 
5.2 PROGRAMAS UTILIZADOS
Para resolver este ejercicio se utilizó el ejemplo de adc_dac.c, donde cada 500 ms se lee un valor a través del ADC se lo muestra en la PC y se lo escribe en el DAC.
5.3 CAMBIOS REALIZADOS
Lo que se realizó en este caso fue utilizar el mismo código eliminado la parte del DAC y eliminando una sección de código que hacía titilar un led. 
5.4 CÓDIGO
El código utilizado es el siguiente:
```c
/*==================[inclusions]=============================================*/

#include "sapi.h"        // <= sAPI header

/*==================[macros and definitions]=================================*/

/*==================[internal data declaration]==============================*/

/*==================[internal functions declaration]=========================*/

/*==================[internal data definition]===============================*/

/*==================[external data definition]===============================*/

/*==================[internal functions definition]==========================*/

/*==================[external functions definition]==========================*/

char* itoa(int value, char* result, int base) {
   // check that the base if valid
   if (base < 2 || base > 36) { *result = '\0'; return result; }

   char* ptr = result, *ptr1 = result, tmp_char;
   int tmp_value;

   do {
      tmp_value = value;
      value /= base;
      *ptr++ = "zyxwvutsrqponmlkjihgfedcba9876543210123456789abcdefghijklmnopqrstuvwxyz" [35 + (tmp_value - value * base)];
   } while ( value );

   // Apply negative sign
   if (tmp_value < 0) *ptr++ = '-';
   *ptr-- = '\0';
   while(ptr1 < ptr) {
      tmp_char = *ptr;
      *ptr--= *ptr1;
      *ptr1++ = tmp_char;
   }
   return result;
}


/* FUNCION PRINCIPAL, PUNTO DE ENTRADA AL PROGRAMA LUEGO DE RESET. */
int main(void){

   /* ------------- INICIALIZACIONES ------------- */

   /* Inicializar la placa */
   boardConfig();

   /* Inicializar UART_USB a 115200 baudios */
   uartConfig( UART_USB, 115200 );

   /* Inicializar AnalogIO */
   /* Posibles configuraciones:
    *    ADC_ENABLE,  ADC_DISABLE,
    *    ADC_ENABLE,  ADC_DISABLE,
    */
   adcConfig( ADC_ENABLE ); /* ADC */

   /* Configuración de estado inicial del Led */
   bool_t ledState1 = OFF;

   /* Contador */
   uint32_t i = 0;

   /* Buffer */
   char uartBuff[20];

   /* Variable para almacenar el valor leido del ADC CH1 */
   uint16_t muestra = 0;

   /* Variables de delays no bloqueantes */
   delay_t delay1;

   /* Inicializar Retardo no bloqueante con tiempo en ms */
   delayConfig( &delay1, 500 );

   /* ------------- REPETIR POR SIEMPRE ------------- */
   while(1) {

      /* delayRead retorna TRUE cuando se cumple el tiempo de retardo */
      if ( delayRead( &delay1 ) ){

         /* Leo la Entrada Analogica AI0 - ADC0 CH1 */
         muestra = adcRead( CH1 );

         /* Envío la primer parte del mnesaje a la Uart */
         uartWriteString( UART_USB, "ADC CH1 value: " );

         /* Conversión de muestra entera a ascii con base decimal */
         itoa( muestra, uartBuff, 10 ); /* 10 significa decimal */

         /* Enviar muestra y Enter */
         uartWriteString( UART_USB, uartBuff );
         uartWriteString( UART_USB, ";\r\n" );

      }

   }

   /* NO DEBE LLEGAR NUNCA AQUI, debido a que a este programa no es llamado
      por ningun S.O. */
   return 0;
}
```
5.5 FUNCIONES IMPORTANTES
```c
void adcInit(adcInit_t config)
```
Configura el ADC para con una frecuencia de sampleo de 200 kHz, una resolución de 10 bits, puede habilitarlo o deshabilitarlo en función del parámetro config.
```c
uint16_t adcRead( adcMap_t analogInput )
```
que lee genera una lectura en el ADC. Es una función bloqueante.
```c
char* itoa(int value, char* result, int base)
```
Convierte un número entero en un string.


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
EJERCICIO 8
8.1 ENUNCIADO
Se pide modificar el ejercicio 3 para fowardear lo que se recibe a través de la UART 2 por la UART 3 y viceversa.
8.2 PROGRAMAS UTILIZADOS
Se utilizaron los programas rx_interrupt.c, blinky-switch.c y echo.c
8.3 CAMBIOS REALIZADOS
Se modifico el programa rx_interrupt para habilitar la interrupción de la UART3 (UART_232). Se modifico la función que se llaman en la interrupción para que seteen un flag, rxUART2 o rxUART3 dependiendo del caso. En el main y quito la parte de hacer titilar un led y se agregó, en base a blinky-switch, un bloque que lee el estado de los pulsadores cada determinado tiempo y en base a eso prender/apagar leds y mandar un carácter por la UART2. A continuación se verifica si los flags de rx de la UART 2/3 está seateado, y si lo está prender/apagar un led y fowardea el caracter a través de la UART 3/2.

8.4 CÓDIGO
```c
/*==================[inclusions]=============================================*/

#include "sapi.h"     // <= sAPI header

/*==================[macros and definitions]=================================*/

/*==================[internal data declaration]==============================*/

/*==================[internal functions declaration]=========================*/

/*==================[internal data definition]===============================*/
const uint8_t cantBotones = 3;

typedef enum{
	LED_1_APAGADO = 'A',
	LED_2_APAGADO,
	LED_3_APAGADO,
	LED_1_PRENDIDO = 'a',
	LED_2_PRENDIDO,
	LED_3_PRENDIDO,
} ledXState_t;

const ledXState_t caracteresLed[][2] = {
		{LED_1_APAGADO,LED_1_PRENDIDO},
		{LED_2_APAGADO,LED_2_PRENDIDO},
		{LED_3_APAGADO,LED_3_PRENDIDO}
};

bool rxUART2 = false;
bool rxUART3 = false;
/*==================[external data definition]===============================*/

/*==================[internal functions definition]==========================*/

/*==================[external functions definition]==========================*/
void redireccionarUart( void *noUsado )
{
	bool rxUART2 = true;
}

void switchLeds( void *noUsado )
{
	bool rxUART3 = true;
}


// FUNCION PRINCIPAL, PUNTO DE ENTRADA AL PROGRAMA LUEGO DE RESET.
int main(void){

   // ------------- INICIALIZACIONES -------------

   // Inicializar la placa
   boardConfig();

   // Inicializar UART_USB a 115200 baudios
   uartConfig( UART_USB, 115200 );

   // Inicializar UART_232 a 115200 baudios
   uartConfig( UART_232, 115200 );

   uartCallbackSet(UART_USB, UART_RECEIVE, redireccionarUart, NULL);
   // Habilito todas las interrupciones de UART_USB
   uartInterrupt(UART_USB, true);

   uartCallbackSet(UART_232, UART_RECEIVE, switchLeds, NULL);
   // Habilito todas las interrupciones de UART_USB
   uartInterrupt(UART_232, true);

   delay_t led1Delay[cantBotones];
   bool valorAnterior[cantBotones];

   for(uint8_t i = 0; i < cantBotones; i++)
   {
	   delayConfig( &(led1Delay[i]), 100 );
	   valorAnterior[i] = 0;
   }

   bool valor;
   ledXState_t c;

   // ------------- REPETIR POR SIEMPRE -------------
   while(1)
   {
	   for(uint8_t i = 0; i < cantBotones ;i++)
	   {
		   if( delayRead(&(led1Delay[i])))
		   {
			   valor = !gpioRead(TEC2+i);
			   if(valor != valorAnterior[i])
			   {
				   gpioWrite(LED1+i,valor);
				   gpioWrite(LEDR+i,valor);
				   uartWriteByte( UART_USB,(char) caracteresLed[i][valor]);
				   valorAnterior[i] = valor;
			   }
		   }
	   }
	   if(rxUART2)
	   {
		   rxUART2 = false;
		   c =(ledXState_t) uartRxRead( UART_USB );
			   switch(c)
			   {
			   case LED_1_APAGADO:
				   gpioWrite(LEDR,OFF);
				   break;
			   case LED_1_PRENDIDO:
				   gpioWrite(LEDR,ON);
				   break;
			   case LED_2_APAGADO:
				   gpioWrite(LEDG,OFF);
				   break;
			   case LED_2_PRENDIDO:
				   gpioWrite(LEDG,ON);
				   break;
			   case LED_3_APAGADO:
				   gpioWrite(LEDB,OFF);
				   break;
			   case LED_3_PRENDIDO:
				   gpioWrite(LEDB,ON);
				   break;
			   default:
				   break;
			   }

			uartWriteByte( UART_232,(char) c);
	   }
	   if(rxUART3)
	   {
		   rxUART3 = false;
		   c =(ledXState_t) uartRxRead( UART_232 );
		   switch(c)
		   {
		   case LED_1_APAGADO:
			   gpioWrite(LED1,OFF);
			   break;
		   case LED_1_PRENDIDO:
			   gpioWrite(LED1,ON);
			   break;
		   case LED_2_APAGADO:
			   gpioWrite(LED2,OFF);
			   break;
		   case LED_2_PRENDIDO:
			   gpioWrite(LED2,ON);
			   break;
		   case LED_3_APAGADO:
			   gpioWrite(LED3,OFF);
			   break;
		   case LED_3_PRENDIDO:
			   gpioWrite(LED3,ON);
			   break;
		   default:
			   break;
		   }
		   uartWriteByte( UART_232,(char) c);
	   }
   }

}
```
8.5 FUNCIONES IMPORTANTES
Las funciones utilizadas en este ejercicio se utilizaron en los anteriores.


