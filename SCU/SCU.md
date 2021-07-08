# **SCU (System Configuration Unit)**

## **Introducción:**

Los registros SCU se utilizan para configurar los distintos modos de funcionamiento que tienen los pins del micro.

Los puertos digitales se agrupan en 16 puertos entre P0-P9 y PA-PF
NXP LPC4337 JBD144 - EDU CIAA - tiene P0-P7 y P9

(Atención, los nombres de los pines no corresponden a los asignados para GPIO)

  

SCU determina la funcion y modo electrido de los pines digitales. Por defecto está configurada la función 0 para todos los pines y se enecuentra habilitado el pull-up.

  

Para los pines que soportan funciones analógicas y digitales, se pueden habilitar la función anaógica en los registros de selección de función de ADC en la SCU. También, el registro de delay del clock para los pines SDRAM EMC_CLK y los registros que seleccionan los pines de interrupción se encuentran en la SCU.

  

El set de entradas y salidas analógicas para ADC y DAC como tambien la mayoria de los pines USB se encuentran separados y NO son controlables a traves del SCU.


## **Estructura:**

### **Nivel Chip – en scu-18xx:43xx.h:**
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

###  **Detalle de Registros:**

Capitulo 17 de [Manual del Usuario - UM10503 LPC43xx/LPC43Sxx ARM Cortex-M4/M0 multi-core microcontroller](https://github.com/ciaa/firmware_v3/blob/master/documentation/CIAA_Boards/NXP_LPC4337/NXP_Info/UM10503.pdf) 



![SCU Register Overview](https://lh3.googleusercontent.com/i_m3Z6C3ju9-Qb6k6rN6iC29X7JZI29plFbjzH5HReoJ5ZIqKp-DuAsmJGplhCPx3jm8BZyBAZ31bW_ZJXZ_DF0zO-J_sVPhF0SidSeCwxR5ahVjpmAhgCtopsS2I6pNMzXq4XupRK92yKDFUrJ-W-wBIXComR1PWVztD0BIsbYmAdDWZb9NN_8m3BNrqGvMhxOJ_1cRqD6cbwqVuPwktuq4rTyn8T7-T-DNKvZVgRSzPeACP1C0S2tdbuKbynGiavcCFiB1NNMFCb7Nx6YZ0pExOmldOOVpzNvzmZGAMlLHuxktFgub4IczSH3O2kZsF0hufv8pr_KFxsNq2TQa_j0MvR3SvV81vY8uPbB8UlpbJJjqiyS1HiRwkop_SdX4qqf6jSP_jajoe9fFQOtJjV0HKNe6pu1DCLjPBp_pYZ_DScqKwzWVQfhmLg2EDF9YNlD5MMXR6dSivRih8KVfAvnMl1NBC5ss1OYOKoMUxhlLwwIOJ9tbDJF8g8PLEJkbdTQXlmZTtUXpzHwBTnHI_UjJIUgl-Ag2xHfvIHK_FqSUA50a0qNQesScxIYCG6Gv1WKDAv7zeZzTD9O_jlv0rEq3pnLc8BvAz-i90H3Gojy6RadbdjKG8RtNjY-_4QibS_l3OM8mQB6L-YZGDNQ4mjHH2nT5Hgb7eEa-a5A4OqB5hQVvdqO7JXIqkGD1uJmWQPTu5n1BxS8ZiYOsQccyzio=w696-h663-no?authuser=1)*Table 191. Register overview: System Control Unit (SCU) (base address 0x4008 6000) - Page 412 – UM10503 LPC43xx/LPC43Sxx ARM Cortex-M4/M0 multi-core microcontroller*

  

Rangos Relativos para cada sección:

  

Puertos P0-P9 y PA-PF

0x000 - 0x498 -

  

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


### **Esquemáticos:**

Esquemático que muestra las conexiones de las salidas del micro en:

- [EDU-CIAA-NXP Esquematico](https://github.com/ciaa/firmware_v3/blob/master/documentation/CIAA_Boards/NXP_LPC4337/EDU-CIAA-NXP/EDU-CIAA-NXP%20Esquematico.pdf)

![EDU-CIAA-NXP Esquematico](https://lh3.googleusercontent.com/I35WI3JxxuEe61PN3ycPkBv3FTc_Ei8Bz7Bm9JiKEQQChTFVujV8VVJCoT5bCnRf0y3gi9OjKZl3en0G1ykXQZ4QD1zCoNraB80K6axkjE8fOkwzkq0hB8JBgVXVHr2oJwnQ6AiNzytcLCXMqGYLIkvH_kcIvJf_4ruUbxtVELHH2VXOcQ8Hd6Ul7DK3H-nHvdvkULwUOkW4ngaiKHN3YyDggytf64pwoxOhDtKFe8GeQlt-6GQUbe5ecs73JM6Wjk3SZ7iVzRoDYctN2ZtDuNzN7i_hDd9gSxAIfkTVbR8ECa33sW4nMreErjr0K_WGUE2AtSM7vdlhi98MUu_ibrySDGhfa_vOnD0jYRg7IgxULHmhr2cfOC2UlEM7Og7IFOCxEa96ULXBG4_yG3-CeejdeZ6s5uLeDkS5DyoP4GqqWVnDLrJ9CnusPpsbP51uWgVXW79HC4Aft2Ayo9od0jsg-oMRIS6c-u-enF93Vp0xh6f78458_OTcQNZmKD-BzqlsLGn7i4TYIb0VuBMP321i3EY-LC2NRZbnnRMupubU7Xk71K105srAxEwxyoNHndolDrly9Vr295ZNyI26ECT4OZ4Xq-_byBYWFTB3uqW2qsIhQCgUJXE0Tfh_VFcZNKCBn9p9NZwLnUyk3WkLmI8bgr61KMobWt803qo0IGB3IG6rTg4gimjJjXAnqKNmUWs1oyvwFuMqdeeHjmn37Ag=w312-h548-no?authuser=1)



Esquemáticos que asocian funciones y posiciones en placa EDU-CIAA:

- [EDU-CIAA-NXP v1.1 Board - 2019-01-03 v5r0](https://github.com/ciaa/firmware_v3/blob/master/documentation/CIAA_Boards/NXP_LPC4337/EDU-CIAA-NXP/EDU-CIAA-NXP%20v1.1%20Board%20-%202019-01-03%20v5r0.pdf)



![Leyenda Pines](https://lh3.googleusercontent.com/u2TSFnFjtstcvRyafX0JDFkWeQTPHoMIcL4Wa0IhZG39h72ZaXrN_iq-8BiKrHOetqO4mjfBYCV2a3cp1UEyY-75H9xZcwlIs94llDOVLCv-sqfa5-DXPNUZKU_GqGE_S2ro6L_7TeLofqXCbuaf29R2uyV-7ELbOSypsSOaHK9rkYVQ7SyMTtp9IZteC8f3RnZMczqI6dW755RXJCo7w2Fx4rtP9nvqYhIgnUqnGAtU1Q3hY4UddGpOgGu-8iCJRwd2jXJk01byJGrjCOy7p8mYcg4oYRelRWadP37zuTE2JfqSeq9bwHgTIF8U6gD7nYdNXzZy6IvhbXfkSFuXRvEMEJ-JzUvSPDehJKoLJNZUb39SX2-QETVrbcztMHqYhUI_s47i_v1pkjgVdJQHEQkgpFFgMbSHKg2LqZBxzOlj-sNnA_6bt9M106uWIhDJ3k6ZcU6oO1z-QtTV2dSASlHRzFFKQHgGVgA4opSdke0_Fg32n3OY1Tx3J9AXE81-P1E6jYvkv3mHzjm0p_znYTlaDg3aJsng1CEWlhhbIP42OKWnRlSRAoYJY34BRb0SSTH79RQqluHjNWJj-TtmA7PJTRy6Ee6cnE0p9Iwb4YqrLjSnLPf3E2rDocg2lp4O1RmwgClREGMy-qpVbgu4Q3KFGpt5YDQF-fvEzwz_GA6AtllghsMrGETTtJyXNLxWEezwLKDnCapkk8Z1dEUbbwQ=w1170-h229-no?authuser=1)


![EDU-CIAA Pines utilizados del NXP LPC4337 (1 a 72)](https://lh3.googleusercontent.com/a6Zz4RAmktCd1wtjX0jOazxAHPTLou9IZkPK_38JOH8Y3bELrGY5qifKF37omZT7ucIdFwxpTuQ0_XqYHaWRnIgP2yEk4R0ABLJBcH6paZmY6ycLjhEaI7Y6-O68ENygez_dBsa4Z9_m38bc2prU1eEUmRuDb0oOjkj0iCArT2Gc2msjd5pUoAvXEo9wuykzkVamsEAfqlNEd4ZtiA0xoDQtMYfkusQWficygFhAfTzOEIfgSFLdV4d6b7OyKpzoQuiUpVNMxMXxg39EWSmq6yXeDJREEXtM6hC-wF05-igysMRqqFm2g37H-aXJzqpCZjxRewUk0T19EqB9RByNcuiYPN0GJAabgD3CEVE4ifKnKP64TnyhLI6wPpdlMLCRoHmgPeSfCJ4jyfm-ghUJPKf0NEVcEaqx-cF2pdQKTpiCavlpkRNjFLOg_NW8iwFil8j2RrJOz15SQRLIIEekglQsJ1-Y0vDpgt7wW9LNJGI8Rp9NFfQd91nZy-H31crHQrtUgAVFUMLfyUJtA6YetLmo6kBrpPpmNfkerqUe-a7gv2TF8syfZWUz1-JU5HGtqhfhisx8b9A_rRCq0swgOxtymRSqhOh2T-t2DZezIByX7vM2y3BAZ8_EE3kLvX40-0jMmCGT0WK_aIxMGC6nFEgNkH72XSzNfGUeYB3-fZ87v9-v2ZnosuWUrp678UR6cSLdtpOjP6EGNPxXW3JXBBA=w1268-h663-no?authuser=1)

![EDU-CIAA Pines utilizados del NXP LPC4337 (73 a 144)](https://lh3.googleusercontent.com/J0s-5Zek_ds4nGjNOEuVPwEoVa3ZHtYkMn4zpifamyfuMMhI5M84eewN--qe260DFPZMudL_D-Igx37-4TCYqhPZjkHxeBvMl2itvFb8e7wfCYpIancV3yuDnDj-BAokT4kXtHTABy1puhnzeq2hMIStc_cG99U98Dp2ND9m7f-v2V8fe7ekF1D23vbUtwgNJCNQuKoH9CgbTWl4uT41jK4ntOXoLKvudYJcZGOjm8evwTOnJJ8fR3S26bfsXpmJ47XYk2yHPPk3pzxBJ0yEKk81hOFD6qSJT433AwhavYK31V6GYlEzXWHTSJoZ2anXJg3_26CB9P2O3gmemD8n852PzwL3alJy-JbHp_jRZ5eZD-XEPWI5zBmK9i7o2unozNii7H4MMVNFg0yv5fjpZHAif49h66wFlQO4S1vEgiyVE8kP673viMqrZmU15VfVxSjEcgbiIbudyp41mJyriGp-z0zDTw8CJ-uzxbCi6flS_Jm51AAm1f38zhmpwR7sKZG0rxpqeEdGqnKQigmkIWwl1lskjwpGcsW0SYuX5rFRRnHjURt6tlMkkUa3tZVHt9Ncqiyt_kSPJ7g7VYOthdQX3RzltK49FTy1Jzq_dqRGlZ1hFpj4AgN68_iBFV03Fo7vAfzt1ivOqw8psKsDnhbwQZvo7EsQix2c8g1w0uwAyAtlRCi3etSI8XEYQOwhNBtXGBo4evoHMa8Bx3cjaYg=w1237-h663-no?authuser=1)


### **Primitivas** **y Macros**:

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

```	c
**#define** LPC_SCU_BASE 0x40086000


**Primitivas**:

En **scu-18xx:43xx.h**

**void**  **Chip_SCU_PinMuxSet**(uint8_t port, uint8_t pin, uint16_t modefunc) --> para setear el modo de funcionamiento de un determinado pin

  

**void**  **Chip_SCU_ClockPinMuxSet**(uint8_t clknum, uint16_t modefunc)

--> configura los las funciones de los pines de clock.

  

**void**  **Chip_SCU_GPIOIntPinSel**(uint8_t PortSel, uint8_t PortNum, uint8_t PinNum)

→ setea los pines GPIO que haran interrupciones

  

**void**  **Chip_SCU_I2C0PinConfig**(uint32_t I2C0Mode)

-->configura el pin de I2C

  

**void**  **Chip_SCU_ADC_Channel_Config**(uint32_t ADC_ID, uint8_t channel)

--> configura el canal de ADC que se pase como parametro.

  

**void**  **Chip_SCU_DAC_Analog_Config**(**void**)

-> habilita la funcion del DAC en el pin P4_4

  

**void**  **Chip_SCU_SetPinMuxing**(**const**  PINMUX_GRP_T *pinArray, uint32_t arrayLength)

--> se setea el modo de funcionamiento de un conjunto de pines. llama a Chip_SCU_PinMuxSet() para cada elemento del array de entrada.
```	

