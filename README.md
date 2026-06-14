[![Open in Visual Studio Code](https://classroom.github.com/assets/open-in-vscode-2e0aaae1b6195c2367325f4f02e2d04e9abb55f0b24a779b69b11b9e10269abc.svg)](https://classroom.github.com/online_ide?assignment_repo_id=24107422&assignment_repo_type=AssignmentRepo)
# Lab06 - Visualización usando pantalla LCD 16x2

# Integrantes
- Paula Cristina Hincapie Peña 1016942585
- Catalina Silva Fandiño 1014857784
- 16/06/2026

# Informe

Indice:

1. [Descripción](#descripción)
2. [Objetivo](#objetivos)
3. [Diseño implementado](#diseño-implementado)
4. [Simulaciones](#simulaciones)
5. [Implementación](#implementación)
6. [Conclusiones](#conclusiones)
7. [Referencias](#referencias)


## Descripción

En el diseño de sistemas digitales basados en FPGA, la comunicación con periféricos de visualización es un reto que requiere un control preciso de los tiempos y secuencias de datos. El presente informe detalla el diseño y la implementación de un controlador en hardware (usando Verilog) para una pantalla LCD alfanumérica de 16x2. Este informe cuenta con dos partes. La primera parte (estática) tiene como objetivo inicializar la pantalla LCD 16x2 y escribir un mensaje fijo (que no cambia) de hasta 32 caracteres en total (16 por cada fila) directamente desde la FPGA. La segunda parte ( dinámica) se encarga de actualizar y mostrar información que cambia en tiempo real (valores provenientes de las entradas externas data_1 y data_2, que en la práctica podrían ser lecturas de sensores, interruptores o contadores de la FPGA) en posiciones específicas de la pantalla, sin borrar ni alterar el texto estático que se escribió al principio.

## Objetivos
- Introducir el concepto de máquina de estado en el diseño de hardware utilizando Verilog, enfocado en el control y funcionamiento de una pantalla LCD.

## Diseño implementado

#### Parte 1 (Estática)

Para la primera parte se utilizaron los recuersos que el profesor del Laboratorio puso a disposición. Con esto lo que se hizo fue inicializar la pantalla LCD 16x2 y escribir un mensaje fijo (que no cambia) de hasta 32 caracteres en total (16 por cada fila) directamente desde la FPGA. Para lograr esto, el módulo ejecuta de manera ordenada y secuencial las siguientes tareas:
1. Adaptación de la velocidad: La FPGA opera a una frecuencia muy alta (50 MHz), pero la pantalla LCD es un periférico antiguo y lento. El módulo incluye un contador interno que reduce esta velocidad, generando un pulso lento (cada 16 ms) asignado a la señal enable. Esto garantiza que la LCD tenga el tiempo suficiente para procesar cada instrucción.
   
T = COUNT_MAX / f_clk
T = 800,000 / 50,000,000 Hz
T = 0.016 s = 16 ms

2. Inicialización por Comandos (CONFIG_CMD1): Al recibir la señal de inicio (ready_i), el módulo pone la señal RS en bajo (RS = 0) para indicarle a la LCD que recibirá órdenes de configuración y no texto. Envía secuencialmente 4 comandos predefinidos (como configurar el bus a 8 bits, encender la pantalla, activar el cursor y limpiar la pantalla). En este estado, la FSM prepara la pantalla LCD para poder recibir caracteres. Para indicarle al hardware de la LCD que lo que se va a transmitir es una orden de configuración y no texto, la FSM asigna de manera estricta la señal de registro de selección en bajo (rs <= 1'b0). Simultáneamente, el bus de datos toma el valor guardado en una memoria ROM de configuración interna indexada por el contador: data <= config_mem[command_counter]. En cada ciclo del reloj de 16 ms, el sistema transmite uno de los 4 comandos esenciales:

| Comando | Valor Hexadecimal | Descripción |
| :--- | :---: | :--- |
| **Configuración de funciones** | `8'h38` | Define bus de 8 bits y visualización en 2 líneas. |
| **Modo de entrada** | `8'h06` | Configura el cursor para que se mueva automáticamente a la derecha. |
| **Control de encendido** | `8'h0c` | Enciende la pantalla y desactiva el parpadeo del cursor. |
| **Limpieza de pantalla** | `8'h01` | Borra cualquier residuo en la memoria de la LCD. |


### Diagramas



## Implementación


## Conclusiones


## Referencias

