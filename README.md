# SistemasEmbebidos1C2021
# Trabajo Práctico N°2


> ### Indice
> 1. SCU
>   - Introducción
>   - Estructura
> 2. GPIO
>   - 1.REGISTROS DEL MICRO CONTROLADOR
>   - 2.Estructura de datos que &quot;matchean&quot; el GPIO
>   - 3.Primitivas
>	- 4.Diagrama
> 3. Timer 
>   - General
>   - Características
>   - Registros de los Timers
>	- Clock
> 4. Puerto Serie 
>   - Introducción
>   - Funciones
>   - Primitivas

----------------------------------------------------------

# SCU (System Configuration Unit)

## Introducción:

Los registros SCU se utilizan para configurar los distintos modos de funcionamiento que tienen los pins del micro.

Los puertos digitales se agrupan en 16 puertos entre P0-P9 y PA-PF
NXP LPC4337 JBD144 - EDU CIAA - tiene P0-P7 y P9

(Atención, los nombres de los pines no corresponden a los asignados para GPIO)



SCU determina la funcion y modo electrido de los pines digitales. Por defecto está configurada la función 0 para todos los pines y se enecuentra habilitado el pull-up.

  

Para los pines que soportan funciones analógicas y digitales, se pueden habilitar la función anaógica en los registros de selección de función de ADC en la SCU. También, el registro de delay del clock para los pines SDRAM EMC_CLK y los registros que seleccionan los pines de interrupción se encuentran en la SCU.

  

El set de entradas y salidas analógicas para ADC y DAC como tambien la mayoria de los pines USB se encuentran separados y NO son controlables a traves del SCU.


## Estructura:

### Nivel Chip – en scu-18xx:43xx.h:

```c
*** Copyright(C) NXP Semiconductors, 2012**

/**
* @brief Array of pin definitions passed to Chip_SCU_SetPinMuxing() must be in this format

*/

**typedef**  **struct** {

uint8_t  pingrp; /* Pin group */

uint8_t  pinnum; /* Pin number */

uint16_t  modefunc; /* Pin mode and function for SCU */

} PINMUX_GRP_T;


/**

* @brief System Control Unit register block

*/

**typedef**  **struct** {

__IO  uint32_t  SFSP[16][32];

__I  uint32_t  RESERVED0[256];

__IO  uint32_t  SFSCLK[4]; /*!< Pin configuration register for pins CLK0-3 */

__I  uint32_t  RESERVED16[28];

__IO  uint32_t  SFSUSB; /*!< Pin configuration register for USB */

__IO  uint32_t  SFSI2C0; /*!< Pin configuration register for I2C0-bus pins */

__IO  uint32_t  ENAIO[3]; /*!< Analog function select registerS */

__I  uint32_t  RESERVED17[27];

__IO  uint32_t  EMCDELAYCLK; /*!< EMC clock delay register */

__I  uint32_t  RESERVED18[63];

__IO  uint32_t  PINTSEL[2]; /*!< Pin interrupt select register for pin int 0 to 3 index 0, 4 to 7 index 1. */

} LPC_SCU_T;
```

### **Nivel Sapi - en sapi_peripherical_map.h:**

```c
* This file is part sAPI library for microcontrollers.

  

/*==================[typedef]================================================*/

  

/* ----- Begin Pin Init  Structs NXP LPC4337 ----- */

  

**typedef**  **struct** {

int8_t  port;

int8_t  pin;

} pinInitLpc4337_t;

  

**typedef**  **struct** {

uint8_t  lpcScuPort;

uint8_t  lpcScuPin;

uint8_t  lpcScuFunc;

} lpc4337ScuPin_t;

  

/* ------ End Pin Init  Structs NXP LPC4337 ------ */

```

###  Detalle de Registros:

Capitulo 17 de [Manual del Usuario - UM10503 LPC43xx/LPC43Sxx ARM Cortex-M4/M0 multi-core microcontroller](https://github.com/ciaa/firmware_v3/blob/master/documentation/CIAA_Boards/NXP_LPC4337/NXP_Info/UM10503.pdf) 



![SCU Register Overview](https://lh3.googleusercontent.com/i_m3Z6C3ju9-Qb6k6rN6iC29X7JZI29plFbjzH5HReoJ5ZIqKp-DuAsmJGplhCPx3jm8BZyBAZ31bW_ZJXZ_DF0zO-J_sVPhF0SidSeCwxR5ahVjpmAhgCtopsS2I6pNMzXq4XupRK92yKDFUrJ-W-wBIXComR1PWVztD0BIsbYmAdDWZb9NN_8m3BNrqGvMhxOJ_1cRqD6cbwqVuPwktuq4rTyn8T7-T-DNKvZVgRSzPeACP1C0S2tdbuKbynGiavcCFiB1NNMFCb7Nx6YZ0pExOmldOOVpzNvzmZGAMlLHuxktFgub4IczSH3O2kZsF0hufv8pr_KFxsNq2TQa_j0MvR3SvV81vY8uPbB8UlpbJJjqiyS1HiRwkop_SdX4qqf6jSP_jajoe9fFQOtJjV0HKNe6pu1DCLjPBp_pYZ_DScqKwzWVQfhmLg2EDF9YNlD5MMXR6dSivRih8KVfAvnMl1NBC5ss1OYOKoMUxhlLwwIOJ9tbDJF8g8PLEJkbdTQXlmZTtUXpzHwBTnHI_UjJIUgl-Ag2xHfvIHK_FqSUA50a0qNQesScxIYCG6Gv1WKDAv7zeZzTD9O_jlv0rEq3pnLc8BvAz-i90H3Gojy6RadbdjKG8RtNjY-_4QibS_l3OM8mQB6L-YZGDNQ4mjHH2nT5Hgb7eEa-a5A4OqB5hQVvdqO7JXIqkGD1uJmWQPTu5n1BxS8ZiYOsQccyzio=w696-h663-no?authuser=1)*Table 191. Register overview: System Control Unit (SCU) (base address 0x4008 6000) - Page 412 – UM10503 LPC43xx/LPC43Sxx ARM Cortex-M4/M0 multi-core microcontroller*


Rangos Relativos para cada sección:

Puertos P0-P9 y PA-PF
0x000 - 0x498 

PA-PF (estos no estan en NXP LPC4337 JBD144)
0x500 - 0x7AC

CLKn pins
0xC00 - 0xC0C

USB1 USB1_DP/USB1_DM pins and I 2 C-bus open-drain pins
0xC80 - 0xC84

ADC pin select registers
0xC88 - 0xC90

EMC delay register
0xD00

SD/MMC delay register
0xD80

Pin interrupt select registers
0xE00 - 0xE04

### Esquemáticos:

Esquemático que muestra las conexiones de las salidas del micro en:

- [EDU-CIAA-NXP Esquematico](https://github.com/ciaa/firmware_v3/blob/master/documentation/CIAA_Boards/NXP_LPC4337/EDU-CIAA-NXP/EDU-CIAA-NXP%20Esquematico.pdf)

![EDU-CIAA-NXP Esquematico](https://lh3.googleusercontent.com/I35WI3JxxuEe61PN3ycPkBv3FTc_Ei8Bz7Bm9JiKEQQChTFVujV8VVJCoT5bCnRf0y3gi9OjKZl3en0G1ykXQZ4QD1zCoNraB80K6axkjE8fOkwzkq0hB8JBgVXVHr2oJwnQ6AiNzytcLCXMqGYLIkvH_kcIvJf_4ruUbxtVELHH2VXOcQ8Hd6Ul7DK3H-nHvdvkULwUOkW4ngaiKHN3YyDggytf64pwoxOhDtKFe8GeQlt-6GQUbe5ecs73JM6Wjk3SZ7iVzRoDYctN2ZtDuNzN7i_hDd9gSxAIfkTVbR8ECa33sW4nMreErjr0K_WGUE2AtSM7vdlhi98MUu_ibrySDGhfa_vOnD0jYRg7IgxULHmhr2cfOC2UlEM7Og7IFOCxEa96ULXBG4_yG3-CeejdeZ6s5uLeDkS5DyoP4GqqWVnDLrJ9CnusPpsbP51uWgVXW79HC4Aft2Ayo9od0jsg-oMRIS6c-u-enF93Vp0xh6f78458_OTcQNZmKD-BzqlsLGn7i4TYIb0VuBMP321i3EY-LC2NRZbnnRMupubU7Xk71K105srAxEwxyoNHndolDrly9Vr295ZNyI26ECT4OZ4Xq-_byBYWFTB3uqW2qsIhQCgUJXE0Tfh_VFcZNKCBn9p9NZwLnUyk3WkLmI8bgr61KMobWt803qo0IGB3IG6rTg4gimjJjXAnqKNmUWs1oyvwFuMqdeeHjmn37Ag=w312-h548-no?authuser=1)


Esquemáticos que asocian funciones y posiciones en placa EDU-CIAA:

- [EDU-CIAA-NXP v1.1 Board - 2019-01-03 v5r0](https://github.com/ciaa/firmware_v3/blob/master/documentation/CIAA_Boards/NXP_LPC4337/EDU-CIAA-NXP/EDU-CIAA-NXP%20v1.1%20Board%20-%202019-01-03%20v5r0.pdf)


![Leyenda Pines](https://lh3.googleusercontent.com/u2TSFnFjtstcvRyafX0JDFkWeQTPHoMIcL4Wa0IhZG39h72ZaXrN_iq-8BiKrHOetqO4mjfBYCV2a3cp1UEyY-75H9xZcwlIs94llDOVLCv-sqfa5-DXPNUZKU_GqGE_S2ro6L_7TeLofqXCbuaf29R2uyV-7ELbOSypsSOaHK9rkYVQ7SyMTtp9IZteC8f3RnZMczqI6dW755RXJCo7w2Fx4rtP9nvqYhIgnUqnGAtU1Q3hY4UddGpOgGu-8iCJRwd2jXJk01byJGrjCOy7p8mYcg4oYRelRWadP37zuTE2JfqSeq9bwHgTIF8U6gD7nYdNXzZy6IvhbXfkSFuXRvEMEJ-JzUvSPDehJKoLJNZUb39SX2-QETVrbcztMHqYhUI_s47i_v1pkjgVdJQHEQkgpFFgMbSHKg2LqZBxzOlj-sNnA_6bt9M106uWIhDJ3k6ZcU6oO1z-QtTV2dSASlHRzFFKQHgGVgA4opSdke0_Fg32n3OY1Tx3J9AXE81-P1E6jYvkv3mHzjm0p_znYTlaDg3aJsng1CEWlhhbIP42OKWnRlSRAoYJY34BRb0SSTH79RQqluHjNWJj-TtmA7PJTRy6Ee6cnE0p9Iwb4YqrLjSnLPf3E2rDocg2lp4O1RmwgClREGMy-qpVbgu4Q3KFGpt5YDQF-fvEzwz_GA6AtllghsMrGETTtJyXNLxWEezwLKDnCapkk8Z1dEUbbwQ=w1170-h229-no?authuser=1)


![EDU-CIAA Pines utilizados del NXP LPC4337 (1 a 72)](https://lh3.googleusercontent.com/a6Zz4RAmktCd1wtjX0jOazxAHPTLou9IZkPK_38JOH8Y3bELrGY5qifKF37omZT7ucIdFwxpTuQ0_XqYHaWRnIgP2yEk4R0ABLJBcH6paZmY6ycLjhEaI7Y6-O68ENygez_dBsa4Z9_m38bc2prU1eEUmRuDb0oOjkj0iCArT2Gc2msjd5pUoAvXEo9wuykzkVamsEAfqlNEd4ZtiA0xoDQtMYfkusQWficygFhAfTzOEIfgSFLdV4d6b7OyKpzoQuiUpVNMxMXxg39EWSmq6yXeDJREEXtM6hC-wF05-igysMRqqFm2g37H-aXJzqpCZjxRewUk0T19EqB9RByNcuiYPN0GJAabgD3CEVE4ifKnKP64TnyhLI6wPpdlMLCRoHmgPeSfCJ4jyfm-ghUJPKf0NEVcEaqx-cF2pdQKTpiCavlpkRNjFLOg_NW8iwFil8j2RrJOz15SQRLIIEekglQsJ1-Y0vDpgt7wW9LNJGI8Rp9NFfQd91nZy-H31crHQrtUgAVFUMLfyUJtA6YetLmo6kBrpPpmNfkerqUe-a7gv2TF8syfZWUz1-JU5HGtqhfhisx8b9A_rRCq0swgOxtymRSqhOh2T-t2DZezIByX7vM2y3BAZ8_EE3kLvX40-0jMmCGT0WK_aIxMGC6nFEgNkH72XSzNfGUeYB3-fZ87v9-v2ZnosuWUrp678UR6cSLdtpOjP6EGNPxXW3JXBBA=w1268-h663-no?authuser=1)

![EDU-CIAA Pines utilizados del NXP LPC4337 (73 a 144)](https://lh3.googleusercontent.com/J0s-5Zek_ds4nGjNOEuVPwEoVa3ZHtYkMn4zpifamyfuMMhI5M84eewN--qe260DFPZMudL_D-Igx37-4TCYqhPZjkHxeBvMl2itvFb8e7wfCYpIancV3yuDnDj-BAokT4kXtHTABy1puhnzeq2hMIStc_cG99U98Dp2ND9m7f-v2V8fe7ekF1D23vbUtwgNJCNQuKoH9CgbTWl4uT41jK4ntOXoLKvudYJcZGOjm8evwTOnJJ8fR3S26bfsXpmJ47XYk2yHPPk3pzxBJ0yEKk81hOFD6qSJT433AwhavYK31V6GYlEzXWHTSJoZ2anXJg3_26CB9P2O3gmemD8n852PzwL3alJy-JbHp_jRZ5eZD-XEPWI5zBmK9i7o2unozNii7H4MMVNFg0yv5fjpZHAif49h66wFlQO4S1vEgiyVE8kP673viMqrZmU15VfVxSjEcgbiIbudyp41mJyriGp-z0zDTw8CJ-uzxbCi6flS_Jm51AAm1f38zhmpwR7sKZG0rxpqeEdGqnKQigmkIWwl1lskjwpGcsW0SYuX5rFRRnHjURt6tlMkkUa3tZVHt9Ncqiyt_kSPJ7g7VYOthdQX3RzltK49FTy1Jzq_dqRGlZ1hFpj4AgN68_iBFV03Fo7vAfzt1ivOqw8psKsDnhbwQZvo7EsQix2c8g1w0uwAyAtlRCi3etSI8XEYQOwhNBtXGBo4evoHMa8Bx3cjaYg=w1237-h663-no?authuser=1)


### Primitivasy Macros:

En **scu-18xx:43xx.h** se definen todas las Macros de SCU y las primitivas.

**Macros**:

```c
* Copyright(C) NXP Semiconductors, 2012

/**

* SCU function and mode selection definitions

* See the User Manual for specific modes and functions supoprted by the

* various LPC18xx/43xx devices. Functionality can vary per device.

*/

**#define** SCU_MODE_PULLUP (0x0 << 3) /*!< Enable pull-up resistor at pad */

**#define** SCU_MODE_REPEATER (0x1 << 3) /*!< Enable pull-down and pull-up resistor at resistor at pad (repeater mode) */

**#define** SCU_MODE_INACT (0x2 << 3) /*!< Disable pull-down and pull-up resistor at resistor at pad */

**#define** SCU_MODE_PULLDOWN (0x3 << 3) /*!< Enable pull-down resistor at pad */

**#define** SCU_MODE_HIGHSPEEDSLEW_EN (0x1 << 5) /*!< Enable high-speed slew */

**#define** SCU_MODE_INBUFF_EN (0x1 << 6) /*!< Enable Input buffer */

**#define** SCU_MODE_ZIF_DIS (0x1 << 7) /*!< Disable input glitch filter */

**#define** SCU_MODE_4MA_DRIVESTR (0x0 << 8) /*!< Normal drive: 4mA drive strength */

**#define** SCU_MODE_8MA_DRIVESTR (0x1 << 8) /*!< Medium drive: 8mA drive strength */

**#define** SCU_MODE_14MA_DRIVESTR (0x2 << 8) /*!< High drive: 14mA drive strength */

**#define** SCU_MODE_20MA_DRIVESTR (0x3 << 8) /*!< Ultra high- drive: 20mA drive strength */

**#define** SCU_MODE_FUNC0 0x0 /*!< Selects pin function 0 */

**#define** SCU_MODE_FUNC1 0x1 /*!< Selects pin function 1 */

**#define** SCU_MODE_FUNC2 0x2 /*!< Selects pin function 2 */

**#define** SCU_MODE_FUNC3 0x3 /*!< Selects pin function 3 */

**#define** SCU_MODE_FUNC4 0x4 /*!< Selects pin function 4 */

**#define** SCU_MODE_FUNC5 0x5 /*!< Selects pin function 5 */

**#define** SCU_MODE_FUNC6 0x6 /*!< Selects pin function 6 */

**#define** SCU_MODE_FUNC7 0x7 /*!< Selects pin function 7 */

**#define** SCU_PINIO_FAST (SCU_MODE_INACT | SCU_MODE_HIGHSPEEDSLEW_EN | SCU_MODE_INBUFF_EN | SCU_MODE_ZIF_DIS)

  

**#define** PORT_OFFSET 0x80 /** Port offset definition */

**#define** PIN_OFFSET 0x04 /** Pin offset definition */
```

En **chip_lpc43xx.h:**

```c
**#define** LPC_SCU_BASE 0x40086000
```

**Primitivas**:

En **scu-18xx:43xx.h**

```c
**void**  **Chip_SCU_PinMuxSet**(uint8_t port, uint8_t pin, uint16_t modefunc) 
```
--> para setear el modo de funcionamiento de un determinado pin 

```c
**void**  **Chip_SCU_ClockPinMuxSet**(uint8_t clknum, uint16_t modefunc)
```
--> configura los las funciones de los pines de clock.  

```c
**void**  **Chip_SCU_GPIOIntPinSel**(uint8_t PortSel, uint8_t PortNum, uint8_t PinNum)
```
→ setea los pines GPIO que haran interrupciones

```c
**void**  **Chip_SCU_I2C0PinConfig**(uint32_t I2C0Mode)
```
-->configura el pin de I2C

```c
**void**  **Chip_SCU_ADC_Channel_Config**(uint32_t ADC_ID, uint8_t channel)
```
--> configura el canal de ADC que se pase como parametro.

```c
**void**  **Chip_SCU_DAC_Analog_Config**(**void**)
```
-> habilita la funcion del DAC en el pin P4_4

```c
**void**  **Chip_SCU_SetPinMuxing**(**const**  PINMUX_GRP_T *pinArray, uint32_t arrayLength)
```
--> se setea el modo de funcionamiento de un conjunto de pines. llama a Chip_SCU_PinMuxSet() para cada elemento del array de entrada.

----------------------------------------------------------

# GPIO

## 1. REGISTROS DEL MICRO CONTROLADOR

### IO

Los registros que controlan la GPIO van desde la posición 0x400F 4000 a la hasta la 0x400F 6320. En este sector se encuentran 9 grupos: B, W, DIR, MASK, PIN, MPIN, SET, CLR, NOT.

En el caso de los grupos B y W, Modificar el registro BX modifica el pin Z del puerto Y donde X = Y\*32 + Z. Para todos los otros registros Modificar el bit Z del registro Y modifica el pin Z del puerto Y.

**B:** Son registros de 8 bits. Hay 256 de estos, cada uno controla un pin. De cada uno de estos registros solo se van a leer o escribir el bit menos significativo. Un 1 en ese bit corresponde a estado alto. El offset va desde 0x0000 hasta 0x00FF.

**W:** Son registros de 32 bits. Hay 256 de estos, cada uno controla un pin. En la lectura se lee 0x FFFF FFFF si esta en alto y 0 si está en bajo. En la escritura, se escribe 0 para que esté en estado bajo y cualquier otro valor para que está en alto. El offset va desde 0x1000 hasta 0x13FC.

**DIR:** Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Escribir un 1 en un bit pone a un pin como salida y si se escribe un 0 se establece como entrada. El offset va desde 0x2000 hasta 0x201C.

**MASK:** Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Poner un 1 en el bit X del registro MASKY hace que no se pueda leer ni escribir en el bit X del registro MPINY. El offset va desde 0x2080 hasta 0x209C.

**PIN:** Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Se puede leer o escribir el estado de un pin. Un 1 corresponde a estado alto. El offset va desde 0x2100 hasta 0x211C.

**MPIN:** Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Si El bit correspondiente de MASK está en 1, entonces el bit no se puede escribir y siempre se lee como 0. Si el bit de MASK está en 0 entonces funciona como PIN. El offset va desde 0x2180 hasta 0x219C.

**SET:** Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Escribir un 1 setea el pin y escribir un 0 no genera cambios. No se puede leer estos registros. El offset va desde 0x2200 hasta 0x221C.

**CLR:** Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Al leer se lee el estado del pin. Escribir un 1 pone el pin en bajo y escribir un 0 no genera cambios. El offset va desde 0x2280 hasta 0x229C.

**NOT:** Son registros de 32 bits. Hay 8 de estos, uno por cada puerto. Escribir un 1 togglea el estado del pin y escribir un 0 no genera cambios. No se puede leer estos registros. El offset va desde 0x2300 hasta 0x231C.

### INTERRUPCIONES POR PIN

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

### INTERRUPCIONES POR GRUPO

Las interrupciones por grupo se generan cuando una o todos los pines de un grupo, dependiendo de la configuración, envían la señal adecuada. Existen 2 interrupciones de este tipo: GINT0, GINT1. Ambas interrupciones presentan la misma estructura de registros. La diferencia radica en que la dirección base de GINT0 es 0x4008 8000 mientras que la de GINT1 es 0x4008 9000.

Tabla 1.3:

| Registro | offset | Función |
| --- | --- | --- |
| CTRL | 0x00 | Bit 0: leer un 0 indica que no hay interrupción pendiente mientras que leer un 1 indica que si la hay. Escribir un 0 no genera cambios y escribir un 1 pone en 0 el bitBit 1: 0 función OR, indica que uno de los pines del grupo tiene que cumplir la condición de trigger para que se genere la interrupción. 1 función AND, indica que todos los pines tienen que cumplir la condición.Bir 2: 0 la interrupción se genera por flanco, 1 la interrupción se genera por nivel |
| PORT\_POLn | 0x20 +n\*0x4 | Bit m: 0 el pin m del gpio port n se activa en LOW, 1 se activa en HIGH. |
| PORT\_ENAn | 0x40 +n\*0x4 | Bit m: 0 el pin m del gpio port n no contribuye a la interrupción, 1el pin contribuye a la interrupción. |

## 2. Estructura de datos que &quot;matchean&quot; el GPIO

### IO

La estructura que marche los registros de GPIO con su número de puerto y pin es ```LPC_GPIO_T```, que se muestra en la figura 2.1.

![Figura GPIO 2.1](/GPIO/LPC_GPIO_T.jpg)

Figura 2.1: Estructura que representa los registros GPIO.

En la figura 2.1 se puede observar que IO corresponde a los registros de lectura y escritura y O corresponde a los de solo escritura. También se puede observar que cada grupo se agregan elementos de más para que cada agrupo posea el offset correcto.

También existe una etiqueta ```LPC_GPIO_PORT_BASE``` a la que se le asigna el valor 0x400F4000, que corresponde a la dirección de inicio de los registros GPIO. Además, se encuentra la línea ```#define LPC_GPIO_PORT ((LPC_GPIO_T *) LPC_GPIO_PORT_BASE)``` que genera un puntero a ```LPC_GPIO_T```con el valor de la dirección de inicio.

Como es necesario saber el numero de puerto y pin para poder gestionar el GPIO, existen otras 3 estructuras que sirven para obtener el numero de pin y puerto a partir de la etiqueta que le asigna la EDU-CIAA a cada pin del microcontrolador.

La primera estructura es un ```enum gpioMap_t```, que se encuentra en el archivo sapi\_peripheral\_map.h. enum. Lo que hace es asignarle a cada etiqueta de la EDU-CIAA un valor de posición. La segunda estructura es un vector de ```pinInitGpioLpc4337_t```, llamado gpioPinsInit, que se encuentra en el archivo sapi\_gpio.c. Lo que hace es obtener un ```pinInitGpioLpc4337_t```, a partir de la posición asignada a cada etiquta. Por último, el ```pinInitGpioLpc4337_t``` contiene 5 ```int8_t```, correspondientes al puerto físico, pin físico, número de función asignada a gpio, puerto gpio y pin gpio. En conjunto estas 3 estructuras lo que hacen en conjunto es apartir de una estiqueta poder manejar la SCU para poner la función gpio y poder manejar la gpio. En las figuras 2.2 a 2.4 se puede observar la estructura ```de pinInitGpioLpc4337_t```.

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

```c
#define LPC_GPIO_PIN_INT ((LPC_PIN_INT_T*) LPC_PIN_INT_BASE)
``` 
Generan un puntero a la dirección inicial de dichos registros.

### INTERRUPCIONES POR GRUPO

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

## 3. Primitivas

### IO: Nivel Chip (NXP)

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

### IO: Nivel Sapi
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

### INTERRUPCIONES POR PIN: Nivel Chip (NXP)

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


### INTERRUPCIONES POR GRUPO: Nivel Chip (NXP)

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

## 4. Diagrama

![Figura GPIO 4.1](/GPIO/GPIO_resumen.jpg)

----------------------------------------------------------

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

----------------------------------------------------------

# Puerto Serie 

## introducción

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

## Funciones
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

 
