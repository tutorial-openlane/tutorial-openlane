# Tutorial OpenLane

1. [Contenedor Docker](#instalar-el-contenedor-docker)
2. [Herramientas e interfaz de trabajo](#herramientas-de-diseño-e-interfaz-de-trabajo)
3. [Ejecutar un flujo de diseño](#ejecutar-el-flujo-de-diseño-con-openlane)
4. [¿Cómo interactuar con archivos locales desde el contenedor?](#cómo-interactuar-con-archivos-locales-desde-el-contendor)
5. [TigerVNC](#tigervnc)
6. [Ejecutar Yosys](#ejecutar-yosys-desde-el-contenedor)
7. [Referencias](#referencias)

## Instalar el contenedor Docker:

1. Antes de instalar Docker Engine por primera vez, se necesita configurar el APT (Advanced Package Tool) del repositorio de Docker:

    ```
    sudo apt-get update
    sudo apt-get install ca-certificates curl
    sudo install -m 0755 -d /etc/apt/keyrings
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    ```
    
2. Ahora se debe agrega el repositorio a las fuentes APT:
    
    * Para linux Mint:

        ```
        echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        ```

    * Para Ubuntu:

        ```
        echo \
        "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
        $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
        sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
        ```

    Actualice:

    ```
    sudo apt-get update
    ```

3. Instalar los paquetes de Docker.

    Para instalar la versión más reciente se debe ejecutar:

    ```
    sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
    ```

4. Verifique que la instalación haya sido exitosa ejecutando la     imagen ```hello-world```:
    ```
    sudo docker run hello-world
    ```

    Este comando descarga una imagen de prueba y la ejecuta en un contenedor. Cuando el contenedor se ejecuta, imprime un mensaje de confirmación y luego se cierra.

5. Hacer Docker disponible sin root: Para permitir que los usuarios sin privilegios de administrador (root) puedan ejecutar comandos de Docker se debe ejecutar en la terminal:

    ```
    sudo groupadd docker
    sudo usermod -aG docker $USER
    sudo reboot # REBOOT!
    ```

    Verifique que ya no se necesitan privilegios de root:

    ```
    docker run hello-world
    ```

    **Este paso es obligatorio, sin esto todos los scripts de OpenLane fallarán** 

## Herramientas de diseño e interfaz de trabajo:

1.  Clonar el siguiente [repositorio](https://github.com/iic-jku/IIC-OSIC-TOOLS):

    ```
    git clone --depth=1 https://github.com/iic-jku/iic-osic-tools.git
    ```

2.  En la siguiente tabla se enlistan un conjunto de comandos  que se debe ejecutar para configurar el entorno de trabajo de OpenLane con un PDK específico, es este caso **SkyWater Technologies sky130A**:

    | SkyWater Technologies `sky130A` |
    |---|
    | `export PDK=sky130A` |
    | `export PDKPATH=$PDK_ROOT/$PDK` |
    | `export STD_CELL_LIBRARY=sky130_fd_sc_hd` |


5. Luego se debe ejecutar el siguiente comando:

    1. Entrar a la carpeta del repositorop clonado:
        ```
        cd iic-osic-tools
        ```

    2. Ejecutar el comando:

        ```
        ./start_vnc.sh
        ```

    El objetivo del *script* [start_vnc.sh]() es iniciar un servidor VNC (Virtual Network Computing) dentro del contenedor, pero la primera vez que se ejecuta este comando, Docker empezará a  descargar las capas de la imagen indicada en el archivo [Dockerfile](https://github.com/iic-jku/IIC-OSIC-TOOLS/blob/main/_build/Dockerfile) del repositorio.

    Una vez que Docker descarga todas las capas, se crea un contenedor basado en esa imagen.

6. Volver a ejecutar el comando:

    ```
    ./start_vnc.sh
    ```

    Ahora, como ya se configuró el contenedor, aparecerá en la terminal lo siguiente:

    ![alt text](/img/image.png)

    Presione ```S``` para iniciar, como lo indica.

    Lo anterior hará que:

    * El puerto ```80``` sea usado para la interfaz web (a través de noVNC).
    
    * El puerto 5901 será usado para el acceso al servidor VNC.

7. Ahora se puede acceder al entorno de escritorio a través de un navegador en: 
    
    ```
    http://localhost/?password=abc123
    ```

    Allí deberá encontar la siguiente interfaz:

    ![alt text](/img/noVNC.png)

    En la parte inferior izquierda podrá encontrar iconos para acceder al gestor de carpetas del contenedor y a la términal.


## Ejecutar el flujo de diseño con Openlane: 

Para comprobar la instalacion de Openlane, correr el flujo de diseño con el ejemplo spm: 

```
python3 -m openlane --run-example spm
```

Esto creará una carpeta en la carpeta designs del contenedor, y en la carpeta eda/designs que se encuentra en el home de la máquinal local. Dentro de la carpeta designs/spm, se creará la carpeta llamada ```runs``` donde se almacenan todos los resultados y archivos generados durante el flujo de diseño.

   

<!---## Ejecutar un flujo de diseño:

Acceda a la terminal que se ve en la interfaz y ejecute los siguientes pasos:

1. En el contenedor, en la ubicación:

    ```
    cd /foss/tools/openlane/2023.08/designs/
    ```

    encontrará dos ejemplos llamados ```spm``` y ```ci```, copie alguna de estas dos carpetas a un directorio en donde se tengan permisos de edición, por ejemplo el directorio ```/headless```. Esto se puede realizar por consola o manualmente usando el gestor de archivos que se encuentra en la parte inferior izquierda del servidor VNC.
   


3. Entre a la carpeta copiada de alguno de los ejemplos, en el directorio seleccionado, por ejemplo :

    ```
    cd /headless/spm 
    ```

    Allí encontrá los *scripts* HDL del ejemplo en la carpeta ```src``` y además encontrará dos *scripts*:
        
    * ```config.json```: Contiene configuraciones específicas para el flujo de diseño, como la ruta de los archivos Verilog, las restricciones de reloj, y las configuraciones del PDK, entre otros.
        
    * ```pin_order.cfg```: Se utiliza para definir la asignación de pines de I/O del diseño.

4. Para correr un flujo de OpenLane es necesario hacer uso del archivo ```flow.tcl``` que se encuentra en la carpeta de Openlane, para lo cual, una vez esté en el directorio donde se encuentra el archivo ```config.json```, deberá ejecutar el flujo de la siguiente forma:

```
/foss/tools/openlane/2023.08/flow.tcl
```
        
Esto creará una carpeta en el directorio del diseño, en este caso ```/headless/spm```, llamada ```runs``` donde se almacenan todos los resultados y archivos generados durante el flujo de diseño.
-->







## ¿Cómo interactuar con archivos locales desde el contendor?

1. ### Montar un Volumen

    1. Pasos para evitar conflictos de puertos:
    
        1. Revisar los contenedores en ejecución:
    
            ```
            docker ps
            ```
        2. Detener el contenedor que esté utilizando el puerto conflictivo: Si el contenedor que ya está en ejecución está utilizando el puerto 5901 o 80, se debe detener ese contenedor    con el comando:
    
            ```
            docker stop <container_id>
            ```
    
    2. **Montar un volumen (bind mount)**:
    
        Se puede montar un directorio de la máquina local, es decir, nuestro computador personal, en el contenedor para compartir archivos, usando el siguiente comando:
        
        ```
        docker run -d -v /ruta/host:/ruta/contenedor -p 5901:5901 -p 80:80 hpretl/iic-osic-tools
        ```
        en donde:
    
        1. ```docker run```: Ejecuta un contenedor Docker, es decir, crea e inicia un nuevo contenedor a partir de la imagen que se espcifique más adelante en el comando.
    
        2. ```-d```: (de detached) indica que el contenedor se debe ejecutar en segundo plano, es decir, el contenedor se ejecutará en el fondo y no ocupará la terminal donde se ejecutó el comando. Esto permite que se pueda seguir usando la terminal mientras el contenedor está corriendo.
    
        3. ```-v /ruta/host:/ruta/contenedor```:
        
            * La opción ```-v``` se utiliza para montar un volumen entre LA máquina local y el contenedor. Esto te permite compartir directorios entre ambos entornos.
    
            * ```/ruta/host```: es la ruta en la máquina local donde están los archivos que se quieren usar.
    
            * ```/ruta/contenedor```:  es la ruta dentro del contenedor donde se montará esa carpeta. Para saber cuáles directorios hay en la imagen de Docker ```hpretl/iic-osic-tools``` se puede:
    
                * Ejecutar el contenedor en modo interactivo:
    
                    ```
                    docker run -it hpretl/iic-osic-tools --skip bash
                    ```
    
                    Este comando permitirá ingresar al contenedor. Por lo general el contenedor ```hpretl/iic-osic-tools``` está configurado para utilizar ```/foss/designs``` como un directorio de trabajo y es el lugar donde normalmente se almacenan y manipulan los archivos de diseño. Este es el directorio predeterminado al que se accede al ejecutar el contenedor con el comando anterior.
    
                * Una vez dentro del contenedor, se podrá explorar los directorios existentes o crear nuevos directorios según se requiera.
    
        4. ```-p 5901:5901```: Mapea el puerto ```5901``` del contenedor al puerto ```5901``` en la máquina local, para poder acceder al contenedor usando **VNC**. Esto es específico para el caso en que se desee unar VNC, de lo contrario no es necesario agregar esta parte al comando, ya que hasta el momento, se ha utilizado el contenedor en la forma **NoVNC** accediendo a la interfaz del mismo a través de ```http://localhost/?password=abc123```. Si se desea acceder al contenedor en usando **VNC** se recomienda que revisar esta [sección](#tiger-vnc).
    
        5. ```-p 80:80```: Mapea el puerto ```80``` del contenedor al puerto ```80``` en la máquina local, para acceder a la interfaz web (como noVNC) como se explicó en la anterior [sección](#herramientas-de-diseño-e-interfaz-de-trabajo).
    
        6. ```hpretl/iic-osic-tools```: Especifica la imagen de Docker que se usará para crear y ejecutar el contenedor.
    
    
        Una vez ejecutado este comando, se podrá interactuar con los archivos de la carpeta especificada (```/ruta/host```) tanto desde la interfaz NoVNC como desde la interfaz VNC (cuya configuración se detallará en la [siguiente sección](#tigervnc)). Los cambios realizados en los archivos, ya sea de manera local o dentro del contenedor, se reflejarán automáticamente en ambas ubicaciones.        


2. ### Utilizar carpeta local:

    La carpeta eda se crea automáticamente como parte de la instalación de una herramienta o entorno relacionado con diseño EDA, como es el caso de OpenLane. Entonces la carpeta designs del contenedor se encuentra enlazada a la carpeta eda/designs que esta en el home de la máquina local. Por lo tanto en esta carpeta se pueden guardar los proyectos que se quieran con el contenedor para usar las herramientas de diseño.  




## TigerVNC

### Introducción

VNC y noVNC son tecnologías que permiten acceder a escritorios remotos, pero tienen diferencias clave que pueden hacer que uno sea preferible sobre el otro dependiendo del caso de uso.

#### VNC (Virtual Network Computing)

VNC es una tecnología de escritorio remoto que transmite la salida gráfica de una máquina remota a través de un servidor VNC. Funciona instalando un servidor VNC en la máquina remota y un cliente VNC en la máquina local.


#### noVNC (VNC a través de WebSockets)

noVNC es una versión de VNC que permite acceder a un servidor VNC usando un navegador web, lo que elimina la necesidad de instalar software adicional en el dispositivo cliente.

Existen varias razones para preferir uno sobre otro, pero en este caso es preferible usar el cliente VNC nativo, ya que generalmente ofrece una experiencia más fluida y eficiente, como por ejemplo, una de las características en la que VNC generalmente tiene ventajas sobre noVNC es en el manejo de funcionalidades como copiar y pegar entre el escritorio remoto y el dispositivo local.

En este contexto surgen herramientas como TigerVNC, un software cliente y servidor para la tecnología VNC, a continuación se explica cómo configurarlo:

1. En la máquina local (host): Ejecutar los siguientes comandos para instalar el cliente TigerVNC:

    ```
    sudo apt-get update
    sudo apt-get install -y tigervnc-viewer
    ```

2. En el contenedor:  Iniciar un servidor VNC dentro del contenedor

    ```
    vncserver :1 -geometry 1680x1050 -depth 24
    ```

    Aunque es probable que después de ejecutar este comando en el contenedor, aparezca un mensaje que indique que ya se está ejecutando ```Xtigervnc server```.

3. En la máquina local (host): Conetarse al contenedor usando VNC ejecutando el siguiente comando:

    ```
    vncviewer localhost:5901
    ```

    Cuando se ejecuta este comando, el cliente VNC se conecta al servidor VNC que está corriendo en el display 1 del contenedor. Esto  permitirá ver y controlar el entorno gráfico del contenedor de manera remota a través de una interfaz gráfica de escritorio.
   Antes de ingresar al contenedor se abrira una ventana emergente donde se debe colocar la siguiente contraseña: ```abc123``` 
   

    Si se configura correctamente, se abrirá una ventana del cliente VNC que muestra el escritorio del contenedor, y se podrá interactuar con él como si se estuviera frente a una máquina física, como se muestra en la siguiente imagen:

    ![alt text](/img/vnc1.png)


## Ejecutar Yosys desde el contenedor

Para ejecutar Yosys, el *software* de síntesis de lógica digital, dentro del contenedor de Docker, se deben seguir estos pasos:

1. Verificar si Yosys está instalado: En la terminal del contenedor ejecutar:

    ```
    yosys --version
    ```

2. Crear un *script* con extensión ```.ys``` para automatizar la carga de los respectivos archivos HDL del diseño y su  síntesis. La estrucutra del archivo debe ser la siguiente:

    ```
    # read design
    read_verilog -sv src/<archivo_1>.sv
    read_verilog -sv src/<archivo_2>.sv
    hierarchy -check -top <nombre_del_modulo_top>

    # the high-level stuff
    proc; opt; fsm; opt; memory; opt

    # mapping to internal cell library
    techmap; opt

    # mapping flip-flops to sky130_fd_sc_hd__tt_025C_1v80.lib
    dfflibmap -liberty /foss/pdks/sky130A/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

    # mapping logic to sky130_fd_sc_hd__tt_025C_1v80.lib
    abc -liberty /foss/pdks/sky130A/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib

    clean

    # write netlist file
    write_verilog <nombre_del_resultado>.v
    write_blif <nombre_del_resultado>.blif
    ```

    En donde:

    * ```read_verilog -sv```: Carga los archivos HDL. El flag ```-sv``` indica que el archivo contiene sintaxis de SystemVerilog. Los archivos pueden tener extensión sv o -svh y el flag seguirá siendo ```-sv```.

    * ```src/<archivo_n>```: Es la ruta de cada uno de las fuentes que componen el diseño. 

    * ```hierarchy -check -top <nombre_del_modulo_top>```: Este paso revisa la jerarquía del diseño y establece el módulo top, que es el punto de entrada al diseño.

    * ```proc; opt; fsm; opt; memory; opt```: Estos comandos realizan varias optimizaciones en el diseño:

        * ```proc```: Procedimiento de optimización de diseño.
        * ```opt```: Optimización general del diseño.
        * ```fsm```: Generación de máquinas de estado finitas.
        * ```memory```: Manejo y optimización de bloques de memoria.
        * ```opt```: Vuelve a optimizar el diseño después de realizar los pasos anteriores.

        Estas etapas ayudan a reducir el tamaño y la complejidad del diseño, preparando el netlist para su mapeo en celdas físicas.

    * ```techmap; opt```

        ```techmap``` mapea el diseño a una tecnología específica (en este caso, la tecnología de celdas estándar definida en las bibliotecas sky130). Esto incluye la sustitución de las celdas genéricas por celdas específicas de la biblioteca.

        El comando ```opt``` posterior realiza una optimización adicional del diseño, posiblemente reduciendo la cantidad de recursos o mejorando el rendimiento.

    * ```dfflibmap -liberty /foss/pdks/sky130A/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib```: Este comando mapea las celdas flip-flop del diseño a las celdas de la biblioteca ```sky130_fd_sc_hd``` usando un archivo de biblioteca de celdas ```sky130_fd_sc_hd__tt_025C_1v80.lib```. Estas celdas son específicas para la tecnología  sky130.

        Principales opciones que se podrán encontrar dentro del repositorio de SkyWater PDK:

        * ```sky130_fd_sc_hd```: Librería de celdas estándar de alto rendimiento (High Density Standard Cells). Es la más comúnmente utilizada en diseños generales de Sky130 para procesadores, memoria, y otros circuitos digitales.

        * ```sky130_fd_sc_hs```: Esta es una versión más optimizada de la librería HD pero para altas velocidades. Proporciona celdas diseñadas para maximizar la velocidad, sacrificando un poco de área.

        * ```sky130_fd_sc_lp```: Librería optimizada para diseños que necesitan bajo consumo de energía LP (Low Power). Utiliza celdas de bajo consumo que son ideales para aplicaciones portátiles o de consumo energético mínimo.

        * ```sky130_fd_sc_ms```: Librería optimizada para memorias y es adecuada para diseños que requieren alta densidad de celdas y gran capacidad de almacenamiento.

        En este caso se escogió ```sky130_fd_sc_hd``` y existen diferentes versiones de la librería **Liberty** que corresponden a distintas configuraciones de la tecnología, como diferentes condiciones de proceso, voltaje, temperatura, etc. Los archivos ```.lib``` son versiones específicas de la librería que se usan para sintetizar el diseño teniendo en cuenta un conjunto particular de parámetros de operación. En este caso particular se escogió ``sky130_fd_sc_hd__tt_025C_1v80.lib```, en donde los sufijos en los nombres de los archivos de la librería representan lo siguiente:

        * ```tt```: Tipo de proceso "typical" (condiciones nominales).
        * ```025C```: Temperatura de 25°C.
        * ```1v80```: Voltaje de 1.80V.

        Además de esta versión tt_025C_1v80.lib, pueden existir otras versiones como:

        * ```sky130_fd_sc_hd__ss_025C_1v80.lib```: Para un proceso ss (de baja velocidad).
        * ```sky130_fd_sc_hd__ff_025C_1v80.lib```: Para un proceso ff (de alta frecuencia).
        * ```sky130_fd_sc_hd__tt_125C_1v80.lib```: Para una temperatura de operación de 125°C.

    * ```abc -liberty /foss/pdks/sky130A/libs.ref/sky130_fd_sc_hd/lib/sky130_fd_sc_hd__tt_025C_1v80.lib```: El comando abc realiza el mapeo de la lógica combinacional del diseño a las celdas de la biblioteca de sky130. Aquí es donde se realiza el mapeo de la lógica del diseño a las celdas físicas de la biblioteca, convirtiendo las operaciones lógicas en celdas de compuertas.

    * Creación del Netlist: Se genera un archivo que contiene el diseño después de la síntesis, optimización y mapeo a celdas. Se pueden generar diferentes tipos de netlists:

        * ```write_verilog <nombre_del_resultado>.v```: Genera un archivo de Verilog. 

        * ```write_verilog <nombre_del_resultado>.v```: Genera  un archivo BLIF (Berkeley Logic Interchange Format).


3. Correr el *script* ```.ys```: En la terminal se debe estar ubicado en el directorio que contiene a la carpeta en donde están las fuentes HDL, en este caso la carpeta es ```src```, por lo que en la terminal donde se ejecturá el *script* se debe estar ubicado en el directorio que contiene a la carpeta ```src``` y se debe ejecutar el *script* usando el siguiente comando:

    ```
    yosys -s script.ys
    ```

    En donde:

    * ```yosys```: Es el comando para ejecutar Yosys.
    * ```-s script.ys```: Indica que Yosys debe cargar y ejecutar las instrucciones que están en el *script* ```.ys```.

    Esto generará un nuevo archivo ```.v``` y ```.blif``` como se ordenó en el *script*.

    Estos archivos contienen  una versión "plana" del diseño, donde los módulos originales se reemplazan por instancias de puertas lógicas básicas.

    Contiene:

   + Instancias de standard cells: Puertas lógicas, flip-flops, etc.
   + Estructura optimizada del circuito.

    ![alt text](/img/yosys_verilog.png)


## Referencias

1. https://github.com/iic-jku/iic-osic-tools
2. https://docs.docker.com/engine/install/ubuntu/
3. https://tigervnc.org/






    
