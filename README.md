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


### Diagramas



## Implementación


## Conclusiones


## Referencias

