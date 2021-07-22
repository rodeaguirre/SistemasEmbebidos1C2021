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
# *Ejercicio 5*

## **Enunciado**

Este ejercicio propone utilizar el ADC para leer un valor analógico y mostrar luego este valor en la PC a través de la UART USB. 

## **Programas Utilizados**

Para resolver este ejercicio se utilizó el ejemplo de adc_dac.c, donde cada 500 ms se lee un valor a través del ADC se lo muestra en la PC y se lo escribe en el DAC.

## **Cambios Realizados**

Lo que se realizó en este caso fue utilizar el mismo código eliminado la parte del DAC y eliminando una sección de código que hacía titilar un led. 

## **Código**

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
## **Funciones Importantes**
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


# *Ejercicio 6*

## **Objetivo:**

Crear un código que permita escribir el DAC en función de lo recibido por la UART y luego leer el ADC y trasmitir el valor leído nuevamente por la UART. 

## **Funcionamiento**

Se leerá continuamente la UART2 y con los caractéres de 0 a 9 se seleccionará que nivel de tensión queremos proporcionarle al DAC. Considerando que el máximo es 3.3V, separamos en niveles que distan de 0.37V cada uno siendo 0 el nivel  correspondiente a 0V y 9 el nivel lo más cercano a 3.3V. 
Luego, el CH1 del ADC estará loopeado al DAC a través de una resistencia de 10KOhm y el micro deberá enviar por la UART2 el valor leido por el ADC. 


## **Camino Lógico:**
TECLA (0..9) -> TERMINAL (PC) -> UART (USB).RX -> DAC -> (0..3,3v) -> RESISTOR 10KOhm -> ( 0v .. 3,3V) -> RESISTOR -> ADC CH1 -> UART (USB).TX -> TERMINAL (PC) -> TEXTO (PANTALLA)

## **Diagrama de Conexiones**
![Diagrama de Conexiones](https://lh3.googleusercontent.com/IjVXNO8mUvWyz5Ecn7OKZSR-6RctfDgU4SvTy-mwyFq1Y9OF5w-V7sIj7doa39tPEvCdTjTcDMPwyGWd4XzCs_Ui0XdoN6v01adJdclOjmiF3tQed7v7SbUNKO2x29mbr_jG21eYwFYibaihIwiFyexj-9lKWEH3snSgHvvcRj-NebIDcOI_qh123zakCjRmBDHYmIuTdvRGYz0ixvJl5x2JPDEmGH1-bp4HBsMvKOVF9sON229NQLK1vW-pBNux7eq5nqtw9pVxLnkAaOg8C_ZCmeMjVhnpD3EWuA1tMKpzY5nnXpFtKwPTDYW8Un89PxDi5uQ7ZxJ9Jm_M7_FJuKtX1TK3E_8mekbHvOR3kTkIy0RADdM_mIWVUiH92ynk83kbck5VJaLbX5ofaLRRH-hkqgpsOdyzm1knMDh1GUltOz_4KmNpRJKLmHb95COXH5aZV7gIXKSlEF54ljdJeWstNXR4KIMZWPwcGbGEHU_QR53RuFzVwZk7PYxyt8c4NJpmdqMhuq3LR4zfa_-HZp0eXpOHZq3QA4T9uD8_E-TnhBtinoMq_dQ6pW0uDHpUPeJ8IXpErnCGf3oYWMToTlIpiflBiw-zVNB9Spg84KkDydsx05tAiqwYwdKux6xaZ91F6hjutV-KHURVw24otqPLuHX4xulyN-kNNdGVSVMWNdgLXwbZ-d4wDGGa0eIXVmDLJr1iPNOHYmTg2OcfTAU=w596-h470-no?authuser=1)


## **Implementación**

Para implementar este programa se partió del ejemplo incluido en el firmware_v3  en */firmware_v3/examples/c/sapi/adc_dac*. 

De aquí se mantuvo el delay1, que se encarga de ejecutar un bloque cada un determinado tiempo (500ms) pero sin bloquear el programa y también la utilización de la función itoa() para imprimir los valores leídos del ADC por la UART en ASCII. 

Luego, fue necesario agregar un bloque de lectura de la UART2. En este caso, se implementó por polling y el valor leído tendrá efecto solamente si está dentro del rango de los número 0-9 (equivalente a 48-57 en ASCII) . Para estos casos, se seteará el DAC con el valor correspondiente al nivel recibido por la UART. 

Además, el programa imprime por la UART cada 500ms el valor leído del CH1 del ADC e informa, en caso de recibir un cambio de nivel válido, a qué nivel se configurará el DAC. 

  
A continuación, se muestra la implementación del programa:

```c

/* Copyright 2016, Eric Pernia.
 * All rights reserved.
 * This file is part sAPI library for microcontrollers.

 * Date: 22/07/2021
 * Modified by: Martin Anus
 */

/*==================[inclusions]=============================================*/

#include "sapi.h"        // <= sAPI header

/*==================[macros and definitions]=================================*/

#define DAC_STEP 113  // paso del DAC para 9 niveles: 1024 (max 16bits / 9 niveles)
#define ZERO_ASCII 48 // valor decimal del 0 en ASCII
#define NINE_ASCII 57 // valor decimal del 9 en ASCII

/*==================[internal data declaration]==============================*/

/*==================[internal functions declaration]=========================*/

/*==================[internal data definition]===============================*/

/*==================[external data definition]===============================*/

/*==================[internal functions definition]==========================*/

/*==================[external functions definition]==========================*/


/**
 * C++ version 0.4 char* style "itoa":
 * Written by Lukás Chmela
 * Released under GPLv3.

 */
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
   dacConfig( DAC_ENABLE ); /* DAC */



   /* Buffer */
   static char uartBuff[10];

   /* Variable para almacenar el valor leido del ADC CH1 */
   uint16_t muestra = 0;
   uint16_t dacValue = 0x0000;

   uint8_t data;



   /* Variables de delays no bloqueantes */
   delay_t delay1;


   /* Inicializar Retardo no bloqueante con tiempo en ms */
   delayConfig( &delay1, 500 );




   /* ------------- REPETIR POR SIEMPRE ------------- */
   while(1) {
	   if(uartReadByte(UART_USB, &data))
		   if(data >= ZERO_ASCII && data <= NINE_ASCII){
			   uartWriteString( UART_USB, "Se cambiara el DAC al nivel: " );
		       uartWriteByte( UART_USB, data);
		       uartWriteString( UART_USB, "\r\n" );

			   dacValue = (data - ZERO_ASCII) * DAC_STEP;
			   dacWrite( DAC, dacValue);
		   }


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
   return 0 ;
}

/*==================[end of file]============================================*/


```	











## Ejercicio 7

#### Enunciado del ejercicio

En el conector CONN_20x2 de la edu-ciaa, armar un loopback a través de puerto serie Uart3, a través de un resistor de valor igual o superior a 1k.:

**Pin 88 {P2_4} bornera 23 [RX]* --/\/\/\-- *[TX] 25 bornera {P2₃} Pin 87**

a. Modificar el programa del ejercicio 2 para redireccionar el envió del carácter de la UART2 a la UART3.
b. Modificar el programa del ejercicio 3 para redireccionar la recepción del carácter de la UART2 a la UART3.
c. Testear que cuando esté conectado el resistor de loopback, se puedan prender o apagar 2 de los leds con los 4 pulsadores.

**PULSADOR -> LED -> UART3.TX -> RESISTOR -> UART3.RX -> LED**

#### Diseño funcional

Se utiliza una resistencia de 5k6 ohms entre el pin 88 y 87. 

Cada tecla de la placa cambia el estado (encendido - apagado) del LED que tiene en frente. A su vez, se envía el mismo caracter por el UART2 y UART3. El UART3 recibe el caracter, y cambia el estado del led que se encuentra a la derecha del anterior. 

#### Programa original

Para poder afrotar el presente ejercicio, me basé en el ejercicio 3, este aporta la estructura general del nuevo código.

Se incorporó la habilitación del transmisor UART 3 tomada del archivo tx_rx_interrupt_bridge.c ubicada en el directorio firmware_v3/examples/c/sapi/uart/tx_rx_interrupt_bridge/src. 

```c
   uartCallbackSet(UART_USB, UART_TRANSMITER_FREE, uartUsbSendCallback, NULL);
```
De este mismo archivo, utilicé el criterio de inicialización de terminales propuesto, de manera tal de que ambas sean reconocidas al realizar el debugging.

```c
   uartWriteByte(UART_USB, 'u');
   uartWriteByte(UART_232, '2');
```

#### Código final

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
# *Ejercicio 8*

## **Enunciado**

Se pide modificar el ejercicio 3 para fowardear lo que se recibe a través de la UART 2 por la UART 3 y viceversa.

## **Programas Utilizados**

Se utilizaron los programas rx_interrupt.c, blinky-switch.c y echo.c

## **Cambios Realizados**
Se modifico el programa rx_interrupt para habilitar la interrupción de la UART3 (UART_232). Se modifico la función que llaman en el interrupt para que seteen un flag. En el main y quito la parte de hacer titilar un led y se agregó, en base a blinky-switch, un bloque que lee el estado de los pulsadores cada determinado tiempo y en base a eso prender/apagar leds y mandar un carácter por la UART2. A continuación se verifica si los flags de rx de la UART 2/3 está seateado, y si lo está prender/apagar un led y fowardea el caracter a través de la UART 3/2.

## **Código**
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
ledXState_t c;
/*==================[external data definition]===============================*/

/*==================[internal functions definition]==========================*/

/*==================[external functions definition]==========================*/
void redireccionarUart( void *noUsado )
{
	rxUART2 = true;
	c =(ledXState_t) uartRxRead( UART_USB );
}

void switchLeds( void *noUsado )
{
	rxUART3 = true;
	c =(ledXState_t) uartRxRead( UART_232 );
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
		   uartWriteByte( UART_USB,(char) c);
	   }
   }

}
```
## **Funciones Importantes**

Las funciones utilizadas en este ejercicio se utilizaron en los anteriores.


