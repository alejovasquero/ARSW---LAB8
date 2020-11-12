### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

#### Informe comparativo

Una vez hechas las pruebas, notamos una enorme diferencia en tiempo de respuesta total con respecto
a una iteración de newman.

![](../images/part2/images/GRAPH.PNG) 

Entre escalado horizontal y vertical vemos un incremento en tiempo de ***14.3%***.
Por el momento vemos que tener una máquina de mejor calidad vale más que tener 3 máquinas
iguales que se reparten la carga.

![](../images/part2/images/GRAPH2.PNG) 

Este decremento en tiempo está dado con un incremento en costo del ***433%*** con 
respecto a la máquina sencilla de 1 core.

La cantidad de respuestas respondidas está dada por el siguiente gráfico. 

![](../images/part2/images/GRAPH3.PNG) 

La máquina fuerte resultó ser menos confiable que las 3 máquinas al momento de recibir varias peticiones
de manera concurrente.

Como conclusión vemos que ambas aproximaciones tiene sus pros y contras, siendo la más
barata la instalación de 3 equipos, y también más confiable con respecto a la recepción de
solicitudes, pero es menos eficiente en términos de procesamiento, por lo que tener una máquina más fuerte es una opción,
pero costará 4.3 veces en este caso.

