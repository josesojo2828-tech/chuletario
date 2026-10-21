# Chuletario Linux

> Todo lo que necesitas para trabajar con Linux y la terminal en el día a día, en una sola página.

**TL;DR:** En Linux todo es archivo y todo se encadena con pipes. `chmod`/`chown` controlan permisos, `systemctl` y `journalctl` controlan servicios, `grep`/`find`/`awk` procesan texto y `ss`/`curl`/`dig` diagnostican la red. La referencia es **Debian GNU/Linux 13 "trixie"** con kernel **6.12** y **bash 5.2.37** (verificado el 21 de octubre de 2026).

**Audiencia:** Este chulet asume que ya sabes usar una terminal: escribir comandos, moverte entre directorios y abrir un editor. Si no, vuelve al [índice del chuletario](./../README.md) y sigue con el de Bash cuando esté disponible.

---

## 📋 Requisitos

| Herramienta | Versión | Nota |
|-------------|----------|------|
| Debian GNU/Linux | `13` "trixie" | Sistema de referencia. Todo funciona igual en Ubuntu, Fedora y Arch |
| Kernel | `6.12` | `6.12.95+deb13-amd64` en la máquina de referencia |
| bash | `5.2.37` | Shell por defecto de Debian |
| systemd | `257` | Gestor de servicios e init |
| coreutils | `9.7` | `ls`, `cp`, `mv`, `rm`, `cat`, `stat`, `date` |
| GNU grep | `3.11` | `grep -r`, `-E`, `-i`, `-v` |
| GNU sed | `4.9` | `sed -i`, `sed -n`, `s///` |
| awk | `mawk 1.3.4` | El `awk` por defecto en Debian. `gawk` añade extensiones GNU |
| GNU findutils | `4.10.0` | `find -printf`, `-exec`, `-mtime` |
| bash-completion | `2.16` | Autocompletado con Tab |

> En macOS las herramientas del sistema son variantes BSD, no GNU: `sed -i` necesita un argumento vacío (`sed -i ''`), `grep` no tiene `--include` y `find` no tiene `-printf`. Todos los comandos de este chulet son **GNU/Linux**.

> `awk` en Debian es `mawk`, no `gawk`: `mawk` cubre el estándar POSIX y los ejemplos de este chulet, pero no las extensiones de GNU como `gensub` o `asort`. Si necesitas esas, instala `gawk`.

```bash
uname -a
cat /etc/os-release | head -3
bash --version | head -1
```

---

## 🚀 Setup

### Actualizar el sistema

```bash
sudo apt update                    # Refresh the package lists
sudo apt upgrade                   # Upgrade installed packages
sudo apt full-upgrade              # Upgrade + install/remove dependencies
sudo apt autoremove                # Remove packages no longer needed
```

> `apt update` no instala nada: solo refresca la lista de versiones disponibles. Ejecutarlo sin `upgrade` no cambia el sistema, y por eso es seguro correrlo las veces que quieras.

### Crear un usuario con sudo

```bash
sudo adduser ubuntu              # Create the user and its home directory
sudo usermod -aG sudo ubuntu     # Add the user to the sudo group
sudo passwd ubuntu               # Set the password
```

Verificar los grupos de un usuario:

```bash
id ubuntu
# uid=1001(ubuntu) gid=1001(ubuntu) grupos=1001(ubuntu),27(sudo)
```

> Trabajar como `root` todo el tiempo es una mala práctica: un `rm -rf` sin ruta absoluta no pregunta nada. Crear un usuario normal con sudo para las tareas puntuales evita la mayoría de los accidentes.

### Instalar las herramientas de terminal

```bash
sudo apt install -y tree fd-find ripgrep htop tmux curl netcat-openbsd dnsutils
```

> `ripgrep` (`rg`) y `fd` (`fdfind`) son reemplazos modernos de `grep -r` y `find`: ignoran `.gitignore` por defecto y son mucho más rápidos en proyectos grandes. En Debian, `fd` se instala como `fdfind` para no chocar con el paquete `fd` de otro proyecto.

---

## 📖 Cheatsheet

### Navegación y sistema de archivos

| Comando | Qué hace |
|---------|----------|
| `pwd` | Directorio actual |
| `ls -la` | Listar todo, con ocultos y detalles |
| `cd ..` / `cd -` | Subir un nivel / volver al directorio anterior |
| `mkdir -p a/b/c` | Crear directorios anidados |
| `cp -r src/ dest/` | Copiar un directorio completo |
| `mv a.txt b.txt` | Renombrar o mover |
| `rm -r dir/` | Borrar un directorio y su contenido |
| `rm -i *.log` | Borrar pidiendo confirmación |
| `ln -s /ruta/original enlace` | Enlace simbólico |
| `file archivo` | Detectar el tipo real de un archivo |
| `stat archivo` | Permisos, dueño, tamaño y fechas |
| `tree -L 2` | Estructura de directorios hasta 2 niveles |

> `rm` no tiene papelera: lo que borras, se borró. `rm -rf` es especialmente peligroso en scripts, porque un `$VARIABLE` vacío deja el comando como `rm -rf /`. Usa `--` para cortar la opción (`rm -r -- "$DIR/"`) y confirma con `-i` cuando dudes.

### Permisos: `chmod` y `chown`

| Octal | Binario | Permisos | Significado |
|-------|---------|----------|-------------|
| `0` | `000` | `---` | Sin permisos |
| `1` | `001` | `--x` | Solo ejecución |
| `2` | `010` | `-w-` | Solo escritura |
| `3` | `011` | `-wx` | Escritura y ejecución |
| `4` | `100` | `r--` | Solo lectura |
| `5` | `101` | `r-x` | Lectura y ejecución |
| `6` | `110` | `rw-` | Lectura y escritura |
| `7` | `111` | `rwx` | Todos los permisos |

- `4` = lectura (`r`), `2` = escritura (`w`), `1` = ejecución (`x`). Se suman: `6` = `4+2` = lectura y escritura.
- El grupo de tres dígitos es `usuario` / `grupo` / `otros`.

| Comando | Qué hace |
|---------|----------|
| `chmod 644 archivo` | Permisos en octal (rw- r-- r--) |
| `chmod 755 script.sh` | Propiario rwx, grupo y otros rx |
| `chmod u+x script.sh` | Modo simbólico: agrega ejecución al dueño |
| `chmod g-w, o-r archivo` | Quita escritura al grupo y lectura a otros |
| `chmod -R 755 dir/` | Recursivo |
| `chown ubuntu:www-data /var/www/app` | Cambiar dueño y grupo |
| `chown -R ubuntu:www-data /var/www/app` | Recursivo |
| `umask 027` | Permisos por defecto para archivos nuevos (775 → 750) |

```bash
# Typical setup for a web application
sudo chown -R ubuntu:www-data /var/www/app
sudo find /var/www/app -type d -exec chmod 755 {} \;
sudo find /var/www/app -type f -exec chmod 644 {} \;
```

> `chmod` y `chown` son operaciones distintas: los permisos deciden *qué puede hacer* un usuario con el archivo; el dueño decide *quién* puede cambiar esos permisos. Cambiar el dueño de un archivo no le da permisos a nadie más, y cambiar los permisos no cambia el dueño.

### `PATH`, shell y ejecución

| Concepto | Detalle |
|----------|---------|
| `echo $PATH` | Rutas donde el shell busca ejecutables, separadas por `:` |
| `export PATH="$HOME/.local/bin:$PATH"` | Agregar un directorio al `PATH` |
| `which python3` / `command -v python3` | Qué ejecutable se usaría (y si existe) |
| `./script.sh` | Ejecuta con el `shebang` del archivo |
| `bash script.sh` | Ejecuta con bash, ignorando el `shebang` |
| `sh script.sh` | Ejecuta con `sh` (más restrictivo) |
| `source script.sh` / `. script.sh` | Ejecuta en el shell actual: las variables quedan disponibles |
| `exec comando` | Reemplaza el proceso actual por el comando |

```bash
export PATH="$HOME/.local/bin:$PATH"   # Add to ~/.bashrc to make it permanent
hash -r                                # Clear the command cache after changing PATH
```

> `source` y `bash script.sh` no son equivalentes: el primero modifica el shell actual (el `export` de dentro del script afecta a tu sesión), el segundo corre en un proceso hijo (sus variables mueren con el script). Por eso `PATH` o `VIRTUAL_ENV` solo se propagan si usas `source`.

### Códigos de salida y señales

| Código | Significado |
|--------|-------------|
| `0` | Éxito |
| `1` | Error genérico |
| `2` | Uso incorrecto de un comando de shell |
| `126` | El comando existe pero no es ejecutable |
| `127` | Comando no encontrado |
| `128+N` | Terminado por la señal N |
| `130` | Interrumpido con `Ctrl+C` (SIGINT) |
| `143` | Terminado por `SIGTERM` (el `kill` por defecto de `systemctl stop`) |

| Señal | Número | Cuándo se envía |
|-------|--------|-----------------|
| `SIGHUP` | 1 | Terminal cerrada: recarga de configuración |
| `SIGINT` | 2 | `Ctrl+C`: interrupción |
| `SIGKILL` | 9 | `kill -9`: no se puede capturar ni ignorar |
| `SIGTERM` | 15 | Terminación educada: la app puede limpiar recursos |
| `SIGSTOP` | 19 | Pausa el proceso (no se puede capturar) |
| `SIGCONT` | 18 | Reanuda un proceso pausado |
| `SIGPIPE` | 13 | El otro extremo del pipe cerró (por eso `yes \| head` termina) |

> `kill -9` es el último recurso: mata el proceso sin darle oportunidad de cerrar archivos, vaciar buffers o terminar transacciones. Prueba siempre `kill` (que envía `SIGTERM`) y espera unos segundos antes de recurrir a `-9`.

### Procesos

| Comando | Qué hace |
|---------|----------|
| `ps aux` | Procesos de todos los usuarios |
| `ps -ef` | Formato BSD con la columna PPID (proceso padre) |
| `top` / `htop` | Uso de CPU y memoria en vivo |
| `kill PID` / `kill -TERM PID` | Enviar una señal |
| `kill -9 PID` | Matar sin permiso (no se puede evitar) |
| `pkill -f "python app.py"` | Matar por patrón de nombre |
| `jobs` / `bg` / `fg` | Trabajos en segundo y primer plano del shell |
| `nohup comando &` | Seguir corriendo después de cerrar la sesión |
| `lsof -i :8080` | Proceso que tiene abierto un puerto |
| `uptime` | Tiempo activo y carga media del sistema |

> `Ctrl+Z` suspende el proceso en primer plano (queda en `jobs`), `Ctrl+C` lo interrumpe, y `fg`/`bg` lo retoman. `nohup` y las unidades de systemd son las dos formas de que un proceso sobreviva al cierre de la terminal; para servicios de verdad, systemd.

### systemd

| Comando | Qué hace |
|---------|----------|
| `systemctl status nginx` | Estado del servicio + últimas líneas del log |
| `systemctl start nginx` | Arrancar ahora |
| `systemctl restart nginx` | Reiniciar (stop + start) |
| `systemctl enable --now nginx` | Habilitar en el arranque y arrancar ahora |
| `systemctl disable nginx` | Quitar del arranque |
| `systemctl is-enabled nginx` | ¿Está habilitado? |
| `systemctl list-units --type=service` | Servicios activos |
| `systemctl cat nginx` | Ver el archivo de la unidad |
| `systemctl daemon-reload` | Releer los archivos de unidad tras editarlos |
| `journalctl -u nginx -f` | Seguir el log del servicio en vivo |
| `journalctl -u nginx --since "1 hour ago"` | Log de la última hora |
| `journalctl --disk-usage` | Cuánto espacio ocupan los logs |

```bash
sudo systemctl edit nginx      # Override without touching the original unit
sudo systemctl daemon-reload
sudo systemctl restart nginx
```

> `systemctl enable` no arranca el servicio: solo lo agrega al arranque. `--now` hace las dos cosas. Editar `/lib/systemd/system/nginx.service` a mano se pierde en la próxima actualización del paquete; `systemctl edit` crea un override en `/etc/systemd/system/` que sobrevive.

### Disco y memoria

| Comando | Qué hace |
|---------|----------|
| `df -h` | Espacio de cada sistema de archivos montado |
| `du -sh /var/log` | Tamaño total de un directorio |
| `du -h --max-depth=1 /var \| sort -h` | Un nivel hacia abajo, ordenado |
| `lsblk` | Discos y particiones en árbol |
| `lsblk -f` | Con sistema de archivos y puntos de montaje |
| `free -h` | Memoria RAM y swap |
| `lsof +L1` | Archivos borrados que siguen abiertos |
| `ncdu` | Explorador de disco interactivo (TUI) |

> `df` informa del **sistema de archivos** (el bloque donde se reserva el espacio) y `du` del **contenido**. Por eso pueden discrepar: un archivo borrado que un proceso tiene abierto sigue ocupando espacio en `df` hasta que el proceso cierra el descriptor. Reiniciar el proceso (o el `lsof +L1` que lo muestra) libera el espacio.

### Red

| Comando | Qué hace |
|---------|----------|
| `ip addr show` | Direcciones IP de cada interfaz |
| `ip route` | Tabla de enrutamiento y gateway |
| `ip link set eth0 up` / `down` | Activar o desactivar una interfaz |
| `ss -tulpn` | Puertos escuchando y sus procesos |
| `ss -tan state established` | Conexiones TCP activas |
| `ping -c 4 1.1.1.1` | Comprobar conectividad (ICMP) |
| `curl -I https://example.com` | Solo las cabeceras de la respuesta |
| `curl -sS -o /dev/null -w '%{http_code}\n' https://example.com` | Solo el código HTTP |
| `dig example.com A +short` | Resolver DNS |
| `dig @1.1.1.1 example.com +short` | Consultar un resolver concreto |
| `nc -zv example.com 443` | Comprobar si un puerto acepta conexiones |

> `ping` usa ICMP y muchos servidores y firewalls lo bloquean: que no responda no significa que el sitio esté caído. Para probar un servicio real, `curl -I` o `nc -zv` contra el puerto exacto son pruebas mucho más útiles.

### Procesamiento de texto

| Comando | Qué hace |
|---------|----------|
| `grep -rn "TODO" src/` | Buscar recursivamente con número de línea |
| `grep -i -E "error\|warn" app.log` | Sin distinguir mayúsculas, regex extendida |
| `grep -v "DEBUG" app.log` | Invertir la coincidencia |
| `grep -l "error" *.log` | Solo los nombres de los archivos con coincidencias |
| `sed -n '10,20p' archivo` | Imprimir un rango de líneas |
| `sed 's/foo/bar/g' archivo` | Reemplazar texto (imprime en stdout) |
| `awk '{print $1}' archivo` | Imprimir la primera columna |
| `awk -F',' '{print $2}' datos.csv` | Separador personalizado |
| `sort -k2 -n` | Ordenar por la segunda columna, numéricamente |
| `sort -u` | Ordenar y eliminar duplicados |
| `uniq -c` | Contar líneas repetidas (precedido de `sort`) |
| `cut -d, -f2,3 datos.csv` | Extraer columnas |
| `wc -l` | Contar líneas |
| `head -n 20` / `tail -n 20` | Primeras / últimas líneas |
| `tail -f app.log` | Seguir el archivo en vivo |
| `xargs` | Construir comandos a partir de la entrada |
| `xargs -0` | Separador nulo: seguro con nombres con espacios |

> `uniq` solo compara líneas **vecinas**: por eso siempre va después de `sort`. Y `xargs` sin `-0` parte la línea por espacios, así que un archivo llamado `mi informe.txt` se convierte en dos argumentos y rompe el comando.

### Pipes y redirecciones

| Sintaxis | Qué hace |
|----------|----------|
| `cmd1 \| cmd2` | La salida de `cmd1` es la entrada de `cmd2` |
| `cmd > archivo` | Redirige stdout (trunca el archivo) |
| `cmd >> archivo` | Redirige stdout (agrega al final) |
| `cmd 2> archivo` | Redirige solo stderr |
| `cmd &> archivo` | Redirige stdout y stderr juntos (bash) |
| `cmd 2>&1` | stderr pasa a donde esté stdout |
| `cmd \| tee salida.log` | Muestra en pantalla y guarda en el archivo |
| `cmd \|& cmd2` | stdout y stderr hacia cmd2 (bash) |

> `>` trunca: si un script falla a mitad de camino, el log anterior ya se perdió. Usa `>>` para no perder historia, y `2>&1 | tee` cuando quieras ver el error en pantalla y en el log a la vez.

### Archivos comprimidos

| Comando | Qué hace |
|---------|----------|
| `tar -czf backup.tar.gz dir/` | Crear un `.tar.gz` |
| `tar -xzf backup.tar.gz` | Extraer |
| `tar -tzf backup.tar.gz` | Listar el contenido sin extraer |
| `zip -r backup.zip dir/` | Crear un `.zip` |
| `unzip -l backup.zip` | Listar el contenido |
| `unzip backup.zip` | Extraer |
| `zcat archivo.log.gz \| grep "error"` | Buscar dentro de un comprimido |
| `zgrep "error" archivo.log.gz` | Lo mismo, más corto |

> `tar` con `z`, `x` y `f` es lo que vas a usar el 95% del tiempo. `tar` no comprime por sí mismo: la `z` delega en `gzip`. Para un directorio, `tar` conserva rutas, permisos y enlaces; `zip` es más portable entre Windows y Linux.

### SSH, SCP y rsync

| Comando | Qué hace |
|---------|----------|
| `ssh usuario@servidor` | Conexión interactiva |
| `ssh usuario@servidor "df -h"` | Ejecuta un comando remoto y sale |
| `ssh -L 8080:localhost:80 usuario@servidor` | Túnel local (port forwarding) |
| `scp archivo.txt usuario@servidor:/tmp/` | Copiar un archivo |
| `scp -r dir/ usuario@servidor:/tmp/` | Copiar un directorio |
| `rsync -avz src/ destino/` | Copia incremental y comprimida |
| `rsync -avz --delete src/ destino/` | Igual, pero borra lo que sobra |
| `rsync -avz --progress src/ usuario@servidor:/tmp/` | Por red, con barra de progreso |

> `scp` copia siempre el archivo completo. `rsync` compara tamaño y fecha y solo transfiere las diferencias: sobre SSH es la herramienta correcta para sincronizar carpetas grandes. Cuidado con `--delete`: el destino queda como una copia exacta del origen, incluidas las eliminaciones.

---

## 💻 Ejemplos

### Ejemplo 1 — encontrar los archivos más grandes

```bash
# Top 10 files by size in the current directory tree
du -ah . 2>/dev/null | sort -rh | head -10
```

Salida:

```text
4,5M	.
2,9M	./logs/big-debug.log
1,2M	./logs/medium.log
492K	./logs/small.log
4,0K	./logs/access.log
```

Variantes útiles:

```bash
# Only files larger than 100 MB, anywhere under /var
find /var -type f -size +100M -printf '%s %p\n' 2>/dev/null | sort -rn | head

# Delete rotated logs older than 30 days
find /var/log -name '*.gz' -type f -mtime +30 -delete
```

> `du -ah` necesita `2>/dev/null` porque al recorrer todo el árbol se topa con directorios sin permiso de lectura y escribe errores en stderr que ensucian la salida. `find -printf` es una extensión de GNU: en BSD no existe.

### Ejemplo 2 — pipeline de `grep` + `awk` sobre un log de acceso

```bash
awk '{print $9}' access.log | sort | uniq -c | sort -rn
```

Salida:

```text
      4 200
      1 500
      1 404
```

Las rutas más solicitadas entre las respuestas correctas:

```bash
grep ' 200 ' access.log | awk '{print $7}' | sort | uniq -c | sort -rn | head -3
```

Salida:

```text
      2 /api/items
      1 /index.html
      1 /assets/app.js
```

Y las peticiones que fallaron, con su código:

```bash
awk '$9 != 200 {print $7, $9}' access.log
```

Salida:

```text
/api/items 500
/login 404
```

> En un log con formato `nginx` los campos son: 1 IP, 2 y 3 usuario, 4 fecha, 5 zona horaria, 6 método, 7 ruta, 8 protocolo, 9 estado y 10 bytes. Los números de campo son posicionales: si cambia el formato del log, hay que recalcular los índices.

### Ejemplo 3 — qué proceso está usando un puerto

```bash
# Without root: only your own processes are visible
ss -tulpn | grep ':8080'

# With root: the process name and PID are shown
sudo ss -tulpn | grep ':8080'
```

Salida:

```text
tcp   LISTEN 0      5      127.0.0.1:8080      0.0.0.0:*    users:(("python3",pid=662757,fd=3))
```

Alternativas:

```bash
# Which process has the port open
sudo lsof -i :8080

# Which ports is this process using
sudo lsof -p 662757 -i

# All the listening ports of the system
ss -tuln

# Is the service answering?
curl -I http://127.0.0.1:8080
```

> `ss` sin `sudo` solo muestra los procesos propios: por eso en un servidor la primera versión no muestra nada. `ss` reemplaza a `netstat`, que está obsoleto y no viene instalado por defecto en Debian 13.

---

## 🧰 10 comandos que uso todos los días

| Comando | Para qué lo uso |
|---------|-----------------|
| `cd -` | Volver al directorio anterior sin memorizar la ruta |
| `ls -lat` | Ver los archivos más recientes primero, con detalles |
| `grep -rn "texto" .` | Buscar en todo el proyecto, con archivo y línea |
| `find . -name "*.log" -mtime +7 -delete` | Limpiar logs viejos |
| `ps aux --sort=-%mem \| head` | Ver qué proceso se come la memoria |
| `du -sh * \| sort -h` | Qué ocupa espacio en el directorio actual |
| `ss -tulpn` | Qué puertos están escuchando y quién los tiene |
| `systemctl status <servicio>` | Estado de un servicio y sus últimas líneas de log |
| `journalctl -u <servicio> -f` | Seguir el log de un servicio en vivo |
| `!!` | Repetir el último comando; `sudo !!` reintenta con permisos |

---

## ⚠️ Gotchas

| Gotcha | Qué pasa | Solución |
|--------|----------|----------|
| `rm -rf` con una variable vacía | `rm -rf /` borra el sistema entero | Valida la variable y usa `rm -rf -- "$DIR/"` |
| `chmod 777` para "arreglar" permisos | Abre el archivo a todos y elimina la trazabilidad | `755` para directorios y ejecutables, `644` para archivos |
| `kill -9` como primera opción | El proceso no puede limpiar nada y deja archivos abiertos | Envía `SIGTERM` primero; reserva `-9` para procesos colgados |
| `chmod` y `chown` confundidos | Cambiar permisos no cambia el dueño | `chown` para el dueño, `chmod` para los permisos |
| `uniq` sin `sort` antes | Solo compara líneas vecinas, no cuenta repetidos globales | `sort \| uniq -c` |
| `xargs` sin `-0` | Rompe con nombres de archivo que tienen espacios | `find ... -print0 \| xargs -0` |
| `sudo` en un pipeline | Solo el primer comando del pipe corre como root | `sudo cmd1 \| cmd2` no protege a `cmd2`; usa `sudo sh -c "cmd1 \| cmd2"` |
| `tar` sin `-C` ni rutas correctas | Extrae donde no esperas y sobrescribe archivos | Revisa el contenido con `tar -tzf` antes de extraer |
| Script sin `shebang` | `Exec format error` al ejecutarlo directamente | Primera línea: `#!/usr/bin/env bash` y `chmod +x` |
| `apt install` sin `apt update` | Instala una versión vieja o no encuentra el paquete | `sudo apt update` primero |

---

## ⚖️ Trade-offs

| Decisión | Ganas | Pierdes |
|----------|-------|--------|
| `systemd` sobre scripts en `rc.local` | Reinicio, logs, dependencias y estado centralizados | Archivos de unidad que hay que aprender |
| Claves SSH sobre contraseñas | Acceso automatizable y sin exponer la contraseña | Requiere administrar `authorized_keys` |
| `rsync` sobre `scp` | Transfiere solo las diferencias | No es estándar en Windows |
| `find` sobre `locate` | Exacto y en vivo | Más lento en el primer barrido del sistema |
| `awk` sobre un script en Python | Sin dependencias y muy rápido en streaming | Menos legible para transformaciones complejas |
| `journalctl` sobre logs en `/var/log` | Rotación, filtros y consultas por unidad | Requiere `systemd` |
| `tmux` sobre terminales múltiples | Sesiones que sobreviven a la desconexión | Hay que aprender sus atajos |
| Herramientas GNU sobre las de BSD | Comportamiento uniforme entre distribuciones | No funcionan igual en macOS |

---

## 📚 Further reading

- [Documentación de Debian](https://www.debian.org/doc/) — el índice de toda la documentación oficial
- [Man pages de Debian](https://manpages.debian.org/) — el manual de cada paquete, para la versión estable actual
- [The Linux Command Line](https://linuxcommand.org/) — libro completo y gratuito de William Shotts
- [systemd.io](https://systemd.io/) — la documentación oficial de systemd, con [NETWORK_ONLINE.target](https://systemd.io/NETWORK_ONLINE/)
- [explainshell](https://explainshell.com/) — descompone un comando largo opción por opción
