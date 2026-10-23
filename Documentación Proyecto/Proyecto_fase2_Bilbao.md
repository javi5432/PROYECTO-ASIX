# IES Enric Soler i Godes
**Curs 2026-2027**  
**Grupo Bilbao**

---

## P2 Instal·lació del servidor Proxmox + Estudi IPs i solució switching laboratori
### Proyecto intermodular

**PROXMOX**

---

## Sumario

| Sección | Página |
| :--- | :--- |
| **1. Introducción y Objetivos.** | 3 |

---

## 1. Introducción y Objetivos

El objetivo de esta práctica es el despliegue de un servidor de virtualización Proxmox VE utilizando un medio de instalación basado en USB Ventoy, la conexión física y lógica a un switch Cisco mediante un cable de consola USB-a-RJ45 desde un entorno Linux Mint, y la posterior configuración de VLANs y trunks para segmentar la red por departamentos (Servidores, TIC, I+D y Gestión).

---

## 2. Preparación del Servidor y Despliegue de Proxmox

### 2.1. Creación del USB Booteable con Ventoy
Usamos un Live USB con Ventoy, el cual permite copiar la ISO de Proxmox directamente en el Servidor.

Se conectó el USB al servidor físico, se seleccionó la unidad desde el menú de arranque (Boot Menu) y se arrancó el instalador de Proxmox.

### 2.2. Proceso de Instalación Paso a Paso
El proceso de instalación guiada se completó aplicando los siguientes parámetros específicos de configuración:

- **Particionado de Disco:** Automático
- **Usuario de acceso:** `root`
- **Contraseña:** `bilbaoasix`
- **Interfaz de Red:** Tarjeta de red física principal del servidor
- **Hostname:** `proxmox-server.local`
- **Dirección IP:** `10.4.10.254`
- **Máscara de Red:** `255.255.255.0/24`
- **Puerta de enlace:** `10.4.10.1`

**Pasos ejecutados durante la instalación:**

1. **Selección del disco de destino:** Se seleccionó el disco duro local y se aplicó la partición automática recomendada por el instalador para la creación de volúmenes LVM.
2. **Configuración del sistema y credenciales:** Se definieron la ubicación horaria y el usuario superadministrador (`root`) asociado a la clave `bilbaoasix`.
3. **Asignación de la red estática:** Se estableció la dirección IP estática `10.4.10.254`, perteneciente al rango de la VLAN 410 (Servidores), garantizando la disponibilidad del servidor tras reiniciar el equipo.
4. **Finalización:** Completado el volcado de paquetes, se retiró la unidad USB y se reinició el servidor.

---

## 3. Acceso por Consola al Switch Cisco

Establecemos conexión por consola al switch Cisco mediante un cable. Como en el primer intento el sistema operativo no detectó el cable, fue necesario ejecutar los siguientes comandos:

```bash
sudo modprobe usbserial
sudo modprobe ftdi_sio
```

Se ha verificado que el sistema reconoce el puerto mediante el comando `lsusb` estando en la ruta `/dev/ttyUSB0`.

### 3.1. Acceso mediante PuTTY
Usamos el programa PuTTY configurando la conexión tipo Serial, apuntando al puerto `/dev/ttyUSB0` a una velocidad de 9600 baudios. Tras un reinicio, se abrió la consola del switch.

---

## 4. Configuración de VLANs y Puertos en el Switch

### 4.1. Creación y Definición de las VLANs
Accedemos al modo de configuración global (`configure terminal`) para dar de alta las VLANs corporativas:

```text
Switch3560 enable
Switch3560#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch3560(config)#^Z
Switch3560#configure terminal
*Mar 7 18:23:00.625: XSYS-5-CONFIG_I: Configured from console by console
Switch3560#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch3560(config)#vlan 410
Switch3560(config-vlan)#name SERVIDORES BILBAO
Switch3560(config-vlan)#exit
Switch3560(config-vlan)#name TIC_BILBAO
Switch3560(config)#vlan 420
Switch3560(config-vlan)#exit
Switch3560(config)#vlan 430
Switch3560(config-vlan)#name ID BILBAO
Switch3560(config-vlan)#exit
Switch3560(config)#vlan 490
Switch3560(config-vlan)#name GESTION BILBAO
Switch3560(config-vlan)#exit
Switch3560(config)#end
Switch3560#show
*Mar 7 18:25:06.706: SYS-5-CONFIG_I: Configured from console by con
Switch3560#show vlan brief
```

**Salida `/dev/ttyUSB0-PuTTY`:**

```text
101 RI Creators          active
102 RI Scada             active
103 RI SGE               active
104 RI Iluminacion       active
105 RI Codificacion      active
106 RI Depuradora        active
110 Servidors            active    Gi0/1
120 Comercial            active    Gi0/2
130 Administracio        active    Gi0/3
-More-
*Mar 7 18:25:42.080: SYS-5-CONFIG_I: Configured from console by vty0 (10140 PRACTIO)
141 VLAN0141             active
142 VLAN0142             active
143 VLAN0143             active
144 VLAN0144             active
145 VLAN0145             active
146 VLAN0146             active
147 VLAN0147             active
148 VLAN0148             active
149 VLAN0149             active
170 Seg_Int Macrolan     active
180 VLAN0180             active
199 Gestio               active    Gi0/4
210 ServidoresMad        active
220 ComercialMad         active
230 ProduccioMad         active
310 SERVIDOR-Sevilla     active
320 RRHH-Sevilla         active
330 DIRECCION-Sevilla    active
399 GESTION-Sevilla      active
410 SERVIDORES BILBAO    active
420 TIC BILBAO           active    Gi0/45
430 ID BILBAO            active
490 GESTION BILBAO       active
900 TRANSIT 2921         active
```

---

### 4.2. Configuración del Puerto Troncal para Proxmox

```text
Switch3560#configure terminal
/dev/ttyUSB0-PuTTY
Enter configuration commands, one per line. End with CNTL
Switch3560(config)#interface GigabitEthernet0/1
Switch3560(config-if)#description TRUNK PROXMOX
Switch3560(config-if)#switchport trunk encapsulation dot1q
Switch3560(config-if)#switchport mode trunk
Switch3560(config-if)#switchport trunk native
Mar 7 18:28:42.477: LINK-3-UPDOWN: Interface GigabitEthernet0/21, changed state to up vlan
*Mar 7 18:28:42.486: XLINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/21, change
Incomplete command.
Switch3560(config-if)#switchport trunk native vlan
*Mar 7 18:28:54.498: XLINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/21, change
Mar 7 18:28:56.495: %LINK-3-UPDOWN: Interface GigabitEthernet0/21, changed state to down
Switch3560#
*Mar 7 18:29:05.403: %SYS-5-CONFIG_I: Configured from console by console
Switch3560#
Switch3560#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Switch3560(config)#interface GigabitEthernet0/1
Switch3560(config-if)#description TRUNK
Mar 7 18:29:34.092: ZLINK-3-UPDOWN: Interface GigabitEthernet0/21, changed state to up
Mar 7 18:29:34.101: %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/21, chang
Switch3560(config-if)#
Switch3560(config-if)#description TRUNK PROXMOX
Switch3560(config-if)#switchport trunk encapsulation dot1q
Switch3560(config-if)#switchport mode trunk
Switch3560(config-if)#switchport trunk native vlan 490
Switch3560(config-if)#switchport trunk allowed vlan 410,420,430,490
Switch3560(config-if)#switchport nonegotiate
Switch3560(config-if)#no shutdown
Switch3560(config-if)#exit
Switch3560(config)#end
Switch3560#
Mar 7 18:32:08.367: SYS-5-CONFIG_I: Configured from console by console

Switch3560(config)#interface range GigabitEthernet 0/2 - 8
Switch3560(config-if-range)#description EQUIPOS_TIC
Switch3560(config-if-range)#switchport mode access
Switch3560(config-if-range)#switchport access vlan 420
Switch3560(config-if-range)#spanning-tree portfast
Warning: portfast should only be enabled on ports connected to a single
host. Connecting hubs, concentrators, switches, bridges, etc... to this
interface when portfast is enabled, can cause temporary bridging loops.
Use with CAUTION

Portfast will be configured in 7 interfaces due to the range command
but will only have effect when the interfaces are in a non-trunking mode.
Switch3560(config-if-range)#no shutdown
Switch3560(config-if-range)#exit
Switch3560(config)#end

Switch3560(config)#interface range GigabitEthernet0/9 - 12
Switch3560(config-if-range)#description EQUIPOS_ID_BILBAO
Switch3560(config-if-range)#switchport mode access
Switch3560(config-if-range)#switchport access vlan 430
Switch3560(config-if-range)#spanning-tree portfast
```

---

## 5. Guardado de la Configuración y Verificación Final

Para asegurar que todos los cambios se aplican en la configuración de ejecución (`running-config`) se ejecuta el comando `do write`.