# 🎯 BACKLOG SESIÓN - Oasis Docker Setup

**Fecha**: 2025-12-25  
**Objetivo**: Levantar Oasis en Docker, crear cuenta, poner avatar, usar invitación PUB, backup USB

---

## 📋 BACKLOG

| ID | Estado | Tarea | Notas |
|----|--------|-------|-------|
| 0 | ✅ COMPLETADO | **Análisis profundo del repositorio** | Ver hallazgos abajo |
| 0.1 | 🔄 EN CURSO | **Pre-Hackaton: Requisitos sistema** | VS Code, git, gh CLI |
| 0.5 | ✅ COMPLETADO | **Pre-Sprint: Actualización Oasis** | 0.4.9 → 0.6.3 ✅ |
| 1 | ⏳ PENDIENTE | Preparar entorno (volúmenes, configs) | Volúmenes ya existen |
| 2 | ⏳ PENDIENTE | Build imagen Docker | `npm run build` |
| 3 | ⏳ PENDIENTE | Levantar contenedor | `npm run up` |
| 4 | ⏳ PENDIENTE | Verificar acceso web localhost:3000 | |
| 5 | ⏳ PENDIENTE | Crear identidad / perfil / avatar | |
| 6 | ⏳ PENDIENTE | Usar invitación PUB | |
| 7 | ⏳ PENDIENTE | **BACKUP credenciales USB** | CRÍTICO - crear backup-keys.sh |

---

## 🛠️ PRE-HACKATON: REQUISITOS SISTEMA (0.1)

| Requisito | Estado | Notas |
|-----------|--------|-------|
| VS Code | ✅ | En uso |
| Git | ✅ | Funcionando |
| Docker | ✅ | v29.1.3 + Compose v2.40.3 |
| NVIDIA Runtime | ✅ | Quadro P2000 detectada |
| gh CLI | ⏳ | Pendiente instalar |
| Auth GitHub | ⏳ | Pendiente web-auth |

---

## 🔧 PRE-SPRINT: ACTUALIZACIÓN OASIS (0.5) ✅ COMPLETADO

**Resultado**: Merge exitoso de oasis 0.6.3

```
Rama: hackaton_261225
Commit: 678819e - Merge oasis 0.6.3 - take theirs for app files
Versión: 0.6.3 (antes 0.4.9)
```

### Archivos PRESERVADOS (nuestros) ✅:
- Dockerfile
- docker-compose.yml  
- docker-entrypoint.sh
- docker-scripts/*
- package.json (raíz)

### Archivos ACTUALIZADOS (theirs) ✅:
- src/backend/backend.js (+168 líneas - fix menciones)
- src/models/*.js (nuevos: courts, parliament, favorites)
- src/views/*.js (nuevos: courts, parliament, favorites)
- src/server/package.json → 0.6.3
- docs/CHANGELOG.md

### Conflicto resuelto:
- `src/AI/ai_service.mjs` - aceptado versión upstream

---

## 📝 ANÁLISIS COMPLETO DEL REPOSITORIO

### Arquitectura descubierta:

```
package.json scripts:
├── npm run setup      → docker-scripts/setup.sh (crea volumes-dev/)
├── npm run build      → docker-compose build
├── npm run up         → setup + docker-compose up -d
├── npm run logs       → docker-compose logs -f
├── npm run shell      → acceso bash al contenedor
├── npm run backup-keys→ docker-scripts/backup-keys.sh ⚠️ NO EXISTE
└── npm run web        → abre http://localhost:3000
```

### docker-entrypoint.sh (501 líneas) hace:
1. `setup_ssb_config()` - Genera clave SHS y config SSB
2. `download_ai_model()` - Descarga modelo 3.8GB si no existe
3. `install_runtime_deps()` - Instala deps faltantes
4. `apply_node_patches()` - Parchea ssb-ref, ssb-blobs, multiserver
5. `setup_oasis_config()` - Activa/desactiva AI según modelo
6. Inicia `backend.js` en modo full

### docker-compose.yml:
- Puerto 3000 (web) y 8008 (SSB)
- GPU NVIDIA habilitada (tu Quadro P2000 sirve)
- Volúmenes bind a `./volumes-dev/{ssb-data,ai-models,logs}`

### ⚠️ PROBLEMAS DETECTADOS:

1. **`npm run backup-keys` referencia un script que NO EXISTE**
   - `docker-scripts/backup-keys.sh` no está en el repo
   - SOLUCIÓN: Crearlo nosotros

2. **El setup.sh crea un dir `configs/` que el compose NO usa**
   - setup.sh: `mkdir -p volumes-dev/{ssb-data,ai-models,configs,logs}`
   - compose: solo monta ssb-data, ai-models, logs
   - No es problema, simplemente sobra

3. **Permisos en Windows**
   - `chmod 700` en setup.sh no aplica en NTFS
   - No debería ser problema para Docker (Linux dentro)

### ✅ COSAS QUE ESTÁN BIEN:

- Los volúmenes `volumes-dev/` YA EXISTEN (los creé antes precipitadamente)
- Docker tiene nvidia runtime configurado
- Tu GPU Quadro P2000 4GB es suficiente para el modelo Q4_K_M

---

---

## 🚨 ISSUE CRÍTICO: ACTUALIZACIÓN DE VERSIÓN

### Situación actual:
| Concepto | Valor |
|----------|-------|
| **Versión local** | 0.4.9 (Sept 2025) |
| **Versión upstream** | 0.6.3 (10 Dic 2025) |
| **Salto** | 14 releases (0.4.9 → 0.5.0 → ... → 0.6.3) |

### Análisis del salto 0.4.9 → 0.6.3:

**Cambios mayores entre 0.5.0 y 0.6.3:**
- 🆕 Parliament plugin (sistema de gobierno)
- 🆕 Courts plugin (resolución de conflictos)
- 🆕 Footer añadido (Core plugin)
- 🆕 Favorites para módulos media
- 🔧 Muchos fixes en Feed, Mentions, Search, Activity
- 🔒 Security fixes en 0.6.2

**Archivos modificados en 0.6.3 específicamente:**
- `src/backend/backend.js` (+168/-66 líneas) ← **CAMBIO GRANDE en mentions**
- `src/models/feed_model.js` (+38/-7 líneas)
- `src/views/activity_view.js`, `feed_view.js`, `main_views.js`, `market_view.js`
- `src/server/package.json` (version bump)

### 🎯 DECISIÓN ESTRATÉGICA:

**OPCIÓN A - Actualizar primero (RECOMENDADO)**
- Pros: Tendremos la última versión, security fixes, mejor UX
- Cons: Puede romper algo del entorno Docker personalizado
- Riesgo: MEDIO (solo fixes, no breaking changes según semver)

**OPCIÓN B - Continuar con 0.4.9**  
- Pros: Sin riesgo de romper nada
- Cons: Versión desactualizada, bugs conocidos en mentions/feeds

### ⚠️ EVALUACIÓN DE RIESGO:

El commit 0.6.3 es **solo FIXES**, no hay breaking changes:
```
+ Fixed mentions (Core plugin).
+ Fixed feeds (Feed plugin).
+ Minor details at market view (Market plugin).
```

El cambio en `backend.js` es una **reescritura de la función `preparePreview`** 
para arreglar las menciones. Es interno, no cambia API.

**VEREDICTO**: Actualizar es **seguro** según semver. El riesgo está en si el 
entorno Docker del fork `alephscript-network-sdk` tiene modificaciones propias 
que conflicten.

---

## 🔄 HISTORIAL DE ITERACIONES

### Iteración 0 - Análisis profundo
**Estado**: ✅ COMPLETADO  
**Hallazgos**: Ver arriba. El repo está bien estructurado pero falta backup-keys.sh

### Iteración 1 - Análisis de versiones
**Estado**: ✅ COMPLETADO  
**Hallazgos**: 
- Local: 0.4.9, Upstream: 0.6.3 (14 releases de diferencia)
- Cambios son fixes, no breaking changes
- Archivos críticos modificados: backend.js, feed_model.js, views

### Iteración 2 - Decisión de actualización
**Estado**: 🔄 PENDIENTE DECISIÓN USUARIO
**Opciones**:
- A) Actualizar a 0.6.3 antes del build
- B) Continuar con 0.4.9, actualizar después  

