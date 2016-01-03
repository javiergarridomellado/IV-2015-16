# Tema 5
## Rafael Lachica Garrido

### Ejercicio 1: Instalar los paquetes necesarios para usar KVM.
[fuente](http://nyacomputing.com/how-to-install-kvm-on-ubuntu-and-run-virtual-machines/).
Instalamos primero cpu-checker:
```
$ sudo apt-get install cpu-checker
[sudo] password for rafaellg8:
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias       
Leyendo la información de estado... Hecho
Los paquetes indicados a continuación se instalaron de forma automática y ya no son necesarios.
  akonadi-backend-mysql akonadi-server consolekit dirmngr icoutils kate-data
```

A continuación, como ha hicimos en el tema 1,comprobamos que está instalado el módulo de KVM, a través del comando **sudo kvm-ok**:
```
rafaellg8@system32:~/Documentos/GII/Cuarto/IV/IV-2015-16$ sudo kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used
```
![kvm-ok](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/kmv-ok_zpsykwyuztd.png?t=1443563958 "km-ok")


## Ejercicio 2
### Ejercicio 2.1: Crear varias máquinas virtuales con algún sistema operativo libre tal como Linux o BSD. Si se quieren distribuciones que ocupen poco espacio con el objetivo principalmente de hacer pruebas se puede usar CoreOS, GALPon Minino, Damn Small Linux, SliTaz y ttylinux.

Dado que KVM es un módulo del kernel, puede que no esté cargado por defecto. Dependiendo del procesador que usemos, lo cargamos con:
```
sudo modprobe kvm-intel
```
Con esto, si no esta ya activado, activamos KVM. En mi caso como mi cpu es Intel Core i3, he puesto intel.

Ahora descargamos y creamos la máquina virtual de [Minino](http://minino.galpon.org/es/descargas),he usado la Alguadaira, ya que es la que menos especacio ocupa.

Una vez descargada, creamos e instalamos la imagen en un disco duro virtual, para ello:
1. Creamos la imagen a través del comando: ```qemu-img create -f qcow2 imagen-cow.qcow2 250M```
![img-Quemu](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-02%20131941_zpsgaj4d7yl.png)

2. Ahora instalamos la imagen a través del comando **qemu-system-x86_64 -hda imagen-cow.qcow2 -cdrom ~/Descargas/slitaz-4.0.iso**.
Lo más probable, es que necesitamos instalar antes qemu, a través de ```sudo apt-get install qemu-system```

![instalacion](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-02%20135422_zpskbjehz3e.png)
![funcionando](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-02%20140125_zpse0hy8m8q.png)

Como es una versión live, cada vez que queramos ejecutar, tenemos que arrancar a través de **qemu-system-x86_64 -hda imagen-cow.qcow2 -cdrom ~/Descargas/slitaz-4.0.iso**

### 2.2: Hacer un ejercicio equivalente usando otro hipervisor como Xen, VirtualBox o Parallels.
Para este ejercicio usaré [VMWare](https://my.vmware.com/web/vmware/free#desktop_end_user_computing/vmware_workstation_player/12_0) y el sitema operativo [Ubuntu server](http://www.ubuntu.com/download/server)
![imagen con el sistema vwmare](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/0cf3bd8e-747f-460b-ae0f-029f785e679e_zpso7xliqdi.png)

## Ejercicio 3 Crear un benchmark de velocidad de entrada salida y comprobar la diferencia entre usar paravirtualización y arrancar la máquina virtual simplemente con qemu-system-x86_64 -hda /media/Backup/Isos/discovirtual.img.
Benchmark:
```
#!/usr/bin/env python
# -*- coding: utf-8 -*-
# Rafael Lachica Garrido
import sys
import random,math
import time

"""
benchmark
Función básica que consta de 3 bucles, y lo único que hace es leer un archivo en un bucle, y en los sucesivos anidados hallar el logaritmo
de un número aleatorio
"""
def benchmark():
    print ("Benchmar fácil, crea una matriz y la guarda en un fichero")
    print ("Creado por : Rafael Lachica Garrido")
    start = time.time()
    try:
        file = open("prueba")
        out = open("salida",'w')
    except:
        print ("Error fichero inexistente o imposible leer")
        print ("Probamos a crearlo")
        file = generateFile()
        out = open("salida",'w')

    i = j =     0
    d = {}
    for i in range(10):
        f = file.read()
        for j in range(len(f)):
            for x in range(1, 50):
                d[x] = math.log(x)
            out.write(str(d[x]))

    end = time.time() - start
    out.write("Time"+str(end))
    out.close()
    file.close()

    print ("Tiempo en segundos")
    print (end)

#Generamos el archivo
def generateFile():
    file = open("prueba",'w')
    for i in range(100000):
        file.write(str(random.random()*random.randint(1,10)))

    file.close()
    file=open("prueba",'r')

    return file

def main():
    if len(sys.argv) != 2:
        benchmark()
    else:
        generateFile()

if __name__ == "__main__":
	main()
```

Lo ejecutamos en local, y el resultado es de 24 segundos:
![imagen](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-02%20192522_zpskggwm0hy.png)

Ejecutamos el benchmark en la máquina virutal de ubuntu server:
![imagen](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/adca8d32-e0fc-418b-9a86-4216350b49d8_zpsdvbkvexn.png)

Para arrancar el sistema de forma normal, hacemos:
```
qemu-system-x86_64 -hda imagen-cow.qcow2 -cdrom ~/Descargas/slitaz-4.0.iso
```
Usamos la opción que dice core base
Y nos dice Stilaz boot time 80 s:
![imagen](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20134641_zpsjrhpvgon.png)

Ahora probamos con la paravirtualización:
```
rafaellg8@system32:~/Desktop/pruebasIV$ qemu-system-x86_64 -boot order=c -drive file=imagen-cow.qcow2,if=virtio
```

## Ejercicio 4: Crear una máquina virtual Linux con 512 megas de RAM y entorno gráfico LXDE a la que se pueda acceder mediante VNC y ssh.
Uno de los sistemas operativos que he probado y que es de los más ligeros es Lubuntu, basado en LXDE. Lo descargamos de aquí:
- [Lubuntu](https://help.ubuntu.com/community/Lubuntu/GetLubuntu)
Hacemos los mismos pasos que antes:
- Añadimos espacio virutal: ```qemu-img create -f qcow2 lubuntu.img 10G```
- Instalamos lubuntu, con la opción de 512 MB, con -m : ```qemu-system-x86_64 -hda lubuntu.img -cdrom lubuntu-15.10-desktop-amd64.iso -m 512M```
![image](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/lubu_zpskf0cmhlm.png)
![imagenInstalando](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20142304_zpsil0mzmsc.png)

## Ejercicio 5: Crear una máquina virtual ubuntu en Azure e instalar en ella un servidor nginx para poder acceder mediante web.
Accedemos al [portal de azure](https://portal.azure.com/) o al portal clásico que para mi es más cómodo [Clásico](https://manage.windowsazure.com)
seleccionamos Máquina Virtual -> Clásica:
![imagen](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/8e886c20-99ed-4f8d-9408-b9ac8782bb6e_zpsp8eby4tp.png)
![imagenCreada](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/36e5a0c3-3a83-44f2-a26b-dfd29e45980d_zpscafptotc.png)
Nombre del servidor será **prueba-iv-rlg**.

A contuación, debemos crear los llamados extremos o ENDPOINTS, para cada puerto, necesitamos al menos HTTP y SSH.
Para ello accedemos al [portal antiguo](https://manage.windowsazure.com/) el cual es mucho más cómodo.
Añadimos algunos extremos:
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20152020_zpsmic8ph65.png)

Configuración, con direcciones IP y extremos añadidos:
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/36e5a0c3-3a83-44f2-a26b-dfd29e45980d_zpscafptotc.png)
Ya tenemos más o menos toda la configuración, ahora necesitamos conectarnos por terminal a la máquina virtual.
Tenemos 2 formas, o bien por ssh, la cual va bastante lento, debido al retardo, o por la interfaz de comandos que nos ofrece un cliente de Azure para nmp. Vamos a probar esta opción, anque después instalaremos por ssh: [Fuente](https://azure.microsoft.com/en-us/documentation/articles/xplat-cli-install/)

1. Instalamos el cliente a través de npm:
```
rafaellg8@system32:~/Desktop/pruebasIV$ sudo apt-get install nodejs-legacy
Leyendo lista de paquetes... Hecho
Creando árbol de dependencias       
Leyendo la información de estado... Hecho
nodejs-legacy ya está en su versión más reciente.
0 actualizados, 0 se instalarán, 0 para eliminar y 12 no actualizados.
rafaellg8@system32:~/Desktop/pruebasIV$ sudo npm install -g azure-cli
```

2. Descargamos nuestra cuenta de azure para poder conectarnos:
```
rafaellg8@system32:~/Desktop/pruebasIV$ azure account download
info:    Executing command account download
info:    Launching browser to http://go.microsoft.com/fwlink/?LinkId=254432
help:    Save the downloaded file, then execute the command
help:      account import <file>
info:    account download command OK
rafaellg8@system32:~/Desktop/pruebasIV$

```

3. Nos logueamos a través de **azure login**:
```
rafaellg8@system32:~/Desktop/pruebasIV$ azure login
info:    Executing command login
-info:    To sign in, use a web browser to open the page https://aka.ms/devicelogin. Enter the code DKYYNDULZ to authenticate. If you're signing in as an Azure AD application, use the --username and --password parameters.
```
Introducimos el código para conectar el dispositivo.
Nos da lo siguiente:
```
/-
|info:    Added subscription Azure Pass
info:    Setting subscription "Azure Pass" as default
+
info:    login command OK
```

4. Listamos las máquinas virutales:
```
info:    Executing command vm list
+ Getting virtual machines                                                     
data:    Name           Status     Location          DNS Name                    IP Address     
data:    -------------  ---------  ----------------  --------------------------  ---------------
data:    pluco          ReadyRole  Central US        pluco.cloudapp.net          10.143.14.66   
data:    prueba-iv-rlg  ReadyRole  South Central US  prueba-iv-rlg.cloudapp.net  100.113.234.182
info:    vm list command OK
rafaellg8@system32:~/Desktop/pruebasIV$
```

Ahora ya tenemos los datos, al final necesitaremos acceder por ssh.

### Instalación Nginx
- Ahora nos conectamos por ssh:
```
ssh azureuser@prueba-iv-rlg.cloudapp.net
```

- Instalamos nginx:
```
root@prueba-iv-rlg:/home/azureuser# apt-get install nginx
```

- Probamos ahora que funciona:
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20162920_zpsqj4rejhc.png)

## Ejercicio 6: Usar juju para hacer el ejercicio anterior.

Juju, es un software libre, para la orquestación de servicios y herramientos de ubuntu que ayudan a desplegar, gestionar y escalar entornos o cualquier nube.
Para su instalación seguimos el [tutorial](https://jujucharms.com/docs/stable/getting-started).
```
sudo apt-get install juju-core
```
Como en local me tira el siguiente error al iniciar, ya que parece ser que no soporta mi distribución [Elementary OS Freya](https://elementary.io/es/), lo he tenido que instalar en una máquina virtual de ubuntuserver.
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/a04b6464-166b-4edb-a37e-4c499d5be9ee_zpsart01yst.png)

Ya tenemos instalado juju, ahora lo iniciamos:
```
rafaellg8:~$ juju
Juju -- devops distilled
https://juju.ubuntu.com/

Juju provides easy, intelligent service orchestration on top of environments
such as Amazon EC2, HP Cloud, OpenStack, MaaS, or your own local machine.

Basic commands:
  juju init             generate boilerplate configuration for juju environments
  juju bootstrap        start up an environment from scratch

  juju deploy           deploy a new service
  juju add-relation     add a relation between two services
  juju expose           expose a service

  juju help bootstrap   more help on e.g. bootstrap command
  juju help commands    list all commands
  juju help glossary    glossary of terms
  juju help topics      list all help topics

Provider information:
  juju help azure-provider       use on Windows Azure
  juju help ec2-provider         use on Amazon EC2
  juju help hpcloud-provider     use on HP Cloud
  juju help local-provider       use on this computer
  juju help openstack-provider   use on OpenStack
```
El comando juju nos muestra la ayuda, lo iniciamos entonces:
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20173248_zpswn5cqha2.png)

Nos informa de que se ha creado un archivo de configuración y que lo editemos para nuestro entorono o distro, y ejecutemos run bootstrap.
En esta web, he encontrado un [tutorial](https://jujucharms.com/docs/1.24/config-azure) que usaré para configurar el archivo para la máquina virtual de Ubuntu en Azure.

Editamos el archivo con los datos de nuestra suscripción que podemos encontrar en nuestra web de azure. También tenemos que crear un espacio de almacenamiento nuevo, yo he creado uno, "jujurlg":
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/4c6f261a-aaa1-44c7-b735-df300c0bc3e2_zps7sznvpwu.png)
También podemos encontrar los datos con el comando de azure account:
```
rafaellg8@system32:~/Desktop/varios$ azure account show
info:    Executing command account show
data:    Name                        : Azure Pass
data:    ID                          : 2cc2475d-2e3d-4d07-b873-e46b595373f7
data:    State                       : Enabled
data:    Tenant ID                   : 0b2c9607-9dd6-4fd2-88f2-268d3df2448c
data:    Is Default                  : true
data:    Environment                 : AzureCloud
data:    Has Certificate             : No
data:    Has Access Token            : Yes
data:    User name                   : falin1993@hotmail.com
data:    
info:    account show command OK
```
Nos dice que tenemos que generar un nuevo certificado para usar Juju.
- Generamos el certificado
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20173740_zpseheele9m.png)
Nos devuelve un archivo **azure.pem**, con el cual generamos el archivo de certificado .cer:
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/c9a009a6-00a3-4f03-8d59-a19beedc92ea_zpsdlralabf.png)

Me he pasado los certificados a una carpeta de github para poder descargarlos luego y trabajar con ellos en Azure, [carpeta github](https://github.com/rafaellg8/varios)

Ahora con este archivo nos tenemos que ir a nuestra [cuenta de Azure](https://manage.windowsazure.com), y añadirlo con los campos que nos dice en el tutorial.

Pinchamos en Configuración --> Gestión de certificado y lo añadimos:
![img](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20170331_zps0kc2snqu.png)

Ahora cambiamos de entorno a azure:
![juju switch](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/491646b6-9237-4ec4-b4b7-4e1621e63ee1_zps9kdtsjjd.png)


## Ejercicio 7: Instalar una máquina virtual con Linux Mint para el hipervisor que tengas instalado.
Seguimos el siguiente [tutorial](http://community.linuxmint.com/tutorial/view/1727) de Linux Mint para su instalación en kvm.
Instalamos:
```
sudo apt-get install qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils virt-manager
```

Ahora creamos otro espacio virtual para Linux Mint:
```
dd if=/dev/zero of=LM_test_os.image bs=1M count=10240
```
Nos da:
```
rafaellg8@system32:~/Desktop/pruebasIV$ dd if=/dev/zero of=LM_test_os.image bs=1M count=10240
10240+0 registros leídos
10240+0 registros escritos
10737418240 bytes (11 GB) copiados, 184,053 s, 58,3 MB/s
```

Ahora instalamos con qmu:
```
rafaellg8@system32:~/Desktop/pruebasIV$ kvm -cdrom linuxmint-17.3-cinnamon-64bit.iso -m 1g -hda LM_test_os.image
```
![imagen](http://i1383.photobucket.com/albums/ah302/Rafael_Lachica_Garrido/Captura%20de%20pantalla%20de%202016-01-03%20184858_zps5cra8ikk.png)

Lo configuramos y le damos a instalar, y ya tenemos nuestro Mint en Qemu.