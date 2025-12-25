# 🚀 BACKLOG EXPANSIÓN - ECOin Wallet + Docker Network

**Fecha**: 2025-12-25  
**Prerequisito**: Completar SESION-BACKLOG.md (hackathon principal)  
**Objetivo**: Levantar ECOin wallet en Docker, vincular con Oasis, backup credenciales

---

## 🏗️ ARQUITECTURA DEL SISTEMA

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            HOST WINDOWS                                      │
│  ┌─────────────────────────────────────────────────────────────────────────┐ │
│  │                        Docker Desktop                                    │ │
│  │  ┌─────────────────────────────────────────────────────────────────────┐│ │
│  │  │              docker network: oasis-network (bridge)                 ││ │
│  │  │                                                                     ││ │
│  │  │   ┌─────────────────────┐       ┌─────────────────────┐            ││ │
│  │  │   │   oasis-server-dev  │       │    ecoin-wallet     │            ││ │
│  │  │   │   ─────────────────│       │   ─────────────────│            ││ │
│  │  │   │   Debian bookworm   │       │   Debian bookworm   │            ││ │
│  │  │   │   Node.js 20.x      │       │   ecoind + ecoin-qt │            ││ │
│  │  │   │   Oasis v0.6.3      │◄─────►│   RPC :7474         │            ││ │
│  │  │   │   :3000 (web)       │  RPC  │   P2P :12000        │            ││ │
│  │  │   │   :8008 (SSB)       │       │   ~/.ecoin/         │            ││ │
│  │  │   └─────────────────────┘       └─────────────────────┘            ││ │
│  │  │            │                              │                         ││ │
│  │  └────────────┼──────────────────────────────┼─────────────────────────┘│ │
│  │               │                              │                          │ │
│  └───────────────┼──────────────────────────────┼──────────────────────────┘ │
│                  │                              │                            │
│    ┌─────────────┴──────────────┐   ┌──────────┴─────────────┐              │
│    │     localhost:3000         │   │    localhost:7474      │              │
│    │     (Oasis Web UI)         │   │    (ECOin RPC)         │              │
│    │     http://localhost:3000  │   │    Wallet Settings     │              │
│    └────────────────────────────┘   └────────────────────────┘              │
│                                                                              │
│    ┌────────────────────────────────────────────────────────────────────┐   │
│    │                    volumes-dev/ (bind mounts)                       │   │
│    │  ├── ssb-data/     → /home/oasis/.ssb       (Oasis identidad)      │   │
│    │  ├── ai-models/    → /app/models            (AI LLM)               │   │
│    │  ├── ecoin-data/   → /home/ecoin/.ecoin     (ECOin wallet)         │   │
│    │  └── logs/         → /var/log/oasis         (logs)                  │   │
│    └────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔗 COMUNICACIÓN ENTRE CONTENEDORES

| Origen | Destino | Puerto | Protocolo | Uso |
|--------|---------|--------|-----------|-----|
| Oasis → ECOin | ecoin-wallet:7474 | 7474 | JSON-RPC | Consultar balance, enviar transacciones |
| Host → Oasis | localhost:3000 | 3000 | HTTP | UI Web Oasis |
| Host → Oasis | localhost:8008 | 8008 | SSB | Protocolo Scuttlebutt |
| Host → ECOin | localhost:7474 | 7474 | JSON-RPC | Debug wallet RPC |
| ECOin → Internet | 46.163.118.220 | 12000 | P2P | Red ECOin (peers) |

### Configuración de red Docker:

```yaml
# docker-compose.yml (fragmento)
networks:
  oasis-network:
    driver: bridge
    ipam:
      config:
        - subnet: 172.28.0.0/16

services:
  oasis-server:
    networks:
      oasis-network:
        aliases:
          - oasis
    
  ecoin-wallet:
    networks:
      oasis-network:
        aliases:
          - ecoin
```

### Configuración Oasis → ECOin:

```json
// src/configs/oasis-config.json
{
  "wallet": {
    "url": "http://ecoin-wallet:7474",  // ← nombre del contenedor en la red
    "user": "ecoinrpc",
    "pass": "ecoinrpc",
    "fee": "5"
  }
}
```

---

## 📋 BACKLOG EXPANSIÓN

| ID | Estado | Tarea | Notas |
|----|--------|-------|-------|
| E1 | ✅ COMPLETADO | **Crear plan de arquitectura** | Diagrama y comunicación documentados |
| E2 | ✅ COMPLETADO | Crear Dockerfile para ECOin | ECOIN_DOCKERIZE/Dockerfile |
| E3 | ✅ COMPLETADO | Crear volumen ecoin-data | volumes-dev/ecoin-data/ creado |
| E4 | ✅ COMPLETADO | Actualizar docker-compose.yml | Servicio ecoin-wallet añadido |
| E5 | ✅ COMPLETADO | Configurar docker network | oasis-network bridge configurada |
| E6 | ⏳ PENDIENTE | Build y deploy ECOin | **⚠️ REQUIERE TIEMPO** |
| E7 | ⏳ PENDIENTE | Sincronizar blockchain | bootstrap.dat + peers |
| E8 | ⏳ PENDIENTE | Generar wallet address | Primera dirección ECOin |
| E9 | ⏳ PENDIENTE | Configurar Oasis → ECOin | Settings → Wallet |
| E10 | ⏳ PENDIENTE | Backup credenciales ECOin | wallet.dat + ecoin.conf |
| E11 | ⏳ PENDIENTE | Verificar integración | Balance en Oasis UI |

---

## ⚠️ ADVERTENCIA: BUILD LARGO

> **🕐 TIEMPO ESTIMADO DE BUILD: 15-30 minutos**
>
> La compilación de ECOin desde fuente incluye:
> 1. Descarga del repositorio (~50MB)
> 2. Compilación de Boost 1.68 (5-10 min)
> 3. Compilación de LevelDB (1-2 min)
> 4. Compilación de ecoind (5-15 min)
>
> **Recomendación**: Ejecutar build en background y continuar con otras tareas.

### Comando para build:

```bash
# Build en foreground (ver progreso)
docker-compose build ecoin-wallet

# Build en background
docker-compose build ecoin-wallet &

# Ver logs del build
docker-compose logs -f ecoin-wallet
```

---

## 📝 ESTADO ACTUAL (Checkpoint)

**Fecha**: 2025-12-25  
**Commit pendiente**: `feat: add ECOin wallet Docker infrastructure`

### ✅ COMPLETADO EN ESTA SESIÓN:

1. **SESION-BACKLOG.md** - Cerrado con resumen final del hackathon
2. **SESION-BACKLOG-EXPANSION.md** - Creado con plan de arquitectura
3. **ECOIN_DOCKERIZE/Dockerfile** - Dockerfile completo para compilar ecoind
4. **ECOIN_DOCKERIZE/ecoin.conf** - Configuración RPC para comunicación con Oasis
5. **ECOIN_DOCKERIZE/docker-entrypoint.sh** - Script de inicialización
6. **docker-compose.yml** - Actualizado con servicio ecoin-wallet y red compartida
7. **volumes-dev/ecoin-data/** - Directorio creado para persistencia

### ⏳ PENDIENTE PARA PRÓXIMA SESIÓN:

1. Ejecutar `docker-compose build ecoin-wallet` (15-30 min)
2. Levantar contenedor ECOin
3. Esperar sincronización blockchain (puede tardar horas)
4. Generar primera dirección wallet
5. Configurar Oasis → ECOin en Settings → Wallet
6. Backup de wallet.dat
7. Verificar integración completa

### 🔧 ARCHIVOS CREADOS/MODIFICADOS:

```
alephscript-network-sdk/
├── SESION-BACKLOG.md           # ✏️ Actualizado (cierre hackathon)
├── SESION-BACKLOG-EXPANSION.md # 🆕 Creado
├── docker-compose.yml          # ✏️ Actualizado (+ecoin-wallet)
├── ECOIN_DOCKERIZE/            # 🆕 Carpeta nueva
│   ├── Dockerfile              # 🆕 Build ecoind desde fuente
│   ├── ecoin.conf              # 🆕 Config RPC
│   └── docker-entrypoint.sh    # 🆕 Entrypoint script
└── volumes-dev/
    └── ecoin-data/             # 🆕 Directorio creado
```

---

## 🐳 E2: DOCKERFILE PARA ECOIN

### Análisis de dependencias (de ecoin.md):

```bash
# Dependencias build
sudo apt-get install build-essential libssl-dev libssl3 \
    libdb5.3-dev libdb5.3++-dev libleveldb-dev \
    miniupnpc libminiupnpc-dev

# Dependencias Qt5 (para ecoin-qt GUI - opcional en Docker)
sudo apt-get install qt5-qmake qtbase5-dev

# Boost viene incluido en src/boost_1_68_0
```

### Dockerfile propuesto:

```dockerfile
# ECOIN_DOCKERIZE/Dockerfile
FROM debian:bookworm-slim

LABEL maintainer="alephscript"
LABEL description="ECOin P2P Cryptocurrency Wallet"
LABEL version="0.3"

# Instalar dependencias de compilación
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libssl-dev \
    libssl3 \
    libdb5.3-dev \
    libdb5.3++-dev \
    libleveldb-dev \
    miniupnpc \
    libminiupnpc-dev \
    git \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

# Crear usuario ecoin
RUN useradd -m -s /bin/bash ecoin

# Clonar repositorio
WORKDIR /build
RUN git clone --depth 1 https://github.com/epsylon/ecoin.git

# Compilar ecoind (servidor/daemon)
WORKDIR /build/ecoin/ecoin/src
RUN make -f makefile.linux USE_UPNP=- USE_IPV6=- -j$(nproc) \
    && strip ecoind \
    && mv ecoind /usr/local/bin/

# Limpiar build
WORKDIR /
RUN rm -rf /build

# Crear directorios
RUN mkdir -p /home/ecoin/.ecoin \
    && chown -R ecoin:ecoin /home/ecoin

# Copiar configuración base
COPY ecoin.conf /home/ecoin/.ecoin/ecoin.conf
RUN chown ecoin:ecoin /home/ecoin/.ecoin/ecoin.conf

# Cambiar a usuario ecoin
USER ecoin
WORKDIR /home/ecoin

# Puerto RPC y P2P
EXPOSE 7474 12000

# Volumen para datos persistentes
VOLUME ["/home/ecoin/.ecoin"]

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s \
    CMD ecoind getinfo || exit 1

# Comando por defecto
CMD ["ecoind", "-printtoconsole"]
```

### ecoin.conf para Docker:

```ini
# ECOIN_DOCKERIZE/ecoin.conf
# Configuración ECOin para Docker

# RPC - Comunicación con Oasis
rpcuser=ecoinrpc
rpcpassword=ecoinrpc
rpcport=7474
rpcallowip=172.28.0.0/16

# Server mode
server=1
daemon=0
listen=1
noirc=1

# Peers conocidos (red ECOin)
addnode=46.163.118.220
addnode=82.223.99.61
addnode=5.253.247.48
addnode=primoroso.laenre.net
addnode=alzamoreno.myasustor.com
addnode=ecoin.hacksito.com
addnode=ecoin0.vps.webdock.cloud
addnode=ecoin1.vps.webdock.cloud
addnode=ecoin3.vps.webdock.cloud
addnode=ecoin4.vps.webdock.cloud
```

---

## 🔧 E4: ACTUALIZACIÓN DOCKER-COMPOSE.YML

### Cambios propuestos:

```yaml
# docker-compose.yml (versión expandida)
services:
  oasis-server:
    # ... configuración existente ...
    networks:
      - oasis-network
    environment:
      - ECOIN_RPC_URL=http://ecoin-wallet:7474
      - ECOIN_RPC_USER=ecoinrpc
      - ECOIN_RPC_PASS=ecoinrpc
    depends_on:
      ecoin-wallet:
        condition: service_healthy

  ecoin-wallet:
    build:
      context: ./ECOIN_DOCKERIZE
      dockerfile: Dockerfile
    container_name: ecoin-wallet
    restart: unless-stopped
    networks:
      - oasis-network
    ports:
      - "7474:7474"    # RPC (para debug desde host)
      - "12000:12000"  # P2P ECOin network
    volumes:
      - ./volumes-dev/ecoin-data:/home/ecoin/.ecoin
    healthcheck:
      test: ["CMD", "ecoind", "getinfo"]
      interval: 30s
      timeout: 10s
      retries: 5
      start_period: 120s  # blockchain sync puede tardar

networks:
  oasis-network:
    driver: bridge
```

---

## 📦 E3: ESTRUCTURA DE VOLÚMENES

```bash
volumes-dev/
├── ssb-data/           # Oasis SSB (existente)
│   ├── secret          # Clave privada SSB
│   ├── config          # Config SSB
│   └── gossip.json     # Peers SSB
├── ai-models/          # Modelos AI (existente)
│   └── oasis-42-1-chat.Q4_K_M.gguf
├── ecoin-data/         # ← NUEVO: ECOin wallet
│   ├── wallet.dat      # 🔴 CRÍTICO - Clave privada ECOin
│   ├── ecoin.conf      # Configuración
│   ├── blkindex.dat    # Índice blockchain
│   ├── blk0001.dat     # Datos blockchain
│   └── debug.log       # Logs
└── logs/               # Logs (existente)
```

---

## 🔐 E10: PROTOCOLO BACKUP ECOIN

### Archivos críticos:

| Archivo | Prioridad | Descripción |
|---------|-----------|-------------|
| `wallet.dat` | 🔴 CRÍTICO | Contiene claves privadas ECOin |
| `ecoin.conf` | 🟡 Importante | Configuración (user/pass RPC) |

### Comando backup:

```bash
# Desde host Windows (Git Bash)
mkdir -p /c/Users/aleph/OASIS/ALEPHLUCAS_WALLET_OASIS/backup-ecoin

# Copiar wallet.dat (PARAR CONTENEDOR PRIMERO)
docker stop ecoin-wallet
cp ./volumes-dev/ecoin-data/wallet.dat \
   /c/Users/aleph/OASIS/ALEPHLUCAS_WALLET_OASIS/backup-ecoin/

# Hash para verificación
sha256sum /c/Users/aleph/OASIS/ALEPHLUCAS_WALLET_OASIS/backup-ecoin/wallet.dat

# Reiniciar
docker start ecoin-wallet
```

---

## 🎯 ORDEN DE EJECUCIÓN

```
E1 (Plan) ─► E2 (Dockerfile) ─► E3 (Volumen) ─► E4 (Compose) ─► E5 (Network)
                                                                    │
                                                                    ▼
E11 (Verificar) ◄─ E10 (Backup) ◄─ E9 (Config Oasis) ◄─ E8 (Address) ◄─ E6+E7 (Build+Sync)
```

---

## 🤖 INSTRUCCIONES PARA AGENTES

### Herramientas MCP a usar:

| Tarea | Herramienta |
|-------|-------------|
| Crear archivos Docker | `create_file` |
| Build imagen | `run_in_terminal` → `docker-compose build` |
| Verificar contenedores | `mcp_copilot_conta_list_containers` |
| Ver logs ECOin | `mcp_copilot_conta_logs_for_container` |
| Configurar Oasis UI | `mcp_playwright_browser_*` |
| Backup wallet | `run_in_terminal` → `docker cp` |

### Comandos útiles:

```bash
# Build ECOin
docker-compose build ecoin-wallet

# Levantar solo ECOin
docker-compose up -d ecoin-wallet

# Ver logs sync blockchain
docker logs -f ecoin-wallet

# Verificar RPC funciona
docker exec ecoin-wallet ecoind getinfo

# Obtener nueva dirección
docker exec ecoin-wallet ecoind getnewaddress ""

# Ver balance
docker exec ecoin-wallet ecoind getbalance
```

---

## 📊 ESTADO ACTUAL

| Componente | Estado |
|------------|--------|
| Plan arquitectura | ✅ Documentado |
| Dockerfile ECOin | ✅ Creado |
| docker-compose actualizado | ✅ Modificado |
| Red oasis-network | ✅ Configurada |
| ECOin corriendo | ⏳ Pendiente build |
| Integración Oasis | ⏳ Pendiente |
| Backup ECOin | ⏳ Pendiente |

---

## 🔄 PRÓXIMOS PASOS (Siguiente sesión)

```bash
# 1. Build de ECOin (15-30 min)
cd /c/Users/aleph/OASIS/alephscript-network-sdk
docker-compose build ecoin-wallet

# 2. Levantar contenedor
docker-compose up -d ecoin-wallet

# 3. Verificar logs (sincronización blockchain)
docker logs -f ecoin-wallet

# 4. Obtener dirección wallet
docker exec ecoin-wallet ecoind getnewaddress ""

# 5. Configurar en Oasis UI
# → http://localhost:3000/settings
# → Wallet section
# → URL: http://ecoin-wallet:7474
# → User: ecoinrpc
# → Pass: ecoinrpc
```

---

**SIGUIENTE PASO**: Ejecutar build y continuar con E6-E11
