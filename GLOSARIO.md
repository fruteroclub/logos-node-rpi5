# Glosario de terminos

Todos los conceptos tecnicos que aparecen en este taller, explicados de forma sencilla.

---

## Hardware

### ARM
Tipo de arquitectura de procesador. A diferencia de los procesadores Intel/AMD de tu laptop (x86), los ARM son mas eficientes en consumo de energia. Tu telefono, tablets y la Raspberry Pi usan ARM. La Pi 5 usa ARM de 64 bits, que tecnicamente se llama **aarch64** o **arm64** — por eso descargamos binarios con ese nombre.

### Raspberry Pi 5
Un ordenador completo del tamano de una tarjeta de credito. Tiene procesador, memoria RAM, puertos USB, ethernet, HDMI, y ranura para microSD. Cuesta ~70 euros y consume menos de 15 watts (como un foco LED). Ideal para tener un servidor en casa que corre 24/7 sin arruinarte en electricidad.

### CPU (Central Processing Unit)
El "cerebro" del ordenador. Es el chip que ejecuta las instrucciones de los programas. Cuando decimos que un nodo "usa mucha CPU", significa que esta haciendo muchos calculos. En la Pi 5 es un BCM2712 con 4 nucleos Cortex-A76.

### Memoria RAM
Memoria temporal y rapida donde el ordenador guarda lo que esta usando en el momento. Piensa en ella como el escritorio donde trabajas: cuanto mas grande, mas cosas puedes tener abiertas a la vez. La Pi 5 tiene 4GB u 8GB. Cuando apagas, se borra.

### Tarjeta microSD
La tarjeta de memoria pequena donde esta instalado el sistema operativo de la Pi. Es su "disco duro". Aqui vive todo: el OS, tus programas, tus archivos. Es la misma tarjeta que usan las camaras y algunos telefonos.

### Cable ethernet
El cable de red fisico (el que parece un telefono gordo con conector transparente RJ45). Conecta tu Pi al router para darle internet. Es mas estable y rapido que WiFi, ideal para un nodo que necesita estar conectado siempre.

### Router
La caja que te da tu proveedor de internet (Movistar, Orange, etc). Conecta tu casa a internet y crea una red local entre tus dispositivos. Cuando conectas la Pi por ethernet al router, la Pi entra en tu red local y puede acceder a internet.

### GPIO (General Purpose Input/Output)
Los pines metalicos que sobresalen de la Raspberry Pi (40 pines en fila). Sirven para conectar sensores, LEDs, motores, pantallas, etc. No los usamos en este taller, pero es lo que hace a la Pi popular para proyectos de electronica y domotica.

### Overclocking
Hacer que el procesador trabaje mas rapido de lo que viene configurado de fabrica. Es como subir las revoluciones de un motor. Da mas rendimiento pero genera mas calor y puede ser inestable. Se configura en `config.txt` de la Pi. No lo necesitamos para el nodo.

### USB-C
El conector de carga de la Pi 5 (el mismo que usan la mayoria de telefonos modernos). La Pi necesita un cargador de al menos 5V/3A para funcionar correctamente.

---

## Software y sistema operativo

### OS (Operating System / Sistema Operativo)
El programa base que controla todo el hardware y permite ejecutar otros programas. Windows, macOS y Linux son sistemas operativos. La Pi usa **Raspberry Pi OS**, que esta basado en Debian Linux.

### Linux
Un sistema operativo libre y gratuito. La mayoria de servidores en internet corren Linux. Raspberry Pi OS es una version de Linux adaptada para la Pi. Cuando usas la terminal de la Pi, estas usando Linux.

### Debian
Una distribucion (version) de Linux conocida por ser estable y confiable. Raspberry Pi OS esta basada en Debian. Tu ordenador (el que usas para conectarte a la Pi) tambien usa Debian.

### Terminal / Linea de comandos
La ventana negra donde escribes comandos. En vez de hacer clic en botones, le dices al ordenador que hacer escribiendo texto. Es mas poderosa que la interfaz grafica y la unica forma de controlar la Pi remotamente por SSH.

### Kernel
El nucleo del sistema operativo. Es el intermediario entre los programas y el hardware. Cuando el kernel arranca, activa el procesador, la memoria, los puertos USB, la red, etc. En la Pi, el kernel esta en la particion de boot como `kernel_2712.img`.

### Firmware
Software de muy bajo nivel que viene integrado en el hardware. La Pi tiene firmware de Broadcom que inicializa el chip antes de que Linux arranque. Los archivos `.elf` y `.dat` en la particion de boot son firmware.

### Particion
Una division logica de un disco o tarjeta SD. Es como dividir un estante en secciones. La SD de la Pi tiene dos particiones:
- **bootfs** — particion de arranque (pequena, 512MB)
- **rootfs** — particion raiz con todo el sistema operativo

### FAT32
Un formato (sistema de archivos) para almacenar datos en un disco. Es antiguo pero universal: Windows, Mac y Linux pueden leerlo. La particion de boot de la Pi usa FAT32 para que cualquier ordenador pueda modificarla (por eso pudimos crear el archivo `ssh` desde Debian).

### ext4
El formato de la particion rootfs. Es el sistema de archivos estandar de Linux. Mas eficiente y seguro que FAT32, pero Windows no puede leerlo directamente.

### Montar / Desmontar
"Montar" significa hacer accesible un dispositivo de almacenamiento (SD, USB, disco) para que puedas leer y escribir archivos. "Desmontar" (`umount`) es lo opuesto: decirle al sistema que termine de escribir todo y libere el dispositivo. Siempre desmonta antes de sacar una SD para evitar corrupcion de datos.

---

## Redes

### IP (Internet Protocol) / Direccion IP
Un numero que identifica a cada dispositivo en una red. Es como la direccion postal de tu ordenador. En tu red local son numeros tipo `192.168.1.69`. Tu Pi recibio una automaticamente al conectarse al router.

### DHCP (Dynamic Host Configuration Protocol)
El mecanismo por el cual tu router asigna IPs automaticamente a los dispositivos que se conectan. Cuando enchufaste la Pi al router, el DHCP le asigno la IP `192.168.1.69` sin que tuvieras que configurar nada.

### NAT (Network Address Translation)
Tu router tiene una sola IP publica (la que ve internet), pero muchos dispositivos en casa. NAT es el truco que usa el router para que todos compartan esa IP publica. Por eso el nodo necesita `--no-public-ip-check`: la Pi no tiene IP publica propia, esta "escondida" detras del router.

### SSH (Secure Shell)
Protocolo para conectarte a otro ordenador de forma remota y segura. Todo lo que escribes y recibes viaja encriptado por la red. Es como hacer una llamada telefonica al otro ordenador, pero en vez de hablar, escribes comandos.

### mDNS (Multicast DNS)
Lo que permite encontrar dispositivos por nombre (como `raspberrypi.local`) en vez de por IP. La Pi tiene un servicio llamado Avahi que anuncia su nombre en la red local. Por eso `ping raspberrypi.local` funciona sin configurar nada.

### nmap
Una herramienta para escanear redes. Cuando la usas con `nmap -sn 192.168.1.0/24`, le pides que busque todos los dispositivos encendidos en tu red local. Util cuando `raspberrypi.local` no funciona y necesitas encontrar la IP de la Pi.

### Puerto
Un numero que identifica un servicio especifico dentro de un ordenador. Piensa en la IP como la direccion de un edificio y el puerto como el numero de oficina. SSH usa el puerto 22. El nodo de Logos usa el puerto 8080 para su API.

### Fingerprint (huella digital SSH)
La primera vez que te conectas a un servidor por SSH, te muestra un "fingerprint" — una huella unica del servidor. Es para que verifiques que te estas conectando al servidor correcto y no a un impostor. Escribes `yes` para aceptarlo y se guarda para futuras conexiones.

---

## Comandos de terminal

### sudo
"Super User DO" — ejecuta un comando como administrador. Algunos comandos necesitan permisos especiales (instalar programas, modificar archivos del sistema, montar discos). Antepones `sudo` al comando y te pide tu contrasena.

### apt
El gestor de paquetes de Debian/Ubuntu. Es como una tienda de aplicaciones desde la terminal. `apt update` actualiza la lista de programas disponibles, `apt upgrade` instala las actualizaciones, `apt install` instala programas nuevos.

### wget
Programa para descargar archivos desde internet por terminal. Le das una URL y descarga el archivo al directorio donde estes. Basicamente un "clic derecho > guardar como" pero desde la terminal.

### tar
Programa para empaquetar y descomprimir archivos. `tar -xzf archivo.tar.gz` descomprime un archivo comprimido. Las flags significan: `x` extraer, `z` descomprimir gzip, `f` el archivo que sigue.

### touch
Crea un archivo vacio. `touch ssh` crea un archivo llamado `ssh` sin contenido. Lo usamos porque la Pi solo necesita que el archivo exista, no importa que tenga dentro.

### lsblk
"List Block devices" — muestra todos los dispositivos de almacenamiento conectados (discos, SDs, USBs) y sus particiones. Util para ver que esta conectado y donde esta montado.

### ping
Envia un "hola, estas ahi?" a otro dispositivo en la red. Si responde, sabes que esta encendido y conectado. Muestra el tiempo que tarda en responder (en milisegundos).

### curl
Herramienta para hacer peticiones HTTP desde la terminal. La usamos para consultar la API del nodo de Logos (`curl http://localhost:8080/...`). Es como visitar una URL en el navegador, pero desde la terminal y solo ves los datos en texto.

### grep
Busca texto dentro de archivos. `grep -A3 known_keys config.yaml` busca la linea que contiene "known_keys" y muestra 3 lineas despues. Muy util para encontrar informacion especifica en archivos de configuracion.

### screen
Programa que crea sesiones de terminal persistentes. Cuando te conectas por SSH y cierras la conexion, todo lo que estaba corriendo se detiene. Con `screen`, el programa sigue corriendo aunque cierres SSH. Puedes volver a esa sesion despues con `screen -r`.

### nohup
"No Hang Up" — ejecuta un programa de forma que no se detenga cuando cierras la terminal. Similar a `screen` pero mas sencillo: `nohup ./programa &` lanza el programa en segundo plano y lo desconecta de tu sesion SSH.

### htop
Un monitor de sistema interactivo. Muestra en tiempo real cuanta CPU, RAM y procesos estan corriendo. Es como el "Administrador de tareas" de Windows pero en la terminal. Util para ver cuantos recursos consume el nodo.

### passwd
Cambia la contrasena del usuario actual. Te pide la contrasena actual y luego la nueva (dos veces). Importante hacerlo despues del primer login para no dejar la contrasena por defecto.

---

## Blockchain y Logos Network

### Blockchain
Una base de datos distribuida donde la informacion se guarda en "bloques" encadenados. Cada bloque contiene transacciones y un enlace al bloque anterior, formando una cadena. Nadie controla la cadena: todos los nodos tienen una copia y se ponen de acuerdo sobre que es valido.

### Nodo
Un ordenador que participa en una red blockchain. Tu Pi corriendo el software de Logos es un nodo. Los nodos se comunican entre ellos, validan transacciones, mantienen una copia de la cadena, y pueden producir bloques nuevos.

### Produccion de bloques
El acto de crear un nuevo bloque y anadirlo a la cadena. En Logos, los nodos que tienen fondos en stake pueden ser seleccionados para producir bloques. Es como ser elegido para anadir la siguiente pagina al libro compartido.

### Peer
Otro nodo en la red. Cuando inicializamos el nodo con `-p`, le damos las "direcciones" de otros nodos conocidos para que pueda encontrar la red. Los peers comparten bloques y transacciones entre si.

### Devnet (Development Network)
Una red de pruebas para desarrolladores. No es la red principal (mainnet). Los tokens no tienen valor real. Sirve para probar que todo funciona antes de lanzar a produccion. Es como un ensayo general antes del estreno.

### Wallet (Cartera)
Un par de claves criptograficas (publica y privada) que te permiten recibir y enviar tokens. La clave publica es como tu numero de cuenta (la compartes para que te envien fondos). La clave privada es como tu PIN (nunca la compartas). El nodo de Logos genera tu wallet automaticamente al inicializarse.

### Clave de wallet
La clave publica de tu wallet. Es el identificador unico de tu cuenta en la red. La necesitas para pedir fondos del faucet y para consultar tu balance.

### Balance
La cantidad de tokens que tiene tu wallet. Se consulta a traves de la API del nodo. En el devnet son tokens de prueba sin valor real.

### Faucet (Grifo)
Un servicio web que regala tokens de prueba gratis. Solo existe en redes de desarrollo (devnet/testnet). Le das tu clave de wallet y te envia tokens para que puedas probar el sistema sin gastar dinero real.

### Stake / Staking
Depositar tokens como "garantia" para participar en el consenso de la red. Si tienes tokens en stake, el protocolo puede seleccionarte para producir bloques. Es como poner una fianza que demuestra tu compromiso con la red.

### Epoch
Un periodo de tiempo definido por el protocolo. En Logos, despues de dos epochs (~3.5 horas), un nodo con fondos en stake empieza a ser elegible para producir bloques. Piensa en un epoch como un "turno" o "ronda" del consenso.

### Consenso (Cryptarchia)
El mecanismo por el cual todos los nodos se ponen de acuerdo sobre que bloques son validos. Logos usa un sistema llamado **Cryptarchia**, que es un tipo de Proof of Stake (prueba de participacion): en vez de gastar energia minando (como Bitcoin), los nodos con tokens en stake votan sobre la validez de los bloques.

### Proof of Stake (PoS)
Un tipo de consenso donde los validadores son seleccionados segun cuantos tokens tienen en stake, no segun cuanta energia gastan. Es mucho mas eficiente energeticamente que Proof of Work (Bitcoin). Tu Pi puede participar porque no necesita hardware potente.

### Zero-Knowledge Proofs (Pruebas de conocimiento cero)
Una tecnica criptografica que permite demostrar que algo es verdadero sin revelar la informacion en si. Ejemplo: puedes demostrar que tienes mas de 18 anos sin mostrar tu fecha de nacimiento. Logos usa esto para que las transacciones sean privadas. Los "circuitos ZK" que descargamos son las reglas matematicas para crear y verificar estas pruebas.

### Bootstrapping
El estado inicial del nodo cuando acaba de arrancar. Esta descargando y sincronizando la cadena de bloques desde los peers. Es como ponerse al dia con toda la historia antes de poder participar. Despues pasa al estado "Online".

### API (Application Programming Interface)
Una forma de comunicarse con un programa mediante peticiones HTTP. El nodo de Logos expone una API en el puerto 8080 que te permite consultar el estado de la cadena, tu balance, y mas. Cuando haces `curl http://localhost:8080/...`, estas usando la API.

### JSON
Un formato de texto para representar datos estructurados. Cuando consultas la API del nodo, te responde en JSON. Se ve algo asi: `{"status": "online", "height": 1234}`. Es el "idioma" universal para que programas se comuniquen entre si.

---

## Seguridad

### Contrasena por defecto
La contrasena que viene preconfigurada en un sistema nuevo. En la Pi es `raspberry`. Es un riesgo de seguridad enorme porque todo el mundo la conoce. Cambiarla con `passwd` es lo primero que debes hacer.

### Encriptacion
Proceso de convertir informacion legible en un codigo secreto. SSH encripta todo lo que envias entre tu ordenador y la Pi. Aunque alguien intercepte los datos, no puede leerlos. Es como hablar en un idioma secreto que solo tu y la Pi entienden.

### rfkill
Una herramienta de Linux que puede bloquear dispositivos inalambricos (WiFi, Bluetooth). El mensaje "Wi-Fi is currently blocked by rfkill" que vimos al conectarnos significa que el WiFi esta desactivado. No nos afecta porque usamos cable ethernet.
