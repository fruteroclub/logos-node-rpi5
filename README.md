# Logos Blockchain Node en Raspberry Pi 5

Taller de [fruteroclub](https://fruteroclub.com) para instalar y correr un nodo de [Logos Network](https://logos.co/) en una Raspberry Pi 5.

No necesitas ser programador ni tener experiencia tecnica. Si puedes seguir instrucciones en una terminal, puedes levantar un nodo.

> Si encuentras un termino que no conoces, consulta el [Glosario de terminos](GLOSARIO.md).

---

## Que es todo esto?

### Raspberry Pi 5

Una Raspberry Pi es un ordenador pequeno y barato (del tamano de una tarjeta de credito) que puede hacer muchas de las cosas que hace tu ordenador normal. La Pi 5 es la version mas reciente: tiene un procesador ARM de 64 bits, suficiente potencia para correr un nodo de blockchain 24/7 con muy poco consumo de energia.

Piensa en ella como un mini servidor que puedes tener en tu casa.

### Logos Network

[Logos](https://logos.co/) es una blockchain enfocada en privacidad y resistencia a la censura. Usa pruebas de conocimiento cero (zero-knowledge proofs) para que las transacciones sean privadas, y un sistema de consenso llamado Cryptarchia (Proof of Stake).

Correr un nodo significa que tu Pi participa directamente en la red: valida transacciones, mantiene una copia de la cadena, y eventualmente puede producir bloques.

### SSH (Secure Shell)

SSH es una forma de controlar un ordenador remotamente desde otro. En vez de conectar monitor, teclado y raton a la Pi, la controlamos desde nuestro ordenador escribiendo comandos en una terminal. Todo lo que escribes viaja encriptado por el cable de red.

### Que vamos a hacer?

1. Preparar la tarjeta SD de la Pi desde nuestro ordenador
2. Conectar la Pi a nuestra red por cable ethernet
3. Acceder a la Pi remotamente por SSH
4. Descargar e instalar el nodo de Logos
5. Verificar que el nodo funciona y se conecta a la red

---

## Requisitos

**Hardware:**
- Raspberry Pi 5 (con su fuente de alimentacion USB-C)
- Tarjeta microSD (minimo 16GB) con Raspberry Pi OS (64-bit) ya instalado
- Cable ethernet
- Un router con puerto ethernet libre

**Software (en tu ordenador):**
- Un sistema Linux (este taller usa Debian) o Mac
- Terminal / linea de comandos
- Lector de tarjetas SD (integrado o USB)

**Conocimientos previos:** Ninguno. Solo ganas de aprender.

---

## Paso 1 — Preparar la SD desde tu ordenador

Antes de encender la Pi necesitamos configurar algunas cosas. Para eso, insertamos la tarjeta SD de la Pi en nuestro ordenador y modificamos archivos directamente.

### 1.1 Insertar la SD y verificar que el sistema la reconoce

Cuando insertas una SD con Raspberry Pi OS, tu ordenador la "monta" automaticamente (es decir, la hace accesible como si fuera una carpeta). Para confirmar que la ve, abrimos una terminal y escribimos:

```bash
lsblk
```

`lsblk` lista todos los dispositivos de almacenamiento conectados. Deberiamos ver algo asi:

```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
mmcblk0     179:0    0   117G  0 disk
├─mmcblk0p1 179:1    0   512M  0 part /media/tu-usuario/bootfs
└─mmcblk0p2 179:2    0 116.5G  0 part /media/tu-usuario/rootfs
nvme0n1     259:0    0 238.5G  0 disk
├─nvme0n1p1 259:1    0   512M  0 part /boot/efi
├─nvme0n1p2 259:2    0   237G  0 part /
└─nvme0n1p3 259:3    0   977M  0 part [SWAP]
```

Lo que nos importa es `mmcblk0` — esa es la tarjeta SD. Tiene dos particiones:

- **`bootfs`** — la particion de arranque. Aqui estan los archivos que la Pi lee al encender: el kernel (nucleo del sistema operativo), configuracion de hardware (`config.txt`), y parametros de inicio (`cmdline.txt`). Es una particion pequena (512MB) en formato FAT32 para que cualquier sistema pueda leerla.

- **`rootfs`** — el sistema de archivos raiz. Aqui esta TODO el sistema operativo: programas, configuracion, la carpeta `/home` con los archivos del usuario, etc. Es el equivalente a tu disco `C:\` en Windows o `/` en Linux.

> **Nota:** `nvme0n1` es el disco duro de tu ordenador — no lo toques. Solo trabajamos con `mmcblk0`.

### 1.2 Explorar la particion de boot

Podemos ver que hay dentro:

```bash
ls /media/tu-usuario/bootfs/
```

Veras archivos como:

```
bcm2712-rpi-5-b.dtb    config.txt    kernel_2712.img
cmdline.txt             overlays/     start4.elf
```

Estos son archivos del firmware de la Raspberry Pi. Los mas importantes:
- `config.txt` — configuracion de hardware (overclocking, pantalla, GPIO, etc.)
- `cmdline.txt` — parametros que se pasan al kernel de Linux al arrancar
- `kernel_2712.img` — el kernel de Linux compilado para la Pi 5 (el "2712" es el chip BCM2712 de la Pi 5)

### 1.3 Habilitar SSH

Por defecto, Raspberry Pi OS trae SSH desactivado por seguridad (no quieres que alguien entre a tu Pi sin que lo sepas). Para activarlo sin necesidad de conectar pantalla ni teclado, hay un truco: si la Pi encuentra un archivo llamado `ssh` (vacio, sin extension) en la particion de boot, activa el servidor SSH automaticamente en el primer arranque.

```bash
sudo touch /media/tu-usuario/bootfs/ssh
```

Desglose del comando:
- `sudo` — ejecutar como administrador (necesario para escribir en la SD)
- `touch` — crea un archivo vacio (o actualiza la fecha si ya existe)
- La ruta apunta a la particion de boot de la SD

Puedes verificar que se creo:

```bash
ls -la /media/tu-usuario/bootfs/ssh
```

### 1.4 Verificar que existe un usuario en el sistema

Antes de intentar conectarnos por SSH, necesitamos saber con que usuario vamos a entrar. Los usuarios de Linux tienen su carpeta en `/home`, asi que miramos en el rootfs de la SD:

```bash
ls /media/tu-usuario/rootfs/home/
```

```
pi
```

Existe el usuario `pi` — ese es el usuario por defecto de Raspberry Pi OS. Lo usaremos para conectarnos por SSH.

> **Sobre el usuario `pi`:** En versiones antiguas de Raspberry Pi OS, el usuario `pi` venia con la contrasena `raspberry`. En versiones mas recientes, puede que te pida crear usuario y contrasena en el primer arranque (mostrando un asistente en pantalla). Si flasheaste el OS con Raspberry Pi Imager, pudiste configurar el usuario y contrasena durante el flasheo.

### 1.5 Desmontar la SD de forma segura

Antes de sacar la SD, hay que desmontarla para asegurarnos de que todos los datos se escribieron correctamente:

```bash
sudo umount /media/tu-usuario/bootfs
sudo umount /media/tu-usuario/rootfs
```

O clic derecho en el icono de la SD en tu escritorio y "Expulsar".

Ya puedes sacar la SD del ordenador.

---

## Paso 2 — Conectar la Pi y acceder por SSH

### 2.1 Conexion fisica

1. **Inserta la microSD** en la ranura de la Raspberry Pi 5 (esta en la parte inferior)
2. **Conecta el cable ethernet** de tu router a la Pi (puerto ethernet de la Pi, el grande)
3. **Conecta la alimentacion** (cable USB-C) — la Pi arranca automaticamente, no tiene boton de encendido
4. **Espera ~30-60 segundos** — la Pi esta arrancando, configurando la red, y (gracias a nuestro archivo `ssh`) activando el servidor SSH

Sabras que encendio porque el LED verde de la Pi parpadeara.

### 2.2 Encontrar la IP de la Pi en tu red

Tu router le asigno una IP a la Pi automaticamente (via DHCP). Necesitamos saber cual es.

**Opcion A: por nombre (mDNS)**

```bash
ping raspberrypi.local
```

Si funciona, veras algo como `PING raspberrypi.local (192.168.1.XX)`. Esa es la IP.

> Esto funciona porque la Pi tiene Avahi (mDNS) instalado, que permite encontrar dispositivos por nombre en la red local sin configurar DNS.

**Opcion B: escanear la red**

Si `raspberrypi.local` no responde:

```bash
# Primero, averigua en que rango esta tu red
ip route | grep default
# Ejemplo de salida: default via 192.168.1.1 dev enp2s0

# Escanea todos los dispositivos en ese rango
nmap -sn 192.168.1.0/24
```

Busca la entrada que diga "Raspberry Pi" o un dispositivo nuevo que no reconozcas.

> Si no tienes `nmap`, instalalo con: `sudo apt install nmap`

**Opcion C: mirar en tu router**

Entra a la interfaz web de tu router (generalmente `192.168.1.1` en el navegador) y busca la lista de dispositivos conectados.

### 2.3 Conectar por SSH

Ya con la IP (o el hostname), nos conectamos:

```bash
ssh pi@raspberrypi.local
# o con IP directa:
ssh pi@192.168.1.XX
```

La primera vez te preguntara si confias en este servidor (fingerprint). Escribe `yes`.

Luego te pide la contrasena. Si usaste la contrasena por defecto: `raspberry`

Si todo va bien, veras algo asi:

```
Linux raspberrypi 6.12.47+rpt-rpi-2712 #1 SMP PREEMPT Debian 1:6.12.47-1+rpt1 aarch64

pi@raspberrypi:~ $
```

Estas dentro de la Pi! Todo lo que escribas ahora se ejecuta en la Raspberry Pi, no en tu ordenador.

> Puede que veas un aviso: "SSH is enabled and the default password for the 'pi' user has not been changed." Esto es un recordatorio de seguridad — lo solucionamos en el siguiente paso.

> **Importante:** Cambia la contrasena por defecto inmediatamente:
> ```bash
> passwd
> ```
> Te pide la contrasena actual y luego la nueva (dos veces).

---

## Paso 3 — Instalar el nodo Logos (dentro de la Pi por SSH)

A partir de aqui, todos los comandos se ejecutan dentro de la Pi (en tu sesion SSH).

### 3.1 Actualizar el sistema

Buena practica antes de instalar cualquier cosa:

```bash
sudo apt update && sudo apt upgrade -y
```

### 3.2 Descargar los binarios del nodo

El nodo de Logos tiene binarios precompilados para la arquitectura ARM64 (que es la de la Pi 5). No necesitamos compilar nada.

```bash
# Descargar el binario del nodo
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-node-linux-aarch64-0.2.1.tar.gz

# Descargar los circuitos ZK (necesarios para las pruebas de conocimiento cero)
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-circuits-v0.4.1-linux-aarch64.tar.gz
```

> **Que es `aarch64`?** Es el nombre tecnico de la arquitectura ARM de 64 bits. La Pi 5 usa un chip BCM2712 con nucleos Cortex-A76, que son ARM de 64 bits. Por eso descargamos la version `aarch64`.

> **Que son los circuitos ZK?** Logos usa zero-knowledge proofs (pruebas de conocimiento cero) para privacidad. Estos circuitos son las "reglas matematicas" precompiladas que el nodo necesita para verificar y crear esas pruebas.

### 3.3 Extraer e instalar

```bash
# Extraer los circuitos ZK
tar -xzf logos-blockchain-circuits-v0.4.1-linux-aarch64.tar.gz

# Extraer el binario del nodo
tar -xzf logos-blockchain-node-linux-aarch64-0.2.1.tar.gz
```

> `tar -xzf` descomprime archivos `.tar.gz`: `x` = extraer, `z` = descomprimir gzip, `f` = archivo

**Importante:** El directorio extraido incluye la version en el nombre. Hay que renombrarlo a donde el nodo lo espera:

```bash
mv logos-blockchain-circuits-v0.4.1-linux-aarch64 ~/.logos-blockchain-circuits
```

> **Tip:** Si no sabes como se llama el directorio extraido, puedes ver el contenido del tar antes de extraer con: `tar -tzf archivo.tar.gz | head -5`

Verifica que el binario existe:

```bash
ls -la logos-blockchain-node
```

### 3.4 Inicializar el nodo

Este comando genera un archivo de configuracion (`user_config.yaml`) con las claves de tu nodo y lo conecta con los peers del devnet (la red de desarrollo):

```bash
./logos-blockchain-node init \
    -p /ip4/65.109.51.37/udp/3000/quic-v1/p2p/12D3KooWL7a8LBbLRYnabptHPFBCmAs49Y7cVMqvzuSdd43tAJk8 \
    -p /ip4/65.109.51.37/udp/3001/quic-v1/p2p/12D3KooWPLeAcachoUm68NXGD7tmNziZkVeMmeBS5NofyukuMRJh \
    -p /ip4/65.109.51.37/udp/3002/quic-v1/p2p/12D3KooWKFNe4gS5DcCcRUVGdMjZp3fUWu6q6gG5R846Ui1pccHD \
    -p /ip4/65.109.51.37/udp/3003/quic-v1/p2p/12D3KooWAnriLgXyQnGTYz1zPWPkQL3rthTKYLzuAP7MMnbgsxzR \
    --no-public-ip-check
```

Desglose:
- `init` — modo de inicializacion, genera configuracion y claves
- `-p` — cada linea es un "peer" (otro nodo) al que conectarse. El formato es: IP/protocolo/ID-del-peer
- `--no-public-ip-check` — necesario porque la Pi esta detras de tu router (NAT), no tiene IP publica directa

### 3.5 Ejecutar el nodo

Antes de ejecutar el nodo directamente, es mejor lanzarlo dentro de `screen` para que siga corriendo aunque cierres la terminal SSH:

```bash
sudo apt install screen -y
screen -S logos
./logos-blockchain-node user_config.yaml
```

El nodo arranca y empiezas a ver muchos logs pasando rapido. El estado inicial sera **"Bootstrapping"** (sincronizando la cadena), y despues de un rato pasara a **"Online"**.

Para "desconectarte" de screen sin detener el nodo: presiona `Ctrl+A` y luego `D`. El nodo sigue corriendo en segundo plano.

Para volver a ver los logs: `screen -r logos`

### 3.6 Verificar que funciona

Abre **otra terminal** en tu ordenador y conectate de nuevo por SSH:

```bash
ssh pi@raspberrypi.local
```

Desde esa segunda sesion, consulta el estado del nodo:

```bash
curl -w "\n" http://localhost:8080/cryptarchia/info
```

Veras algo asi:

```json
{
  "lib": "1820d270a382...",
  "tip": "55539dee68bd...",
  "slot": 46926,
  "height": 3242,
  "mode": "Bootstrapping"
}
```

- **height** — el numero de bloque en el que va tu nodo (va subiendo mientras sincroniza)
- **mode** — el estado actual. Empieza en `Bootstrapping` (descargando bloques) y cuando alcance la cadena cambia a `Online`
- **slot** — la posicion temporal actual en el consenso
- **tip** — el hash del ultimo bloque que tu nodo conoce

> **Nota:** Veras muchos logs pasando rapido en la primera terminal, con numeros que se repiten (como "120"). Eso es normal — el nodo esta descargando y procesando bloques. Puede tardar varios minutos dependiendo de cuantos bloques tenga que sincronizar.

### 3.7 Monitorear el progreso de sincronizacion

Puedes ejecutar el `curl` varias veces para ver como avanza:

```
Primera consulta:      height: 3242    slot: 46926    mode: "Bootstrapping"
~10 min despues:       height: 7287    slot: 119972   mode: "Bootstrapping"
~30 min despues:       height: 9152    slot: 150031   mode: "Bootstrapping"
~1 hora despues:       height: 16333   slot: 283270   mode: "Bootstrapping"
...
Cuando alcance la red: height: 156345  slot: ??????   mode: "Online"
```

El `height` sube mientras descarga bloques. Cuando tu nodo alcance el bloque mas reciente de la red, `mode` cambiara de `Bootstrapping` a `Online`. Eso significa que tu nodo ya esta sincronizado y participando en la red en tiempo real.

Para saber cuanto te falta, compara tu `height` con el height actual de la red en el [Block Explorer](https://devnet.blockchain.logos.co/web/explorer/). En nuestro caso, la red iba en ~156,000 bloques.

> **Cuanto tarda?** En nuestra experiencia (abril 2026), la cadena del devnet tenia ~156,000 bloques. En una Pi 5 conectada por ethernet, el nodo sincronizaba aproximadamente 16,000 bloques por hora. Eso da un estimado de **8-10 horas** para una sincronizacion completa desde cero. Ten paciencia — es como descargar el historial completo de la red.

> **Tip:** No necesitas estar mirando todo el rato. Lanza el nodo dentro de `screen`, desconecta con `Ctrl+A` + `D`, y vuelve a revisar mas tarde. Si necesitas apagar la Pi (por ejemplo, se acaba la bateria), el nodo retoma donde se quedo al volver a encender (ver seccion "Apagar la Pi de forma segura" mas abajo).

---

## Paso 4 — Obtener fondos y producir bloques

Para que tu nodo produzca bloques, necesitas fondos del devnet (tokens de prueba).

### 4.1 Encontrar tu clave de wallet

```bash
grep -A3 known_keys user_config.yaml
```

Copia la clave que aparece.

### 4.2 Pedir fondos del faucet

Ve al faucet del devnet: https://devnet.blockchain.logos.co/web/faucet/

> Necesitas credenciales que se obtienen en el [Discord de Logos](https://discord.com/channels/973324189794697286/1468535289604735038).

### 4.3 Verificar tu balance

```bash
curl -w "\n" http://localhost:8080/wallet/TU_CLAVE/balance
```

### 4.4 Produccion de bloques

Despues de recibir fondos y esperar ~3.5 horas (dos epochs del consenso), tu nodo empieza a producir bloques automaticamente. Felicidades, tu Raspberry Pi esta participando activamente en la red!

---

## Tips utiles

### Mantener el nodo corriendo al cerrar SSH

Si lanzaste el nodo con `screen` (como recomendamos en el paso 3.5), ya esta cubierto. Solo recuerda:

- Desconectar screen: `Ctrl+A` luego `D`
- Reconectar: `screen -r logos`
- Ver sesiones activas: `screen -ls`

**Alternativa con nohup** (mas sencillo pero sin acceso a los logs en vivo):

```bash
nohup ./logos-blockchain-node user_config.yaml > logos.log 2>&1 &
# Ver los logs despues:
tail -f logos.log
```

### Apagar la Pi de forma segura

Esto es importante: **nunca desconectes la alimentacion de golpe**. Si la Pi se apaga sin avisar, puede corromper la tarjeta SD y perder datos. Siempre apaga limpio:

```bash
ssh pi@192.168.1.69
sudo shutdown now
```

Espera unos segundos hasta que el LED verde deje de parpadear. Ya puedes desconectar la alimentacion.

### Reanudar el nodo despues de apagar

Cuando vuelvas a encender la Pi (conectando la alimentacion), solo necesitas:

```bash
ssh pi@192.168.1.69
screen -S logos
./logos-blockchain-node user_config.yaml
```

El nodo **retoma donde se quedo**. Los bloques que ya descargo quedan guardados en la SD. Si estaba sincronizando y llevaba 16,000 bloques, sigue desde ahi — no empieza de cero.

> **Ejemplo real:** En nuestro taller, la cadena tenia ~156,000 bloques. La sincronizacion tardo varias horas. Tuvimos que apagar la Pi (se acababa la bateria de la power bank) y al reconectar, el nodo retomo exactamente donde se habia quedado.

### Sobre la alimentacion

La Raspberry Pi 5 necesita un cargador de **5V / 5A** (25W) con conector USB-C. El cargador oficial de Raspberry Pi es la mejor opcion (~12 euros).

**Cargadores que NO sirven:**
- Cargadores de telefono tipicos (5V/2A) — insuficientes, causan throttling
- Power banks — funcionan temporalmente pero se agotan y pueden cortar sin aviso
- Cargadores USB genericos de 500mA — la Pi ni enciende

**Cargadores que SI sirven:**
- Cargador oficial Raspberry Pi 5 (5V/5A) — recomendado
- Cualquier cargador USB-C PD de al menos 27W (5V/3A minimo)

> Si la Pi no recibe suficiente energia, veras un icono de rayo en pantalla y el procesador se frena (throttling) para consumir menos. Esto hace que la sincronizacion sea mucho mas lenta.

### Reiniciar la Pi remotamente

```bash
sudo reboot
```

Espera 30 segundos y vuelve a conectar por SSH.

### Ver cuanta memoria/CPU usa el nodo

```bash
htop
```

---

## Que es lo que acabamos de hacer?

Resumen de lo que lograste en este taller:

1. **Aprendiste a montar y modificar una SD** desde otro ordenador — util para configurar dispositivos "headless" (sin pantalla)
2. **Habilitaste SSH** creando un simple archivo en la particion de boot — un truco clasico de Raspberry Pi
3. **Te conectaste remotamente** a otro ordenador en tu red local usando SSH
4. **Instalaste un nodo de blockchain** descargando binarios precompilados
5. **Tu Pi es ahora un participante activo** en la red de Logos, validando transacciones y (potencialmente) produciendo bloques

Todo esto con un ordenador de ~70 euros que consume menos de 15 watts. Asi se descentraliza una red.

---

## Alternativas: otras formas de correr un nodo

No tienes Raspberry Pi? Hay otras opciones. Las dividimos en dos categorias:

### Self-hosting (tu hardware, en tu casa)

Esta es la opcion mas descentralizada. El nodo corre en hardware que tu controlas, en tu red. Nadie puede apagarlo excepto tu.

| Opcion | Precio aprox. | Consumo | Arquitectura | Notas |
|--------|--------------|---------|--------------|-------|
| **Raspberry Pi 5** | ~70 EUR | ~15W | ARM64 (aarch64) | La que usamos en este taller |
| **Raspberry Pi 4** | ~50 EUR | ~10W | ARM64 (aarch64) | Mas lenta pero funcional |
| **Orange Pi 5** | ~60 EUR | ~10W | ARM64 (aarch64) | Alternativa china, buen rendimiento |
| **Rock Pi 4** | ~55 EUR | ~10W | ARM64 (aarch64) | Otra alternativa ARM |
| **Mini PC x86** (ej: Beelink, MinisForum) | ~100-150 EUR | ~15-30W | x86_64 | Mas potente, usa el binario `linux-x86_64` |
| **Laptop/PC viejo** | Gratis (ya lo tienes) | ~50-100W | x86_64 | Cualquier PC con Linux sirve |

Para todas las opciones ARM64 los pasos son practicamente iguales a este taller (mismo binario `aarch64`). Para x86_64 solo cambia el archivo que descargas:

```bash
# En vez de aarch64, descargas x86_64
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-node-linux-x86_64-0.2.1.tar.gz
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-circuits-v0.4.1-linux-x86_64.tar.gz
```

El resto (init, ejecutar, verificar) es identico.

> **Por que self-hosting?** Porque la gracia de una red descentralizada es que los nodos esten distribuidos en muchos lugares, controlados por muchas personas. Si todos corren nodos en el mismo datacenter de Amazon, no es realmente descentralizado — Amazon podria apagarlos todos a la vez.

### VPS / Nube (servidor alquilado)

Si no tienes hardware o necesitas un nodo siempre encendido sin depender de tu electricidad, puedes alquilar un servidor virtual (VPS). Es menos descentralizado (dependes de un proveedor), pero es facil y rapido.

| Proveedor | Precio aprox. | Notas |
|-----------|--------------|-------|
| **Hetzner** | ~4-5 EUR/mes | Buena relacion precio/rendimiento, servidores en Europa |
| **DigitalOcean** | ~6 USD/mes | Droplet basico, facil de usar |
| **OVH** | ~3-5 EUR/mes | Economico, servidores en Europa |
| **Contabo** | ~4-5 EUR/mes | Mucho almacenamiento por poco precio |

**Pasos resumidos para VPS:**

1. Crear un VPS con Ubuntu/Debian (minimo 2GB RAM, 20GB disco)
2. Conectarte por SSH: `ssh root@IP-DE-TU-VPS`
3. Seguir este taller desde el **Paso 3** (descargar binarios `x86_64`, init, ejecutar)

```bash
# Desde tu ordenador
ssh root@123.456.789.10

# Ya dentro del VPS (mismo proceso que en la Pi)
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-node-linux-x86_64-0.2.1.tar.gz
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-circuits-v0.4.1-linux-x86_64.tar.gz
# ... etc (mismos pasos de extraccion, init y ejecucion)
```

> **Consideracion sobre descentralizacion:** Un nodo en la nube sigue sumando a la red — es mejor que no tener nodo. Pero si puedes, prioriza self-hosting. La diversidad de ubicaciones y proveedores es lo que hace fuerte a una red descentralizada.

---

## Enlaces

- [Logos Blockchain GitHub](https://github.com/logos-blockchain/logos-blockchain)
- [Release v0.2.1](https://github.com/logos-blockchain/logos-blockchain/releases/tag/0.2.1)
- [Devnet Dashboard](https://devnet.blockchain.logos.co/web/)
- [Discord de Logos](https://discord.com/channels/973324189794697286/1468535289604735038)
- [fruteroclub](https://fruteroclub.com)
