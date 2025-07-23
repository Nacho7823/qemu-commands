# Resumen de uso de QEMU

QEMU es un emulador y virtualizador de hardware que permite ejecutar sistemas operativos y programas en diferentes arquitecturas.

## Instalación

En sistemas basados en Debian/Ubuntu:
```bash
sudo apt-get install qemu qemu-system
```

## CPU

- Asignar núcleos de CPU:
  ```bash
  qemu-system-x86_64 -smp 4
  ```
  - `-smp 4`: 4 núcleos de CPU.
- Usar aceleración KVM:
  ```bash
  qemu-system-x86_64 -enable-kvm
  ```
  - `-enable-kvm`: usa KVM si está disponible (virtualización por hardware).
- Usar emulación TCG multi-thread:
  ```bash
  qemu-system-x86_64 -accel tcg,thread=multi
  ```
  - `-accel tcg,thread=multi`: emulación por software multi-hilo.
- Usar CPU del host:
  ```bash
  qemu-system-x86_64 --cpu host
  ```
  - `--cpu host`: expone las características de la CPU física al huésped.

## RAM

- Asignar memoria RAM:
  ```bash
  qemu-system-x86_64 -m 4096
  ```
  - `-m 4096`: asigna 4 GB de RAM a la máquina virtual.

## Disco

### Crear una imagen qcow2:
  ```bash
  qemu-img create -f qcow2 disco.qcow2 20G
  ```
  - `-f qcow2`: formato qcow2.
  - `disco.qcow2`: nombre del archivo.
  - `20G`: tamaño de la imagen.

- Crear una imagen raw:
  ```bash
  qemu-img create -f raw disco.img 10G
  ```
  - `-f raw`: formato img (raw).
  - `disco.img`: nombre del archivo.
  - `10G`: tamaño de la imagen.

### Ejecutar una imagen de disco:
  ```bash
  qemu-system-x86_64 -hda disco.img -m 1024
  ```
  - `-hda disco.img`: imagen de disco duro.

- Añadir discos adicionales:
  ```bash
  qemu-system-x86_64 -hda disco.img -hdb datos.img
  ```
  - `-hdb datos.img`: segundo disco.

## Red

### NAT (red básica, salida a Internet):
  ```bash
  qemu-system-x86_64 -net nic -net user
  ```
  - `-net user`: modo NAT, no requiere configuración extra.

### Bridge (puente, conecta la VM a la red física):
  Es un switch virtual

  #### Crear un bridge en Linux:
  1. Crear el bridge:
     ```bash
     sudo ip link add name br0 type bridge
     sudo ip link set br0 up
     ```
  2. Agregar una interfaz física al bridge (ejemplo con eth0):
     ```bash
     sudo ip link set eth0 master br0
     ```
  3. Asignar una dirección IP al bridge (opcional):
     ```bash
     sudo ip addr add 192.168.1.1/24 dev br0
     ```

  #### Conectar una VM a un bridge:
  ```bash
  qemu-system-x86_64 -net nic -net bridge,br=br0
  ```
  - `br0`: interfaz de bridge en el host.

### TAP (interfaz de red virtual, requiere configuración previa):

  Una interfaz TAP simula una interfaz Ethernet a nivel de capa 2 (tramas). Es usada con qemu para conectar vms a una red real o virtual. Se diferencia de TUN, que trabaja en la capa 3 (paquetes IP).

  #### Cómo crear una interfaz TAP en Linux
  Para crear una interfaz TAP manualmente:
  ```bash
  sudo ip tuntap add dev tap0 mode tap
  sudo ip link set tap0 up
  ```

  - El primer comando crea la interfaz virtual `tap0` en modo TAP.
  - El segundo comando activa la interfaz para que pueda ser usada por QEMU u otras aplicaciones.
  
  #### Usar TAP con QEMU
  - Conectar una VM a una interfaz TAP:

  ```bash
  qemu-system-x86_64 -net nic -net tap,ifname=tap0,script=no
  ```
  - `tap0`: interfaz TAP creada en el host.
  - `script=no`: evita ejecutar scripts automáticos.



  #### Reenvío entre eth0 y tap (bridge):
  1. Crear interfaz tap:
     ```bash
     sudo ip tuntap add dev tap0 mode tap
     sudo ip link set tap0 up
     ```
  2. Crear bridge y agregar eth0 y tap0:
     ```bash
     sudo ip link add name br0 type bridge
     sudo ip link set eth0 master br0
     sudo ip link set tap0 master br0
     sudo ip link set br0 up
     ```
  3. Ejecutar QEMU usando el bridge:
     ```bash
     qemu-system-x86_64 -net nic -net tap,ifname=tap0,script=no -net bridge,br=br0
     ```
  Esto conecta la VM a la red física a través de eth0 y tap0 usando el bridge br0.

### Otros modos:
  - `-net none`: sin red.
  - `-net socket`: conecta dos VMs por socket.

## GPU (Gráficos)

- Usar aceleración gráfica (OpenGL):
  ```bash
  qemu-system-x86_64 -vga virtio -display sdl,gl=on
  ```
  - `-vga virtio | -vga virtio-vga-gl`: adaptador gráfico moderno.
  - `-display sdl,gl=on`: habilita OpenGL.
  
  Agregar soporte para venus
  - `-device virtio-vga-gl,hostmem=4G,blob=true,venus=true`

## USB

- Habilitar controlador USB:
  ```bash
  qemu-system-x86_64 -device usb-ehci -device usb-tablet
  ```
  - `-device usb-ehci`: controlador USB 2.0.
  - `-device usb-tablet`: dispositivo de entrada USB.

- Pasar un dispositivo USB físico (ejemplo con vendorid y productid):
  ```bash
  qemu-system-x86_64 -device usb-host,vendorid=0xXXXX,productid=0xYYYY
  ```
  - Reemplaza `0xXXXX` y `0xYYYY` por los IDs del dispositivo USB.

- Listar dispositivos USB conectados:
  ```bash
  lsusb
  ```
  - Usar los IDs obtenidos para el parámetro anterior.

## ISO

- Ejecutar una imagen ISO:
  ```bash
  qemu-system-x86_64 -cdrom archivo.iso -boot d -m 2048
  ```
  - `-cdrom archivo.iso`: especifica la imagen ISO.
  - `-boot d`: arranca desde el CD.
