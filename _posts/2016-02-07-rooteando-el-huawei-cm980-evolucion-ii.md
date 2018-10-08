---
layout: post
current: post
navigation: True
title: Rooteando el Huawei CM980 (Evolucion II)
author: l4sh
date: 2016-02-07
category: tech
class: post-template
subclass: post
tags:
  - android
comments: true
---

Este es un post algo viejito. Lo cargo por acá en caso que alguien necesite documentación sobre como realizar el procedimiento.

Luego de un tiempo pariendo con el pésimo servicio que ofrece movilnet para GSM (deberían cambiar el slogan a: movilnet… contigo siempre… excepto en GSM) tenía dos opciones: cambiar de operadora o comprar un teléfono CDMA en la misma operadora. Al final opté por la segunda adquiriendo hace unos dias un Huawei CM980 Evolución II y no me arrepiento. Por primera vez en meses pude hacer una llamada desde mi teléfono que conectara sin tener que remarcar varias veces.

El teléfono es bastante rápido, almenos en comparación con su predecesor (UM840 Evolución), aunque después de un rato jugando con el sientes que falta algo, el preciado root. Hasta el miercoles no se había publicado información sobre como rootear este teléfono, cuando sitios como androidmovida.com y Android Venezuela publicaron la información con los pasos para hacerlo. El método fue descubierto aparentemente por un usuario de taringa llamado SHUNREY16: <http://www.taringa.net/posts/celulares/15840908/root-para-huawei-cm980-en-Venezuela.html>

Si no sabes que es rootear un dispositivo o para que sirve lee el siguiente parrafo, si llegaste buscando como hacerlo completamente desde linux lee más abajo.

Rootear o rooting es el proceso de adquirir privilegios de administrador en un sistema tipo unix, en este caso un dispositivo android. En cuanto a por qué es tan importante para algunos de nosotros el root: principalmente es por el control, poder eliminar las aplicaciones que los fabricantes y operadoras te obligan a tener en el dispositivo que compraste por algun contrato o alianza que tengan con X empresa, poder bloquear anuncios publicitarios a nivel de sistema (bloqueando los hosts que sirven dichos anuncios), restringir el acceso de varias aplicaciones al nivel que creas conveniente y no al que el desarrollador cree conveniente, tomar capturas de pantalla del teléfono, agregar mas potencia a la shell del telefono, overclocking, underclocking y prácticamente cualquier operación que como administrador desees realizar en el telefono; si lo compraste debería ser realmente tuyo y controlado por ti.

Hasta acá todo muy bien, con la excepción que la información publicada esta enfocada a Windows, por lo que decidí publicar los pasos para hacerlo 100% desde GNU/Linux. Una de las principales diferencias entre los metodos publicados es la instalación de los controladores para el teléfono; como sabemos la compatibilidad de hardware es controlada por el kernel por lo que no es necesario instalar nada adicional para detectar el dispositivo (almenos no en Fedora 17 con linux 3.6.2)

Se necesitan los archivos `Recovery_Sonic_u8650_v6.0.1.0_r1` y `Superuser-3.0.7-efgh-signed` los cuales corresponden al recovery para el Huawei U8650 y Superuser.

Los pasos a seguir:
Copiar el archivo `Superuser-3.0.7-efgh-signed.zip` en la tarjeta SD (sin descomprimirlo).

En el telefono ir a *Configuración >> Aplicaciones >> Desarrollo* y activar Depuración USB.

Apagar el telefono (noté que en algunas ocasiones es necesario sacar e introducir de nuevo la batería para que pueda realizarse el siguiente paso).

Presionar y mantener presionados las teclas `VOL DOWN` + `PWR ON` para colocarlo en modo fastboot (debería quedarse en el logo de movilnet)

Descomprimir el archivo `Recovery_Sonic_u8650_v6.0.1.0_r1.zip`. Ir a la carpeta que se creó y dar permisos de ejecucion al script install-recovery-linux.sh y a la herramienta fastboot que se encuentra en la carpeta tools.

```
chmod +x install-recovery-linux.sh
chmod +x tools/fastboot-linux
```
Para comprobar que el teléfono es detectado por fastboot se puede correr y solicitar que liste los dispositivos (es necesario tener privilegios elevados en la maquina donde se ejecuta).

```
sudo tools/fastboot-linux devices
```

En mi caso mostró `???????????? fastboot` lo que quiere decir que encontró un dispositivo, como es el único conectado asumiremos que es el CM980. Ahora toca correr el `install-recovery-linux.sh`.

```
sudo ./install-recovery-linux.sh
```

Lo cual debería dar esta salida

```
erasing ‘recovery’… OKAY
sending ‘recovery’ (4258 KB)… OKAY
writing ‘recovery’… OKAY
rebooting…
```

Luego de esto el teléfono se apaga, para confirmar que el recovery fue instalado es necesario iniciar el telefono en modo recovery presionando y manteniendo las teclas VOL UP + PWR ON. Debería mostrar algo como:

```
- reboot system now
- install zip fom sdcard
- wipe data/factory reset
- wipe cache partition
- backup and restore
- mounts and storage
- advanced
```

Para desplazarse por los menús se usan los botones `VOL UP`, `VOL DOWN` y `PWR ON` para seleccionar.

Seleccionamos *install zip from sdcard >> choose zip from sdcard* y buscamos el archivo zip con el superuser que colocamos al principio (`Superuser-3.0.7-efgh-signed.zip`)

Luego de esto se muestra una pantalla preguntando si se desea realizar la acción con una advertencia en mayúsculas indicando que esto no puede ser desecho, y varias opciones de selección que dicen No. Nos desplazamos hasta `yes - - install Superuser-3.0.7-efgh-signed.zip`, seleccionamos y esperamos. Al terminar mostrará `Install complete. Enjoy!`

Con esto debería estar rooteado el teléfono, si quieres comprobarlo podrias intentar correr `su` en Terminal Emulator y ver que sucede. Si el proceso se completó de forma exitosa debe mostrarse la notificación de Superuser indicando que la aplicacion Terminal Emulator solicita acceso de super usuario, al concederlo el caracter de la shell debería cambiar de `$` a `#`.
