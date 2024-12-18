# Tutorial OpenLane

1. [Contenedor Docker](#instalar-el-contenedor-docker)
2. [Herrmientas e interfaz de trabajo](#herramientas-de-diseño-e-interfaz-de-trabajo)
3. [Ejecutar un flujo de diseño](#ejecutar-un-flujo-de-diseño)
4. [¿Cómo interactuar con archivos locales desde el contenedor?](#cómo-interactuar-con-archivos-locales-desde-el-contendor)
5. [TigerVNC](#tigervnc)
6. [Referencias](#referencias)

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

## Ejecutar un flujo de diseño:

Acceda a la terminal que se ve en la interfaz y ejecute los siguientes pasos:

1. En el contenedor, en la ubicación:

    ```
    cd /foss/tools/openlane/2023.08/designs/
    ```

    encontrará dos ejemplos llamados ```spm``` y ```ci```, copie alguna de estas dos carpetas a un directorio en donde se tengan permisos de edición, por ejemplo el directorio ```/headless```. Esto se puede realizar por consola o manualmente usando el gestor de archivos que se encuentra en la parte inferior izquierda del servidor VNC.
   


3. Entre a la carpeta copiada de alguno de los ejemplos, en el directorio seleccionado, por ejemplo :

    ```
    cd /headless/spm Usa el siguiente comando para ver los contenedores en ejecución:
    ```

    Allí encontrá los *scripts* HDL del ejemplo en la carpeta ```src``` y además encontrará dos *scripts*:
        
    * ```config.json```: Contiene configuraciones específicas para el flujo de diseño, como la ruta de los archivos Verilog, las restricciones de reloj, y las configuraciones del PDK, entre otros.
        
    * ```pin_order.cfg```: Se utiliza para definir la asignación de pines de I/O del diseño.

4. Para correr un flujo de OpenLane es necesario hacer uso del archivo ```flow.tcl``` que se encuentra en la carpeta de Openlane, para lo cual, una vez esté en el directorio donde se encuentra el archivo ```config.json```, deberá ejecutar el flujo de la siguiente forma:

```
/foss/tools/openlane/2023.08/flow.tcl
```
        
Esto creará una carpeta en el directorio del diseño, en este caso ```/headless/spm```, llamada ```runs``` donde se almacenan todos los resultados y archivos generados durante el flujo de diseño.

## ¿Cómo interactuar con archivos locales desde el contendor?

1. Pasos para evitar conflictos de puertos:

    1. Revisar los contenedores en ejecución:

        ```
        docker ps
        ```
    2. Detener el contenedor que esté utilizando el puerto conflictivo: Si el contenedor que ya está en ejecución está utilizando el puerto 5901 o 80, se debe detener ese contenedor con el comando:

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

    Si se configura correctamente, se abrirá una ventana del cliente VNC que muestra el escritorio del contenedor, y se podrá interactuar con él como si se estuviera frente a una máquina física, como se muestra en la siguiente imagen:

    ![alt text](/img/vnc1.png)




## Referencias

1. https://github.com/iic-jku/iic-osic-tools
2. https://docs.docker.com/engine/install/ubuntu/
3. https://tigervnc.org/






    
