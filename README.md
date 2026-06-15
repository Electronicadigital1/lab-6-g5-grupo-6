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

2. Inicialización por Comandos: Al recibir la señal de inicio (ready_i), el módulo pone la señal RS en bajo (RS = 0) para indicarle a la LCD que recibirá órdenes de configuración y no texto. En este estado, la FSM prepara la pantalla LCD para poder recibir caracteres. Para indicarle al hardware de la LCD que lo que se va a transmitir es una orden de configuración y no texto, la FSM asigna la señal de registro de selección en bajo (rs <= 1'b0). Simultáneamente, el bus de datos toma el valor guardado en una memoria ROM de configuración interna por el contador: data <= config_mem[command_counter]. En cada ciclo del reloj de 16 ms, el sistema transmite uno de los 4 comandos esenciales:

| Comando | Valor Hexadecimal | Descripción |
| :--- | :---: | :--- |
| **Configuración de funciones** | `8'h38` | Define bus de 8 bits y visualización en 2 líneas. |
| **Modo de entrada** | `8'h06` | Configura el cursor para que se mueva automáticamente a la derecha. |
| **Control de encendido** | `8'h0c` | Enciende la pantalla y desactiva el parpadeo del cursor. |
| **Limpieza de pantalla** | `8'h01` | Borra cualquier residuo en la memoria de la LCD. |

3. Escritura de la primera fila: Una vez configurada la pantalla, la FSM cambia la señal de control a alto (RS = 1), pasando a "Modo Datos". En este estado, un contador recorre los primeros 16 bytes de una memoria local (static_data_mem) donde se guardó el texto, y envía los códigos ASCII de las letras uno a uno para rellenar la fila superior de la LCD. Cada letra se plasma en la pantalla de izquierda a derecha. En cada flanco de reloj, el contador incrementa en uno (data_counter <= data_counter + 1). La máquina de estados repite este proceso de forma iterativa data_counter < 16 (NUM_DATA_PERLINE). Una vez se completa la transferencia del carácter número 16 (llenando la fila superior), la FSM transiciona hacia CONFIG_CMD2.

En estos estados, la Máquina de Estados (FSM) se encarga de leer los caracteres de la memoria interna y enviarlos a la pantalla LCD. Su comportamiento se define por las siguientes variables:

| Señal / Variable | Tipo | Descripción y Comportamiento en el Estado |
| :--- | :---: | :--- |
| **`static_data_mem`** | Entrada (Dato interno) | Memoria ROM interna de la cual se lee el código ASCII del carácter correspondiente que se va a imprimir. |
| **`data_counter`** | Entrada (Condición) | El estado evalúa su valor actual para saber qué posición de la memoria leer y determinar si ya se alcanzó el límite de la fila (`data_counter == 16`). |
| **`rs` (Register Select)** | Salida (Señal de Control) | Se asigna obligatoriamente a `1'b1` (Nivel Alto) para indicarle a la LCD que la información enviada es texto y no un comando. |
| **`data [7:0]`** | Salida (Bus de Datos) | Se le asigna el byte exacto del carácter extraído de la memoria (`static_data_mem[data_counter]`) para enviarlo físicamente a los pines de la pantalla. |
| **`data_counter`** | Salida (Actualización) | Se incrementa su valor en 1 (`data_counter <= data_counter + 1`) para que en el siguiente ciclo de reloj se apunte a la siguiente letra. |

4. Salto de Línea Interno: Físicamente, las pantallas LCD 16x2 no saltan de fila de forma automática al llenarse la primera; requieren recibir una instrucción de control que mueva el puntero de su memoria interna DDRAM. Al entrar en este estado, el hardware realiza tres operaciones inmediatas:
   
| Operación | Código Verilog | Propósito |
| :--- | :---: | :--- |
| **Limpieza del contador** | `data_counter <= 'b0;` | Reinicia el contador a cero para prepararlo y sincronizarlo con el inicio de la escritura de la siguiente línea. |
| **Cambio a modo comando** | `rs <= 1'b0;` | Regresa la línea de selección de registro a bajo, indicándole a la LCD que recibirá una instrucción de control y no un carácter de texto. |
| **Instrucción de salto** | `data <= 8'hc0;` | Deposita en el bus el comando `START_2LINE`, el cual mueve físicamente el puntero de la memoria DDRAM al inicio de la segunda fila. |

5. Escritura de la Segunda Fila: La FSM vuelve a conmutar la señal a modo datos (rs <= 1'b1) para habilitar nuevamente la impresión de caracteres ASCII. Dado que el mensaje de la segunda línea se encuentra guardado en la misma memoria ROM justo después del texto de la primera línea (posiciones de la 16 a la 31), el hardware altera el direccionamiento aplicando un desfase aritmético: data <= static_data_mem[NUM_DATA_PERLINE + data_counter]. Las letras se imprimen secuencialmente en la fila inferior de la LCD. El data_counter vuelve a incrementarse desde 0 hasta 15. Mientras esto ocurre, el estado se mantiene retenido en WR_STATIC_TEXT_2L. En el instante en que el contador registra el carácter número 16 (data_counter == 16), la transferencia del bloque estático se da por concluida de manera exitosa. En este punto, la máquina de estados realiza su acción final volviendo al estado IDLE, dejando el mensaje "enganchado" (latched) visualmente en la pantalla gracias a la memoria RAM propia del periférico.


#### Parte 2 (Dinámica) 

La segunda parte es basicamente lo mismo que la primera, usando la adaptación de la velocidad, la inicialización de comandos, la escritura de la primera fila, el salto de línea y la escritura de la segunda fila, sin embargo se le agregó tres cosas muy importantes:
1. Evaluador: Es el responsable de gestionar la condición de salida al finalizar la escritura de la segunda línea de texto. A nivel de arquitectura modular, este bloque se compone de tres elementos digitales interconectados: un registro contador (`data_counter`) que incrementa su valor con cada carácter enviado a la pantalla, un circuito comparador lógico que verifica constantemente si se ha alcanzado la capacidad máxima de la fila (`data_counter == 16`), y un multiplexor encargado del enrutamiento de la Máquina de Estados (FSM). Mientras el límite de caracteres no se alcance, el sistema se mantiene iterando en el mismo estado de escritura. Sin embargo, en el instante exacto en que el comparador detecta que la fila está completamente llena, emite una señal de control activa. Esta señal obliga al sistema a realizar un "cambio de vías" síncrono en el siguiente flanco de reloj, redirigiendo el flujo de ejecución hacia el reposo absoluto (`IDLE`) en la versión estática, o hacia el bucle infinito de refresco de variables (`WR_Din_TEXT`) en la versión dinámica.
   

| Componente Físico | Elemento en el Código | Función Principal | Comportamiento en el Sistema |
| :--- | :--- | :--- | :--- |
| **Registro (Memoria)** | `data_counter` | **Llevar el conteo** | Grupo de flip-flops que almacena el número de caracteres enviados. Se incrementa en `1` con cada pulso de reloj síncrono. |
| **Comparador Lógico** | `if (data_counter == 16)` | **Evaluar la condición** | Circuito que compara constantemente el valor del registro con el límite fijo (`16`). Emite un `0` lógico si aún faltan letras, o un `1` lógico exacto en el momento en que se llena la fila. |
| **Multiplexor (Enrutador)** | `next_state = ...` | **Ejecutar el salto** | Recibe la señal del comparador. Si recibe un `0`, enruta la FSM para que se mantenga en el mismo estado. Si recibe un `1`, "cambia las vías" y direcciona la FSM hacia su nuevo destino (`IDLE` o `WR_Din_TEXT`). |

Una vez que el evaluador confirma que los 16 caracteres de la segunda fila han sido enviados con éxito a la pantalla LCD, el diseño arquitectónico del sistema permite tomar dos caminos diferentes dependiendo del objetivo final del controlador. Esta decisión de enrutamiento define el comportamiento a largo plazo de la máquina de estados:

2. Ruta A: Terminación y Reposo (Módulo Puramente Estático)
   
Esta es la ruta seleccionada cuando el sistema solo necesita mostrar un mensaje fijo ("Hardcoded"). Al tomar este camino, la Máquina de Estados (FSM) da por concluida su rutina de inicialización y escritura, retornando a su estado de reposo absoluto (`IDLE`). Debido a que el controlador interno de la pantalla LCD posee su propia memoria de video (DDRAM), el mensaje transmitido queda retenido de forma permanente. La FPGA ya no necesita enviar más datos ni procesar lógica adicional, lo que detiene la actividad del bus y mantiene el texto "congelado" en la pantalla.

3. Ruta B: Bucle de Refresco Continuo (Módulo Dinámico)

Esta ruta se utiliza cuando el sistema actúa como un monitor en tiempo real. En lugar de "descansar", el enrutador empuja a la FSM hacia un nuevo estado cíclico. La máquina adquiere una nueva misión: monitorear ininterrumpidamente las entradas de hardware `data_1` y `data_2`. El sistema entra en un bucle infinito donde reubica estratégicamente el cursor y sobreescribe únicamente las coordenadas específicas de los datos numéricos. Esto ocurre a la velocidad dictada por la base de tiempos (`clk_16ms`), permitiendo que el usuario vea cómo los valores cambian de forma fluida y en tiempo real, sin borrar jamás los títulos fijos previamente escritos en las etapas iniciales.


### Diagramas
Se realizó el respectivo diagrama de la máquina de estado de la parte estática:

<p align="center">
  <img src="https://github.com/user-attachments/assets/0e8ca497-0ef0-459f-b8a0-c9d65b43f623" alt="Máquina de Estados Estática" width="600">
</p>
<p align="center">
  <em>Figura 3. Máquina de Estados Estática.</em>
</p>

También se realizó el diagrama de la máquina de estados dinámica:

<p align="center">
  <img src="https://github.com/user-attachments/assets/3c92131c-52e3-451e-888c-17c3a17f974a" alt="Máquina de Estados Dinámica" width="600">
</p>
<p align="center">
  <em>Figura 4. Máquina de Estados Dinámica.</em>
</p>


## Implementación

### LCD Estático

En esta parte, se implementó el código proporcionado por el docente para analizar el resultado de un LCD Estático, el cual fue el siguiente:




## Conclusiones

1. La FSM como eje de sincronización: La implementación de una Máquina de Estados Finitos (FSM) combinada con una base de tiempos ralentizada (`clk_16ms`) demostró ser indispensable para adaptar la alta velocidad de la FPGA (50 MHz) a los estrictos y lentos tiempos de respuesta que exige el hardware de la pantalla LCD 16x2.
2. Eficiencia mediante la reutilización de código: El diseño de la arquitectura destaca por su modularidad, comprobando que no es necesario diseñar un sistema desde cero para mostrar datos dinámicos. La versión dinámica reutiliza el 100% de la lógica de inicialización y escritura de etiquetas del módulo estático, optimizando así el uso de recursos lógicos de la FPGA.
3. Traducción directa de lógica a hardware: El análisis del "evaluador" (la condición de salida `data_counter == 16`) evidencia cómo las instrucciones de control de alto nivel en Verilog se materializan directamente en componentes físicos específicos, tales como registros para el conteo, comparadores lógicos para la evaluación y multiplexores para el enrutamiento de estados.
4. Actualización dinámica sin parpadeos: Se concluye que la forma más eficiente de mostrar variables en tiempo real en una LCD no es borrando y reescribiendo toda la pantalla, sino mediante el uso estratégico del modo comando para mover el puntero de la memoria DDRAM hacia coordenadas específicas, logrando sobreescribir únicamente los datos numéricos y preservando el texto estático.


## Referencias
[1] Electronicadigital1, "Lab_6," Repositorio del curso 2026-1, GitHub, 2026. [En línea]. Disponible en: https://github.com/Electronicadigital1/2026-1/tree/main/Labs/Lab_6.
