# =============================================================================
# ECOin P2P Cryptocurrency Wallet - Dockerfile
# =============================================================================
# Descripción: Construye ecoind (daemon) desde fuente para integrar con Oasis
# Basado en: https://ecoin.03c8.net / https://github.com/epsylon/ecoin
# =============================================================================

FROM debian:bookworm-slim

LABEL maintainer="alephscript"
LABEL description="ECOin P2P Cryptocurrency Wallet Daemon"
LABEL version="0.3"
LABEL org.opencontainers.image.source="https://github.com/epsylon/ecoin"

# =============================================================================
# STAGE 1: Build dependencies
# =============================================================================

# Evitar prompts interactivos
ENV DEBIAN_FRONTEND=noninteractive

# Instalar dependencias de compilación
RUN apt-get update && apt-get install -y --no-install-recommends \
    # Build essentials
    build-essential \
    g++ \
    make \
    # OpenSSL
    libssl-dev \
    libssl3 \
    # Berkeley DB
    libdb5.3-dev \
    libdb5.3++-dev \
    # LevelDB
    libleveldb-dev \
    # UPnP (opcional pero recomendado)
    miniupnpc \
    libminiupnpc-dev \
    # Git para clonar
    git \
    # Certificados SSL
    ca-certificates \
    # Herramientas útiles
    curl \
    procps \
    && rm -rf /var/lib/apt/lists/*

# =============================================================================
# STAGE 2: Clone and build ECOin
# =============================================================================

# Crear directorio de build
WORKDIR /build

# Clonar repositorio ECOin (shallow clone para ahorrar espacio)
RUN git clone --depth 1 https://github.com/epsylon/ecoin.git

# Primero compilar LevelDB (requerido por ECOin)
WORKDIR /build/ecoin/ecoin/src/leveldb
RUN make libleveldb.a libmemenv.a -j$(nproc) || true

# Compilar ecoind (daemon sin GUI)
WORKDIR /build/ecoin/ecoin/src

# El Makefile de ECOin usa Boost incluido en src/boost_1_68_0
# Necesitamos compilar las librerías de Boost primero
WORKDIR /build/ecoin/ecoin/src/boost_1_68_0
RUN ./bootstrap.sh --prefix=/usr/local --with-libraries=system,filesystem,program_options,thread,chrono \
    && ./b2 install -j$(nproc) link=static threading=multi runtime-link=static

# Volver al src de ECOin y compilar
WORKDIR /build/ecoin/ecoin/src

# Build con opciones optimizadas para Docker (sin UPnP, sin IPv6)
RUN make -f makefile.linux USE_UPNP=- USE_IPV6=- -j$(nproc) \
    && strip ecoind \
    && mv ecoind /usr/local/bin/ \
    && chmod +x /usr/local/bin/ecoind

# Verificar que el binario existe
RUN ecoind --version || echo "Build completed"

# =============================================================================
# STAGE 3: Setup runtime environment
# =============================================================================

# Limpiar directorio de build para reducir tamaño de imagen
WORKDIR /
RUN rm -rf /build

# Crear usuario no-root para seguridad
RUN useradd -m -s /bin/bash -u 1000 ecoin

# Crear directorio de datos
RUN mkdir -p /home/ecoin/.ecoin \
    && chown -R ecoin:ecoin /home/ecoin

# Copiar configuración base
COPY --chown=ecoin:ecoin ecoin.conf /home/ecoin/.ecoin/ecoin.conf

# Copiar bootstrap.dat si existe (acelera sync inicial)
# COPY --chown=ecoin:ecoin bootstrap.dat /home/ecoin/.ecoin/bootstrap.dat

# Copiar script de entrypoint
COPY --chown=ecoin:ecoin docker-entrypoint.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/docker-entrypoint.sh

# =============================================================================
# STAGE 4: Runtime configuration
# =============================================================================

# Cambiar a usuario ecoin (no root)
USER ecoin
WORKDIR /home/ecoin

# Puertos:
# - 7474: JSON-RPC (comunicación con Oasis)
# - 12000: P2P ECOin network
EXPOSE 7474 12000

# Volumen para persistencia de datos
VOLUME ["/home/ecoin/.ecoin"]

# Health check - verifica que el daemon responde
HEALTHCHECK --interval=30s --timeout=10s --start-period=120s --retries=3 \
    CMD curl -sf --user "${RPC_USER:-ecoinrpc}:${RPC_PASS:-ecoinrpc}" \
        --data-binary '{"jsonrpc":"1.0","id":"health","method":"getinfo","params":[]}' \
        -H 'content-type: text/plain;' \
        http://127.0.0.1:7474/ || exit 1

# Entrypoint para configuración dinámica
ENTRYPOINT ["docker-entrypoint.sh"]

# Comando por defecto: ejecutar daemon en foreground
CMD ["ecoind", "-printtoconsole", "-logtimestamps"]
