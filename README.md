# L5_Descripcion_de_puertos_GPIO
Este repositorio contiene el código y documentación correspondiente a encender 10 led a foma de contador binario. Funciona con 2 botones uno para incrementar en uno la cuenta y otro para decrementarla

## Funcionamineto del proyecto

Este proyecto fue escrito sobre la plantilla **[arm-blink](https://github.com/Ryuuba/arm-blink).** 

Por lo que se agregaron 3 archivos correspondientes a las 3 funcione necesarias para el funcionamineto del proyecto.

### main.s

Consiste en habitlitar 10 puertos de escritura [PA0-PA9] como salidas a las que se conectrarán los leds y 2 puertos de lectura P10-P11

Una vez habilitados los puertos.

Se establece un 0 en las salid y se inicializa un contador en 0 

Y entra a un loop que se repetira de forma infinita. Este loop verifica el estado del boton A si este esta presionado verifica si el boton B esta presionado. Si ambos botones etan presionados restablece el valor del contador a 0. Si solo el boton A esta presionado incrementa en 1 el contador y si solo el boton B esta presionado incrementa el contador en 1.

```cpp
// Argumnetos = 0
// Variables = 1    
//      contador
int main(){
    //Notas 0 es apagado 1 es encendido 
    Habilitar 10 puertos de escritura -> ORD
    Habilitar 2 puertos de lectura -> IRD
        //Para habilitar pines como salida = 3, entrada 8
    counter = 0;    //Inicializa contador 
    loop(){
        read buttonA input;
        if(buttonA = 0){           //Si ambos son 0 no hace nada
            read buttonB input;
            if(buttonB = 0){
            }else
                counter --;         //Si button B es 1 decrementa
        }else{
            if(buttonB = 0)
                 counter ++;        //Si button A es 1 incrementa
            else
                counter = 0;        //Si ambos estan presionados el contador reinicia 
        }     
    }
}
```

### is_button_pressed

Esta funcion consiste en un contador antirrebote. Para lograr su objetivo realiza los siguiente.

Comienza leyendo el estado del boton 0 (apagado) o 1 (encendido) . Si el boton no esta encendido regresa un falso, en caso contrarrio entra a un ciclo for que verifica si se obtienen 4 o más estados de encendido seguidos.

En caso de que esta condicion no se cumpla

```cpp
// Argumentos = 1
// Variables = 2
//      contador
//      i
bool is_button_pressed(pin){
    read button input;
    if (button is not pressed)      //Si la lectura es 0 no esta presionado
        return false;
    int counter 0;
    for(i = 0; i < 10; i++){
        wait 5 ms;          //Funcion
        read button input;  //
        if (button is not pressed) {
            counter = 0;                // bounce, reset counter
        } else {
            counter = counter + 1;      // stable, increase counter
        if (counter >= 4)               // require 4 consecutive positive readings
            return true;
        }
    }
return false;
}
```

Esta funcion fue extraida del libro Zhu, Y. (2017). *Embedded Systems with Arm Cortex-M Microcontrollers in Assembly Language and C: Third Edition*. Capitulo 14.8

### delay

La funcion delay es una funcion que simula un retrazo o espera a traves de 2 ciclos for anidados. Corresponde a la implemnetacion en ensamblador de la siguiente funcion:

```cpp
//#include <iostream>
// Argumnetos = 1
// Variables = 3
//     - iterator i
//     - iterator j
//     - ticks
void wait_ms(ms){
    int ms, i, j;
    for(i = 0; i < ms; i++)
        for(j = 0; j < 255; j++); // adjust 255 to achieve 1 ms delay
}
```

Esta funcion fue extraida del libro Zhu, Y. (2017). *Embedded Systems with Arm Cortex-M Microcontrollers in Assembly Language and C: Third Edition*. Capitulo 14.8

## Fomar de compilar el software

Una vez se ha termiando el programa es necesario obtener el binario para poder cargarlo en el µC utilizando los siguinetes comandos:

### 1. Ensamble

Ensambla el contenido del archivo main.s mediante la ejecucion del comando. En este caso el archivo **main.s** contiene el codigo principal del programa

`arm-as main.s -o main.o`

Que dará como resultado el código objeto

### 2. Construcción del objeto binario

Se construye un onjeto binario a traves del objeto **main.s** utilizando el comando:

`arm-objcopy -O binary main.o main.bin`

### 3. Escritura en el µC

Para escribir en el µC hay 2 opciones:

- Utilizando el comando
    
    `st-flashwrite 'main.bin' 0x8000000`
    
- Utilizando el software STM32 CubeProgramer

## Diagrama de configuracion de hardware

![Untitled](L5_Descripcion_de_puertos_GPIO%208fe019e28aad4edc9122462229408efb/Untitled.png)
