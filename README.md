# 🌴 OASIS Hackathon Kit 2025

> **Guía (asistida por IA o no) para prepararse para el hackathon de OASIS**  
> *Último finde de 2025 · Organiza [SolarNET.HuB](https://solarnethub.com)*

![OASIS UI](docs/assets/Oasis_UI.png)

---

## 📅 El Plan: Antes y Durante

```
┌─────────────────────────────────────────────────────────────────────┐
│                                                                     │
│   🗓️  ANTES DEL HACKATHON           🚀 DURANTE EL HACKATHON        │
│   ─────────────────────              ─────────────────────          │
│                                                                     │
│   ✅ Clonas este repo               → Ya tienes OASIS corriendo    │
│   ✅ Levantas Docker                → Tu avatar listo en la red    │
│   ✅ Creas tu avatar                → Conectado al PUB             │
│   ✅ Te conectas al PUB             → Participas desde el minuto 0 │
│   ✅ Haces backup en USB            → Sin dramas de "perdí mi ID"  │
│                                                                     │
│   📖 Sigue HACKATON_GUIDE.md        🎯 A hackear se ha dicho       │
│                                                                     │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🏠 ¿Qué es OASIS?

**OASIS** es una red social distribuida, descentralizada, federada y realmente libre, basada en [SSB (Secure Scuttlebutt)](https://scuttlebutt.nz).

- 🔐 **Tu identidad es tuya** - Par de claves Ed25519, sin servidores centrales
- 🌐 **Mesh networking** - Funciona offline, sincroniza cuando hay conexión
- 🤖 **IA colectiva integrada** - Modelo "42" entrenado con contenido de la red
- 💰 **ECOin** - Criptomoneda interna + Renta Básica Universal
- ⚖️ **Gobernanza** - Parlamento y Cortes descentralizadas
- 🎭 **L.A.R.P.** - 1+8 casas para organización federal

> *"Una red donde tú tienes el control, no una corporación."*

---

## 🚀 Quickstart (Docker)

```bash
# 1. Clona el repo
git clone https://github.com/AcidGambit/oasis-alephscript-network-sdk.git
cd oasis-alephscript-network-sdk

# 2. Prepara volúmenes
mkdir -p volumes-dev/{ssb-data,ai-models,logs}

# 3. Build + Run
docker compose up --build -d

# 4. Accede
open http://localhost:3000
```

**Requisitos**: Docker 24+, 8GB RAM mínimo. GPU NVIDIA opcional pero recomendada para IA.

---

## 📚 Documentación

| Documento | Descripción |
|-----------|-------------|
| [HACKATON_GUIDE.md](HACKATON_GUIDE.md) | 💬 **Conversación completa** con el Agente IA - Todo el proceso paso a paso |
| [SESION-BACKLOG.md](SESION-BACKLOG.md) | ✅ Backlog de tareas de la sesión de preparación |
| [SESION-BACKLOG-EXPANSION.md](SESION-BACKLOG-EXPANSION.md) | 🔧 Expansión: ECOin wallet en Docker |
| [docs/](docs/index.html) | 🌐 Landing page para GitHub Pages |
| [GPU_SIMPLE.md](GPU_SIMPLE.md) | 🎮 Configuración de GPU para IA local |

---

## 🔗 El Ecosistema

```
                    ┌─────────────────────────┐
                    │   solarnethub.com       │
                    │   ═══════════════       │
                    │   La casa organizadora  │
                    │   del hackathon         │
                    └───────────┬─────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  OASIS App    │     │   ECOin Chain   │     │   Wiki/Docs     │
│  ───────────  │     │   ───────────   │     │   ───────────   │
│  Red social   │     │  Criptomoneda   │     │  Conocimiento   │
│  distribuida  │     │  + UBI semanal  │     │  colectivo      │
└───────────────┘     └─────────────────┘     └─────────────────┘
        │                       │                       │
        └───────────────────────┴───────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │  TÚ (tu nodo local)   │
                    │  ═══════════════════  │
                    │  Soberanía digital    │
                    └───────────────────────┘
```

**Links útiles:**
- 🏠 **Casa madre**: [solarnethub.com](https://solarnethub.com)
- 📖 **Wiki**: [wiki.solarnethub.com](https://wiki.solarnethub.com)
- 💰 **ECOin**: [ecoin.03c8.net](https://ecoin.03c8.net)
- 🔧 **Código fuente**: [solarnethub.com/git](https://solarnethub.com/git)

---

## ⚠️ BACKUP: No lo olvides

Tu identidad en la red es un archivo llamado `secret`. **Si lo pierdes, pierdes tu avatar para siempre.**

```bash
# Ubicación dentro del contenedor
/home/oasis/.ssb/secret

# Ubicación en tu máquina (con volúmenes Docker)
./volumes-dev/ssb-data/secret

# COPIA A USB EXTERNO ← HAZLO
cp ./volumes-dev/ssb-data/secret /media/TU_USB/oasis-backup/
```

El script [docker-scripts/backup-keys.sh](docker-scripts/backup-keys.sh) hace esto automáticamente con verificación SHA256.

También puedes usar la función `/legacy` integrada en OASIS para exportar todo cifrado.

---

## 🎭 Para Freaks, Groupies y Satélites

Este repositorio es un **proyecto satélite/parásito** del ecosistema OASIS, creado con la intención de:

1. **Documentar** el proceso de preparación para un hackathon
2. **Facilitar** la entrada a nuevos habitantes de la red
3. **Demostrar** que un agente de IA puede guiar (o acompañar) el proceso

*¿Eres un freak de la descentralización? ¿Un groupie del SSB? ¿Un satélite orbitando OASIS?*

**Bienvenido/a.** Este kit es para ti.

---

## 📸 Galería del Proceso

| Setup en VS Code | Docker Desktop | Docker Compose |
|------------------|----------------|----------------|
| ![IDE](docs/assets/OASIS_IDE.png) | ![Desktop](docs/assets/OASIS_DockerDesktop.png) | ![Docker](docs/assets/OASIS_DOCKER.png) |

---

## 📜 Licencia

**GNU Affero General Public License v3** - *animus iocandi*

El código de OASIS pertenece a sus creadores en [SolarNET.HuB](https://solarnethub.com).

Este repositorio es una derivación/documentación con propósitos educativos y de participación en hackathon.

> *"Si algo de aquí te sirve, compártelo. Si lo mejoras, devuélvelo a la comunidad."*

```
  ╔═══════════════════════════════════════════════════════════╗
  ║                                                           ║
  ║   Hecho con 🤖 + ☕ durante la preparación del            ║
  ║   Hackathon OASIS · Último finde de 2025                  ║
  ║                                                           ║
  ║   github.com/AcidGambit/oasis-alephscript-network-sdk     ║
  ║                                                           ║
  ╚═══════════════════════════════════════════════════════════╝
```

---

<details>
<summary>📋 README Original de OASIS (click para expandir)</summary>

## Oasis

Oasis is a decentralized social network client built with SSB technology.

### Frontend Features
- 🌍 Multi-language
- 🌚 Dark-mode design  
- 👁️ Dyslexia mode
- 🔊 Screen reader accessible
- 💬 Public posts, replies, and mentions
- 🔐 Private messages
- 🖼️ Image and audio handling
- 🌐 External link previews
- 📰 RSS feed generation

### Modules
| Module | Description |
|--------|-------------|
| activity | Network activity |
| agenda | Collective calendar |
| audios | Audio sharing |
| banking | UBI system |
| blockchain | Distributed ledger |
| cipher | Encrypted messages |
| courts | Justice system |
| cv | Resumes |
| documents | Document management |
| events | Events |
| favorites | Bookmarks |
| feed | Timeline |
| forum | Discussion forum |
| images | Image gallery |
| inhabitants | Network users |
| jobs | Job board |
| legacy | Backup/Restore |
| market | P2P marketplace |
| opinions | Opinion system |
| parliament | Governance |
| polls | Polls |
| tribes | Federated groups |

### Links
- SNH Website: https://solarnethub.com
- Documentation: https://wiki.solarnethub.com
- Code of Conduct: https://wiki.solarnethub.com/docs/code_of_conduct

</details>

---

*¿Preguntas? Únete a La Plaza en el PUB de solarnethub.com*
