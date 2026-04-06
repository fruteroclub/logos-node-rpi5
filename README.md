# Logos Blockchain Node en Raspberry Pi 5

Taller de [fruteroclub](https://fruteroclub.com) para instalar y correr un nodo de [Logos Network](https://logos.co/) en una Raspberry Pi 5.

## Requisitos

- Raspberry Pi 5 con Raspberry Pi OS (64-bit)
- Tarjeta SD con el OS flasheado
- Cable ethernet
- Ordenador con Linux/Mac para controlar la Pi por SSH
- Conexion a internet

## Paso 1 — Preparar la SD desde tu ordenador

Antes de encender la Pi, montamos la SD en nuestro ordenador para habilitar SSH.

### 1.1 Insertar la SD y verificar que se monta

```bash
lsblk
```

Deberiamos ver algo asi:

```
mmcblk0     179:0    0   117G  0 disk
├─mmcblk0p1 179:1    0   512M  0 part /media/tu-usuario/bootfs
└─mmcblk0p2 179:2    0 116.5G  0 part /media/tu-usuario/rootfs
```

- `bootfs` = particion de arranque (config, kernel)
- `rootfs` = sistema de archivos raiz (el OS completo)

### 1.2 Habilitar SSH

Raspberry Pi OS busca un archivo llamado `ssh` en la particion de boot. Si existe, activa el servidor SSH en el primer arranque.

```bash
sudo touch /media/tu-usuario/bootfs/ssh
```

### 1.3 Verificar el usuario

```bash
ls /media/tu-usuario/rootfs/home/
# Output: pi
```

El usuario por defecto es `pi`.

## Paso 2 — Conectar la Pi y acceder por SSH

### 2.1 Conexion fisica

1. Inserta la SD en la Raspberry Pi 5
2. Conecta el cable ethernet del router/switch a la Pi
3. Conecta la alimentacion y espera ~30 segundos

### 2.2 Encontrar la IP de la Pi

```bash
# Opcion A: por hostname (si mDNS funciona en tu red)
ping raspberrypi.local

# Opcion B: escanear la red local
ip route | grep default          # ver tu gateway (ej: 192.168.1.1)
nmap -sn 192.168.1.0/24          # escanear dispositivos
```

### 2.3 Conectar por SSH

```bash
ssh pi@raspberrypi.local
# o con IP directa:
ssh pi@192.168.1.XX
```

Contrasena por defecto: `raspberry`

> **Importante**: Cambia la contrasena despues del primer login con `passwd`

## Paso 3 — Instalar el nodo Logos (dentro de la Pi por SSH)

### 3.1 Descargar binarios

El nodo tiene binarios precompilados para ARM64 (la arquitectura de la Pi 5).

```bash
# Descargar el nodo y los circuitos ZK
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-node-linux-aarch64-0.2.1.tar.gz
wget https://github.com/logos-blockchain/logos-blockchain/releases/download/0.2.1/logos-blockchain-circuits-v0.4.1-linux-aarch64.tar.gz
```

### 3.2 Extraer e instalar

```bash
# Extraer circuitos ZK (necesarios para pruebas de conocimiento cero)
tar -xzf logos-blockchain-circuits-v0.4.1-linux-aarch64.tar.gz
mv logos-blockchain-circuits ~/.logos-blockchain-circuits

# Extraer el binario del nodo
tar -xzf logos-blockchain-node-linux-aarch64-0.2.1.tar.gz
```

### 3.3 Inicializar el nodo

Esto genera un archivo `user_config.yaml` y conecta con los peers del devnet:

```bash
./logos-blockchain-node init \
    -p /ip4/65.109.51.37/udp/3000/quic-v1/p2p/12D3KooWL7a8LBbLRYnabptHPFBCmAs49Y7cVMqvzuSdd43tAJk8 \
    -p /ip4/65.109.51.37/udp/3001/quic-v1/p2p/12D3KooWPLeAcachoUm68NXGD7tmNziZkVeMmeBS5NofyukuMRJh \
    -p /ip4/65.109.51.37/udp/3002/quic-v1/p2p/12D3KooWKFNe4gS5DcCcRUVGdMjZp3fUWu6q6gG5R846Ui1pccHD \
    -p /ip4/65.109.51.37/udp/3003/quic-v1/p2p/12D3KooWAnriLgXyQnGTYz1zPWPkQL3rthTKYLzuAP7MMnbgsxzR \
    --no-public-ip-check
```

> `--no-public-ip-check` porque la Pi esta detras de NAT (tu router)

### 3.4 Ejecutar el nodo

```bash
./logos-blockchain-node user_config.yaml
```

### 3.5 Verificar que funciona

En otra terminal SSH:

```bash
curl -w "\n" http://localhost:8080/cryptarchia/info
```

Deberia devolver info del estado del nodo (empieza en "Bootstrapping", luego pasa a "Online").

## Paso 4 — Obtener fondos y producir bloques

1. Busca tu clave de wallet:
   ```bash
   grep -A3 known_keys user_config.yaml
   ```

2. Ve al faucet del devnet: https://devnet.blockchain.logos.co/web/faucet/
   (necesitas credenciales del Discord de Logos)

3. Verifica tu balance:
   ```bash
   curl -w "\n" http://localhost:8080/wallet/TU_CLAVE/balance
   ```

4. Despues de ~3.5 horas (dos epochs), el nodo empieza a producir bloques automaticamente.

## Enlaces

- [Logos Blockchain GitHub](https://github.com/logos-blockchain/logos-blockchain)
- [Release v0.2.1](https://github.com/logos-blockchain/logos-blockchain/releases/tag/0.2.1)
- [Devnet Dashboard](https://devnet.blockchain.logos.co/web/)
- [Discord de Logos](https://discord.com/channels/973324189794697286/1468535289604735038)
- [fruteroclub](https://fruteroclub.com)
