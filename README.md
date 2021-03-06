### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

![Imágen 1](images/part1/part1-vm-basic-config.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

![](images/part1/recursos.png)

2. ¿Brevemente describa para qué sirve cada recurso?

    * ***Network watcher***: Supervisa, diagnostica y obtiene información sobre el rendimiento y el estado de la red
    * ***Virtual network***: Construcción de redes virtuales
    * ***Máquina virtual***: Máquina de trabajo
    * ***Ip***: IP pública para nuestra máquina
    * ***Security group***: Filtrado de tráfico de red
    * ***Disco***: Disco de sistema.
    * ***Vertical scalability***: Interfaz de red para nuestra máquina
    * ***Key***: Llave SSH para conexión
    
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

    * Al iniciar una sesión SSH, los procesos creados por esta quedarán enlazados e esta sesión.
    Una vez la sesión se cierre, todos los procesos y sus hijos serán cerrados.
    * El puerto 3000 se abre debido a que por defecto todos los puertos estarán cerrados, a excepción
    de los más importantes, como 22 de ssh o el 80 de web.
    
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

![](images/part1/beforeEV/docTimes0.png)

![](images/part1/beforeEV/docTimes.PNG)

La funcion recibe un numero, pero por lo que es tan grande el numero a calcular, la funcion ocupa mas capacidad de procesamiento y memoria para calcular el valor para devolver.

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
![](images/part1/beforeEV/docMachine.PNG)

El procesamiento es alto y se refleja en el reporte de la maquina, que se usa un 100% de capacidad para la funcion, adicionalmente se puede ver que el uso del disco es critico durante un momento mientras se usa la funcion.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición. 
    
        Para petición antes de realizar escalado se obtuvieron los siguientes resultados.
        
        ![](images/part1/beforeEV/docTimes.PNG)
        
        Desde el browser, las peticiones rozaban los 10 minutos máximo.
        Con postman se obtuvieron peores resultados.
        
        ![](images/part1/beforeEV/docPostman2.png)
        
        Estos resultados están justificados teniendo en cuenta el excesivo uso de CPU.
        
        ![](images/part1/beforeEV/docTimes.PNG)
        
        Para solucionar esto, hacemos escalado vertical, lo que nos da los siguientes resultados.
        
        ![](images/part1/afterEV/docTimes.PNG)
        
        Las respuestas han llegado a segundos, y postman se ha completado más rápido.
        
    * Si hubo fallos documentelos y explique.

    
        Durante Postman se encontró el siguiente fallo.
        
        ![](images/part1/afterEV/docPostmanERR.png)
        
        Este error debe ser causado debido a que al hacer escalado, la máquina está
        realizando tareas de configuración, por lo que cualquier petición no será atendida.
        
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

    - B1ls: 
  Solo funciona en Linux
  CPU: 1
  RAM: 0.5
  Temp SSD: 4 
  Discos Maximos: 2
  Costo: 3,80 US$
     - B2ms
  CPU: 2
  RAM: 8
  Temp SSD: 16 
  Discos Maximos: 4
  Costo: 60,74 US$
  
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

    * En este escenario, el cambio en procesamiento es del doble, y en memoria. se ve una mejora de 16 veces la inicial.
    Dado que la máquina es bastante básica, realizar mejoras muestra resultados notables, en nuestro caso, de minutos.
    
        En el futuro, entre más actualizaciones, quizás la mejora sería cada vez más pequeña. 
    
    * Al escalar verticalmente se tuvo que volver a iniciar la aplicación, debido a que hubo algún reinicio o cambio de procesos en la máquina.
    
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

    Como vimos en el punto anterior, al realizar mejoras, se implica un reinicio. lo que 
    no es conveniente cuando se tienen arquitecturas basadas en la alta disponibilidad 
    y donde no se puede perder información que está en memoria. 

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

    La gráfica muestra la mejora a través del tiempo.

    ![](images/part1/afterEV/docMachine.PNG)
    
    Está mejora está dada por el aumento en unidades de procesamiento y en memoria, 
    lo que ayuda mucho en tareas que requieran alto almacenamiento que no tenga que irse a disco, lo que produce latencias muy altas.
    
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

    La petición original de fibonacci de 1000000 duraba 4 minutos en un browser, pero por medio de postman y escalamiento vertical,
    se obtuvo un promedio de 27 segundos de respuesta.
    
     ![](images/part1/afterEV/mejora.PNG)

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

![](images/part2/images/page.PNG)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

SKU (Stock Keeping Unit)

Para hacer un seguimiento de un conjunto de recursos se necesita identificar que productos se usan para disminuir costes adicionales.

Comprende las aplicaciones, metricas de licencias y terminos basicos que se puedan adicionar a la identificacion del producto. Tambien se pueden ver que derechos existen sobre este.

Pueden haber muchas maneras de identificar los productos, EAN, UPC, SKU pueden ser alternativas que se busquen para abordar el maximo de informacion que se pueda dar sobre el recurso, se difernecian en el tipo de codificacion y el tamaño que posee la identificacion, el uso externo e interno de los estandares.

Traffic Manager
Proporciona equilibrio de carga para el DNS global. Analiza las peticiones DNS que entran para responder a un punto optimo de conexion, que puede ser personalizado con la directiva de enrutamiento.

Application Gateway
Provee funcionalidades con equilibrio de carga de nivel 7 para la aplicacion, entrega aplicaciones y puede configurarse como ina puerta accesible desde internet.

Load Balancer:
Proporciona servicios de equilibrio de carga de capa 4 de alto rendimiento y baja latencia para protocolos UDP y TCP. Posee opciones de sondeo de estado en TCP y HTTP.

El balanceador de carga necesita una ip publica porque el cliente cuando hace una peticion al servidor, realmente esta accediendo al balanceador de carga para que de esta manera el balanceador pueda atender la peticion en un servidor disponible. Adicionalmente puede nombrarse con una identificacion que le permita al DNS buscar de manera rapida el acceso.

* ¿Cuál es el propósito del *Backend Pool*?

Define el conjunto de recursos que serviran para el trafico de las reglas en el balanceo de carga.

* ¿Cuál es el propósito del *Health Probe*?

Pueden detectar fallos en una aplicacion, como endpoints desde el backend, tambien para supervisar tiempos de carga y descarga.

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

Pueden haber reglas internas o externas que permitan la distribucion optima de las peticiones que se reciben, puede afectar la escalabilidad a reglas que esten muy limitadas o sean demasiado especificas para ciertos casos donde se quiere aumentar el rendimiento de la aplicacion.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

Virtual Network
Permiten la comunicacion segura entre recursos de Azure y el internet, ademas provee mas beneficios para la infraestructura, como por ejemplo el aislamiento de zonas de trabajo, escalamiento y disponibilidad.

Subnet

Permiten segmentar la red en una o mas subredes y desplegar recursos en estas zonas.

Address space

Se asigna un espacio de direcciones para que Azure pueda asignar direcciones en ese espacio a los recursos que se necesiten para el despliegue de los servicios.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

Azure opera globalmente, por lo cual el cliente puede modificar las zonas para obtener el nivel mas optimo que le permita cumplir con sus necesidades.

La redundancia se basa en la replicacion de las instancias para que permita un aseguramiento de los datos y evitar perdidas de informacion.

Azure permite poner zonas de disponibilidad en respuesta a posibles fallos, de manera que ofrece alta disponibilidad, son unicas y pertenecen a areas especificas en una region

* ¿Cuál es el propósito del *Network Security Group*?

Filtrar el trafico de la red de recursos de azure en una red virtual, contiene reglas de seguridad que permiten o deniegan por la entrada y salida de algun recurso, puerto y protocolo del destino.

* Informe de newman 1 (Punto 2)

[INFORMES](informes/parteA.md)

* Presente el Diagrama de Despliegue de la solución.

![](images/part2/images/components.png)




