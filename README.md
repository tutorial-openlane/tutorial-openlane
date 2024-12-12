# Tutorial OpenLane

1. [Contenedor Docker](#instalar-el-contenedor-docker)
2. [Herrmientas e interfaz de trabajo](#herramientas-de-diseño-e-interfaz-de-trabajo)
3. [Ejecutar un flujo de diseño](#ejecutar-un-flujo-de-diseño)
4. [Tiger VNC](#tiger-vnc)
5. [Referencias](#referencias)

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

    * El puerto 80 sea usado para la interfaz web (a través de noVNC).
    
    * El puerto 5901 será usado para el acceso al servidor VNC.

7. Ahora se puede acceder al entorno de escritorio a través de un navegador en: 
    
    ```
    http://localhost/?password=abc123
    ```

    Allí deberá encontar lo siguiente:

    ![alt text](/img/image-1.png)

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

## Tiger VNC


## Referencias

1. https://github.com/iic-jku/iic-osic-tools
2. https://docs.docker.com/engine/install/ubuntu/
3. https://tigervnc.org/






    
