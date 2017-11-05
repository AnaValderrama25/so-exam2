### Examen 2
**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Docente:** Daniel Barragán C.  
**Tema:** Namespaces, CGroups, LXC  
**Correo:** daniel.barragan at correo.icesi.edu.co  
  
  
**Estudiante:** Ana Valderrama V.  
**Código:** A00065868  
**Correo:** ana_fernanda_25@hotmail.com  




### Objetivos
* Comprender los fundamentos que dan origen a las tecnologías de contenedores virtuales
* Conocer y emplear funcionalidades del sistema operativo para asignar recursos a procesos
* Conocer y emplear capacidades de CentOS7 para la virtualización

### Prerrequisitos
* Virtualbox o WMWare
* Máquina virtual con sistema operativo CentOS7

### Descripción
El segundo parcial del curso sistemas operativos trata sobre el manejo de namespaces, cgroups y virtualización por medio de LXC/LXD

### Desarrollo Actividades

3. Realice una prueba de concepto empleando systemd y el recurso de control CPUQuota teniendo en cuenta los requerimientos que se describen a cotinuación. Incluya evidencias del funcionamiento de lo solicitado (30%):
 * Las pruebas se realizaran sobre un solo núcleo de la CPU
 * Se deben ejecutar dos procesos
 * Cada proceso debe poder acceder solo al 50% de la CPU
 * Cuando uno de los procesos se cancela, el que continua ejecutándose no debe acceder a mas del 50% de la CPU  
 Primero, en VirtualBox se configuró el sistema para que solo contara con un procesador.  
 ![][1]  
 Se inició sesión en Multiputty, y se comprobó con el comando ***lscpu*** que sólo se cuenta para la prueba con un procesador:  
 ![][2]  
 Luego, en el usuario operativos en el directorio de **scripts** se creó otro directorio con el nombre **parcial2** y en ese directorio con el editor se crearon dos scripts en bash: countHola.sh y countAdios.sh:  
 ![][3]  
 Y se le dieron permisos de ejecución:  
 ```  
 $ chmod +x countHola.sh  
 $ chmod +x countAdios.sh  
 ```  
 Luego, para convertir estos procesos en servicios se ejecutaron los siguientes comandos: 
 ```  
 #cd /etc/systemd/system  
 #vi countHola.service  
 #cat countHola.service  
 [Unit]  
 Description = making network connection up  
 [Service]  
 ExecStart = /home/operativos/scripts/parcial2/countHola.sh  
 CPUQuota = 50%  
 [Install]  
 WantedBy = default.target  
 #chmod 777 countHola.service  
 #vi countAdios.service  
 #cat countAdios.service  
 [Unit]  
 Description = making network connection up  
 [Service]  
 ExecStart = /home/operativos/scripts/parcial2/countAdios.sh  
 CPUQuota = 50%  
 [Install]  
 WantedBy = default.target  
 #chmod 777 countAdios.service   
 ```  
 Se puede observar que al convertir los scripts a servicios es necesario pasar la ruta del script *.sh* que va a ejecutar, y que además en **[Service]** es donde se restringe el uso de la CPU a un porcentaje específico (50%) a través del controlador ***CPUQuota***.   
 Luego, se habilitaron los servicios a través del comando *enable* de systemctl y se verificó su estado, en aún inactivos:  
 ![][4]  
 Luego de verificar el estado y hacer ***systemctl daemon-reload*** , se iniciaron los procesos con el comando *start* de systemctl:  
 ![][5]  
 Se verificó su estado:  
 ![][6]  
 Luego, con el comando ***top*** se observa cuanto % de CPU esta siendo utilizado por los procesos. No superior al 50%,  
 ![][7]  
 Para demostrar que aún con la cancelación de uno de los procesos, el otro que continua ejecutándose solo puede acceder al 50% de CPU, se mata el proceso con *PID* 8108 que corresponde al countAdios.sh, y se observa que countHola.sh emplea hasta el 49,5% pero no supera el 50%.  
 ![][8]   
 
4.  Realice una prueba de concepto empleando systemd y el recurso de control CPUShares teniendo en cuenta los requerimientos que se describen a continuación. Incluya evidencias del funcionamiento de lo solicitado (30%):
 * Las pruebas se realizaran sobre un solo núcleo de la CPU
 * Se deben ejecutar dos procesos
 * Uno de los procesos tendrá el 25% de la CPU mientras que el otro tendrá el 75% de la CPU
 * Cuando uno de los procesos se cancela, el que continua ejecutándose debe poder llegar al 100% de la CPU
5. Por medio de las evidencias obtenidas en los puntos anteriores y de fuentes de consulta en Internet, elabore las definiciones para los grupos de control CPUQuota y CPUShares, además concluya acerca de cuando es preferible usar un recurso de control sobre otro (20%)
6. El informe debe ser entregado en formato pdf a través del moodle y el informe en formato README.md debe ser subido a un repositorio de github. El repositorio de github debe ser un fork de https://github.com/ICESI-Training/so-exam2 y para la entrega deberá hacer un Pull Request (PR) respetando la estructura definida. El código fuente y la url de github deben incluirse en el informe (10%)  

### Referencias
https://github.com/ICESI/so-containers
