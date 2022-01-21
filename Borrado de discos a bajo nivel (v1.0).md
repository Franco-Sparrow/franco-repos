# INDEX

1. Borrado de los discos.

# 1. Borrado de los discos

Suponiendo que los tres discos nuevos insertados en el servidor son reconocidos como:
- `/dev/sdc`
- `/dev/sdd`
- `/dev/sde`

Para estar seguros de que los discos estan completamente borrados, se debe eliminar cualquier rastro de metadatos o informacion o particion en el disco. Antes de hacer el `dd` se debe hayar el valor optimo para el tamaño de bloque `bs` (Por defecto, 512 Bytes):
```bash
stat -c "%o" /dev/sdc && \
stat -c "%o" /dev/sdd && \
stat -c "%o" /dev/sde
```

In this case, the output is as follows:
```bash
4096
4096
4096
```

Esto significa que `4096` es el blocksize optimo para estos discos y sera el valor utilizado pata el formateo a bajo nivel.

Borrar los discos antes de utilizarlos en el soft-raid:
```bash
root@server:~# apt install tmux -y && \
tmux new -s dd
```

Crear varios paneles y comenzar el borrado del dispositivo de disco `/dev/sdc` en el panel3, utilizando el valor del tamaño de bloque obtenido para dicho disco (en este caso, 4096):
```bash
Ctrl+B
SHIFT+"
SHIFT+"
root@server:~# dd if=/dev/zero of=/dev/sdc bs=4096 status=progress
```

Desplazarse hacia el panel2 y comenzar el borrado del dispositivo de disco `/dev/sdd`, utilizando el valor del tamaño de bloque obtenido para dicho disco:
```bash
Ctrl+B
FlechaArriba
root@server:~# dd if=/dev/zero of=/dev/sdd bs=4096 status=progress
```

Desplazarse hacia el panel1 y comenzar el borrado del dispositivo de disco `/dev/sde`, utilizando el valor del tamaño de bloque obtenido para dicho disco::
```bash
Ctrl+B
FlechaArriba
root@server:~# dd if=/dev/zero of=/dev/sde bs=4096 status=progress
```

Cuando termine de llenar los discos de ceros (borrar los discos), salir de la sesion tmux, ejecutando lo siguiente en cada panel:
```bash
Ctrl+X
y
Ctrl+X
y
Ctrl+X
y
```

Listo!!!, con esto ya tienes los discos completamente borrados a bajo nivel.
