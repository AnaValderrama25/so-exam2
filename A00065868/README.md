### Examen 2
**Universidad ICESI**  
**Curso:** Sistemas Operativos  
**Docente:** Daniel Barragán C.  
**Tema:** Namespaces, CGroups, LXC  
**Correo:** daniel.barragan at correo.icesi.edu.co  
  
  
**Estudiante:** Ana Valderrama V.  
**Código:** A00065868  
**Correo:** ana_fernanda_25@hotmail.com   
**GitUrl:** https://github.com/AnaValderrama25/so-exam2/



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
 
 Como se esta empleando la misma máquina virtual que en el punto anterior, como recurso solo se tiene un procesador en la CPU. Se reutilizaron los procesos creados también en el punto anterior, pero en el archivo en el que se convirtieron en servicios en **/etc/systemd/system**, en lugar de tener la restriccón de ***CPUQuota*** se tiene la restricción ***CPUShares***, para el proceso countHola se asigna:  
  ```  
  CPUShares = 7680  
  ```  
  Y para el proceso countAdios, se asigna:  
  ```  
  CPUShares = 2560  
  ```  
  ![][9]
  Según el ejemplo en la guía de systemd de so-processes, el total de procesos sumaba 10240, y para asignar el peso multiplicaba ese numero por el porcnetaje deseado, así *10240x0,75=7680* asigna un 75% al proceso ***countHola***, y *10240-7680=2560* asigna el 25% restante a el proceso ***countAdios***.  
  Desde el usuario root se activan los servicios digitando los comandos:  
  ```  
  #systemctl start countHola.service   
  #systemctl start countAdios.service   
  
  ```  
  Con el comando top se puede observar que uno de los procesos ocupa casi el 75% y el otro el 25% efectivamente:  
  ![][10]  
  Luego, se termina forzosamente el servicio countHola, gracias al comando ***kill*** y el  ***PID*** del proceso, se puede observar que el proceso que continua ejecutándose sobrepasa el 75% ocupando más del 95% de CPU, se estima que si es necesario podría llegar a 100%:  
  ![][11]  
  
5. Por medio de las evidencias obtenidas en los puntos anteriores y de fuentes de consulta en Internet, elabore las definiciones para los grupos de control CPUQuota y CPUShares, además concluya acerca de cuando es preferible usar un recurso de control sobre otro (20%) 
Se pueden agrupar los procesos mediante grupos de control dependiendo del tipo de recurso (CPU, Memory, IO) y las capacidades que este posea. 
### CPUQuota  
Asigna un porcentaje de tiempo de CPU a un proceso específico que va a ser ejecutado, toma el valor que este descrito en **[Service]** con el signo "%" como el límite máximo de tiempo que el proceso puede tener de CPU relativo al tiempo total disponible de una CPU. CPUQuota controla el atributo *cpu.max* en la jerarquía unificada del grupo de control.  

### CPUShares  
Es una opción que ya esta *deprecated*, en su reemplazo entro **CPUWeight**. **CPUShares** es un grupo de control de la CPU, viene por defecto con el valor 1024, y corresponde con el tiempo de CPU que se le puede asignar a un proceso. Si existen varios procesos ejecutándose al mismo tiempo, el porcentaje al que equivalga el valor que posee en **CPUShares** es lo que le dará en el tiempo de CPU. Por ejemplo, en el punto anterior el total de CPUShares entre los dos procesos era 10240, al ejecutarse los dos al tiempo, el countHola.sh que tenía un valor de 7680 le correspondería un 75% de tiempo de CPU como valor máximo, countAdios.sh tenía el 25% restante pues tenía un valor de 2560, es decir le correspondería el 25% de tiempo de CPU como máximo.  
  
Para concluir, CPUShares y CPUQuota son grupos de control que restringen el uso de un recurso (CPU) a un proceso específico. Cuál usar depende de la situación, por ejemplo CPUShares es útil cuando se tienen varios procesos y deben ser gestionados de forma eficiente, si existe alguno que merece prioritariamente más tiempo de CPU que otro se le asignará un peso mayor al de los otros, en caso de solo existir un proceso en ejecución éste podrá emplear el 100% de tiempo de CPU de ser necesario. Por otro lado, CPUQuota puede ser útil en un caso en el que un proceso emplea demasiado tiempo de CPU y agota el recurso, con CPUQuota se le asigna un límite que no podrá superar aun incluso si es el único proceso en ejecución.   


### Referencias
https://github.com/ICESI/so-containers  
https://github.com/ICESI/so-processes  
https://unix.stackexchange.com/questions/218074/how-to-know-number-of-cores-of-a-system-in-linux  
https://www.freedesktop.org/software/systemd/man/systemd.resource-control.html  
https://docs.docker.com/engine/admin/resource_constraints/#configure-the-default-cfs-scheduler  
https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/resource_management_guide/sec-modifying_control_groups  
https://cloud.google.com/compute/quotas  
https://peteris.rocks/blog/can-you-kill-it/  
https://github.com/systemd/systemd/issues/5820  




[1]: images/Procesador1.PNG  
[2]: images/Procesador2.PNG  
[3]: images/Processes.PNG  
[4]: images/AuthenticationStatus.PNG  
[5]: images/StartProcesses.PNG  
[6]: images/ActiveProcesses.PNG  
[7]: images/CPUusage.PNG  
[8]: images/KillCountAdios.PNG  
[9]: images/CPUShares.PNG  
[10]: images/CPUShares2.PNG  
[11]: images/CPUShares3.PNG  
