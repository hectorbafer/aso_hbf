# 📄 Proyecto de Módulo: Integración de Sistemas Heterogéneos

## 💼 Contexto empresarial
Trabajas para una empresa que históricamente ha trabajado solo con entornos Windows. Debido al aumento de costes en licencias, la dirección técnica ha decidido que el nuevo **Servidor de Almacenamiento** sea una máquina **Linux**, ya que es más eficiente y además, es gratuito.

Sin embargo, el Director de Seguridad ha impuesto un requisito innegociable: **"No quiero gestionar dos bases de datos de usuarios distintas. Los usuarios deben usar sus contraseñas actuales de Windows para entrar a las carpetas de Linux."**

Como el equipo técnico actual desconoce cómo conectar ambos mundos, se te ha encargado la investigación del procedimiento y la ejecución del despliegue.

## ✅ Objetivo
**1.** Desplegar una infraestructura de red básica con Active Directory.  
**2.** Documentar y analizar los métodos existentes para integrar un host Linux en un dominio Microsoft.  
**3.** Implementar la solución investigada para unir el servidor Linux al dominio.  
**4.** Configurar Samba para compartir recursos utilizando ACLs basadas en usuarios/grupos del dominio (no locales).  

## 📌 Fase A: Infraestructura base
Para el proyecto usaré las siguientes máquinas con dos adaptadores de red, una en **red interna** para que se comuniquen y la otra en **NAT**. Las versiones que he usado son:
- Windows Server 2025
- Ubuntu Server 24.04
- Windows 10

| Host          | IP             | Máscara de subred | Servidor DNS   |
| ------------- | -------------- | ----------------- | -------------- |
| `WServer-HBF` | 192.168.100.10 | 255.255.255.0 /24 | 127.0.0.1      |
| `server-hbf`  | 192.168.100.20 | 255.255.255.0 /24 | 192.168.100.10 |
| `W10-HBF`     | 192.168.100.30 | 255.255.255.0 /24 | 192.168.100.10 |

> 💬 No hará falta poner la puerta de enlace porque sería para que saliese hacia un router que en este caso no existe.

### Configuración de `Windows Server`
Pulsamos la combinación de teclas `Win+X` y escribimos `ncpa.cpl`, hacemos clic derecho en el adaptador de red y clicamos en `Propiedades`. Luego, clicamos en `Protocolo de Internet versión 4 (TCP/IPv4)` y pondremos lo siguiente:

![ipWS](Imagenes/ipWS.png)

Lo comprobamos poniendo el comando `ipconfig /all`.

![ipconfigWS](Imagenes/ipconfigWS.png)

Lo que haremos ahora por si acaso, será desactivar el **Firewall**. Desde el **Panel de control**, vamos hacia `Sistema y seguridad → Firewall de Windows Defender` y en la parte de la izquierda, clicamos en `Activar o desactivar el Firewall de Windows Defender`. Clicamos en `Desactivar Firewall de Windows Defender` tanto en `Redes privadas` como en `Redes públicas o invitadas`, aceptamos y ya está desactivado el **Firewall**.

![firewallWS](Imagenes/firewallWS.png)

> 💬 Para la práctica sí que podremos quitar el Firewall para evitar errores pero para un entorno real de empresa no se tendrá que desactivar.

### Configuración de `Ubuntu Server`
Editamos el archivo de configuración para poner la IP estática, pondremos este comando:
```bash
sudo nano /etc/netplan/50-cloud-init.yaml
```

![ipU](Imagenes/ipU.png)

Aplicamos los cambios con el comando `sudo netplan apply`. Para ver la IP pondremos `ifconfig`. Pero antes instalamos las `net-tools` con `sudo apt install net-tools`.

![ifconfigU](Imagenes/ifconfigU.png)

### Configuración de `Windows 10`
Hacemos la combinación de teclas `Win+X` y escribimos `ncpa.cpl`, hacemos clic derecho en el adaptador de red y clicamos en `Propiedades`. Luego, clicamos en `Protocolo de Internet versión 4 (TCP/IPv4)` y pondremos lo siguiente:

![ipW](Imagenes/ipW.png)

Lo comprobamos poniendo el comando `ipconfig /all`.

![ipconfigW](Imagenes/ipconfigW.png)

Desactivamos el **Firewall** de la misma manera que hicimos en nuestro **Windows Server**.

![firewallW](Imagenes/firewallW.png)

> 💬 Para la práctica sí que podremos quitar el Firewall para evitar errores pero para un entorno real de empresa no se tendrá que desactivar.

### Instalación de Active Directory en `Windows Server`
Para instalar **Active Directory** (o AD), vamos a la parte superior derecha, clicamos en `Administrar` y `Agregar roles y características`.

![agregarRoles](Imagenes/agregarRoles.png)

Para la instalación seguiremos los siguientes pasos:

![ad1](Imagenes/ad1.png)

Clicamos en el `checkbox` para reiniciar el servidor en caso necesario y clicamos en **Instalar**.

![ad2](Imagenes/ad2.png)

Cuando lo tengamos instalado, tendremos que crear el bosque. Nos saldrá un icono de peligro (⚠️) al lado de la bandera de notificaciones. 

![promover](Imagenes/promover.png)

Al clicar, nos saldrá una ventana para promover el controlador de dominio. Seguiremos los siguientes pasos para tener un nuevo bosque completamente limpio.

Escogemos la opción de agregar un nuevo bosque y le pondremos un nombre.

![controladorDominio1](Imagenes/controladorDominio1.png)

Lo dejamos como está y le pondremos una contraseña que nos sea fácil de recordar.

![controladorDominio2](Imagenes/controladorDominio2.png)

Clicamos directamente en **Siguiente**.

![controladorDominio3](Imagenes/controladorDominio3.png)

Este será el nombre que va a tener nuestro NetBIOS, es decir, lo que veremos al iniciar sesión antes del nombre de usuario.

![controladorDominio4](Imagenes/controladorDominio4.png)

Estas serán las rutas que tendrá el AD. Se recomienda no tocarlo a no se que estemos seguros de ello.

![controladorDominio5](Imagenes/controladorDominio5.png)

Esto de aquí, es un resumen de lo que hemos hecho, podemos clicar en `Ver script` para verlo más detalladamente.

![controladorDominio6](Imagenes/controladorDominio6.png)

![scriptAD](Imagenes/scriptAD.png)

Ahora tendremos que esperar un poco hasta que nos salga lo siguiente y podamos clicar en **Instalar**.

![controladorDominio7](Imagenes/controladorDominio7.png)

Al clicar en Instalar, llegará un punto en el que nuestra máquina se reiniciará para aplicar los cambios que hemos hecho. Y con esto ya tendríamos instalado nuestro **AD** en Windows Server.

Cuando se haya reiniciado, podremos ver que al iniciar sesión, podremos ver el dominio que hemos hecho.

![sesion](Imagenes/sesion.png)

Iniciamos sesión y ahora nos iremos hacia `Herramientras → Usuarios y equipos de Active Directory` y crearemos las **Unidades Organizativas** de `Ventas`, `IT` y `Gerencia`. Luego, creamos a los usuarios y grupos que se piden dentro de esas Unidades Organizativas.  
Creamos la Unidad Orgnizativa con clic derecho en nuestro dominio y vamos a `Nuevo → Unidad Organizativa`. Para los usuarios y grupos, crearé los siguientes:
- **Ventas:**
  - G_Ventas
  - UsuV1
  - UsuV2
- **IT:**
  - G_IT
  - UsuIT1
  - UsuIT2
- **Gerencia:**
  - G_Gerencia
  - UsuG1
  - UsuG2

Cuando tengamos a los usuarios creados y dentro del grupo asignado, nos quedará de la siguiente manera:

![usuariosV](Imagenes/usuariosV.png)

![usuariosIT](Imagenes/usuariosIT.png)

![usuariosG](Imagenes/usuariosG.png)


## 📌 Fase C: Implantación e interoperabilidad


---
### [⬅️ Volver a Proyecto de Módulo](../index.md)
---