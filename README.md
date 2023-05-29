# L5_Descripcion_de_puertos_GPIO
Este repositorio contiene el código y documentación correspondiente a encender 10 led a foma de contador binario. Funciona con 2 botones uno para incrementar en uno la cuenta y otro para decrementarla


## Funcionamiento del proyecto

Este proyecto fue escrito sobre la plantilla https://github.com/Ryuuba/arm-gpio-template.git**.** 

El proyecto está compuesto por los archivos 

### main

Consiste en habilitar 10 puertos de escritura [PA0-PA9] como salidas a las que se conectaran los leds y 2 puertos de lectura [P10-P11]

Una vez habilitados los puertos. Da comienzo un loop infinito que verifica el estado de los botones bajo la lógica siguiente:

```c
buttonA = readbutton (PortA, pin10)
buttonA = readbutton (PortA, pin10)
if(buttonA && buton B)
	counter = 0
else if (buttonA && !buton B)
	counter ++;
if(!buttonA && buton B)
	counter --
output (counter)
```

**Marcos** 

| Variables | Compensación |
| --- | --- |
| counter | r7 |
| buttonA | r7+4 |
| buttonB | r7+8 |
|  |  |
| r7 |  |
| lr |  |

### digital_read

Función encargada de leer la entrada de los botones y hacer el desplazamiento necesario para obtener el bit correspondiente al botón.

### ouput

Esta función recibe como argumento un entero de 10 bits correspondiente al contador, está encargada de emitir el valor del contador a través de un puerto digital correspondiente a los 10 leds.

Se utiliza la máscara `0x3FF` correspondiente a 1032 para mostrar el contador en 10 leds.

### read_button

Esta función consiste en un contador antirrebote. Para lograr su objetivo realiza los siguientes.

Comienza leyendo el estado del botón 0 (apagado) o 1 (encendido). Si el botón no está encendido regresa un falso, en caso contrario entra a un ciclo for que verifica si se obtienen 4 o más estados de encendido seguidos.

En caso de que esta condición no se cumpla.

Como argumentos recibe el puerto y pin a leer

```c
#define DELAY 50
#define SAMPLES 10
#define POSITIVE_READINGS 4

int digital_read(int port, int pin);
void delay(int ms);

int read_button(int port, int pin)
{
    int bit = digital_read(port, pin);
    if (!bit)
        return 0;
    int counter = 0;
    for (int i = 0; i < SAMPLES; i++)
    {
        delay(5);
        bit = digital_read(port, pin);
        if (!bit)
            counter = 0;
        else{
            counter++;
            if (counter >= POSITIVE_READINGS)
                return 1;
        }
    }
    return 0;
}
```

**Marcos** 

| Variable | Compensación |
| --- | --- |
| pin | r7 |
| port | r7+4 |
|  | r7+8 |
| bit | r7+12 |
| i | r7+16 |
| counter | r7+20 |
| r7 |  |
| lr |  |

La lógica de esta función fue extraída del libro Zhu, Y. (2017). *Embedded Systems with Arm Cortex-M Microcontrollers in Assembly Language and C: Third Edition*. Capítulo 14.8

### delay

La función delay es una función que simula un retraso o espera a través de 2 ciclos for anidados. 

Esta función tiene 3 variables:

- iterador i
- iterador j
- ticks

Y un argumento: ms 

Corresponde a la implementación en ensamblador de la siguiente función:

```cpp
void wait_ms(ms){
    int ms, i, j;
    for(i = 0; i < ms; i++)
        for(j = 0; j < 255; j++); // adjust 255 to achieve 1 ms delay
}
```

Esta función fue extraída del libro Zhu, Y. (2017). *Embedded Systems with Arm Cortex-M Microcontrollers in Assembly Language and C: Third Edition*. Capitulo 14.8

### Modificaciones para encender 10 leds

- **Configurar los puertos de entrada y salida**
    
    De tal forma que se etsbalce los pines A0-A9 como salidas y A10-A11 como entradas.
    
    Al realizar las modificaciones el setup queda de la siguinete froma:
    
    ```nasm
    setup: @ Starts peripheral settings
            # enables clock in Port A
            ldr     r0, =RCC_BASE
            mov     r1, #4
            str     r1, [r0, RCC_APB2ENR_OFFSET]
            # configures pin 0 to 7 in GPIOA_CRL
            ldr     r0, =GPIOA_BASE 
            ldr     r1, =0x33333333 
            str     r1, [r0, GPIOx_CRL_OFFSET] 
            # configures pin 8 to 15 in GPIOA_CRL
            ldr     r1, =0x44448833
            str     r1, [r0, GPIOx_CRH_OFFSET] 
    ```
    
- **Establecer equivalencias de pines de entrada**
    
    En necesario modificar los valores de los pines de entrada en este caso se cambia 
    
    ```nasm
    .equ PIN5, 0x5
    .equ PIN6, 0x6
    ```
    
    Por
    
    ```
    .equ PIN10, 0x10
    .equ PIN11, 0x11
    ```
    
    Y esta modificacion se hace todo el codigo donde aparezcan estas equivalencias.
    
- **Modificar la máscara en el archivo output**
    
    Para encende 10 leds la mascara debe ser 1023. 
    
    Al ser una constante que sale del ramgo es necesario cargarla en un registro antes.
    
    El codigo modificado queda de la siguinte froma:
    
    ```nasm
    output:
            ldr     r1, =GPIOA_BASE
            ldr     r2, =0x000003FF
            and     r0, r2
            str     r0, [r1, GPIOx_ODR_OFFSET]
            bx      lr
    ```
    

## Forma de compilar el software

Una vez se ha terminado el programa es necesario obtener el binario para poder cargarlo en el µC utilizando los siguientes comandos:

### 1. Ensamble

Ensambla el contenido del archivo main.s mediante la ejecución del comando. En este caso, el archivo **main.s** contiene el código principal del programa

`arm-as main.s -o main.o`

Que dará como resultado el código objeto

### 2. Construcción del objeto binario

Se construye el archivo binario a través del objeto **main.s** utilizando el comando:

`arm-objcopy -O binary main.o main.bin`

### 3. Escritura en el µC

Para escribir en el µC hay 2 opciones:

- Utilizando el comando
    
    `st-flashwrite 'main.bin' 0x8000000`
    
- Utilizando el software STM32 CubeProgramer

---

O bien, al utilizar la nueva plantilla es posible crear directamente el archivo binario utilizando el comando `make`. Dicho comando creará el archivo `prog.bin`

## Diagrama de configuración de hardware
![image](https://github.com/NataliaVictoria/L5_Descripcion_de_puertos_GPIO/assets/89500688/56669772-981d-4123-af7a-9a195ac8339c)
