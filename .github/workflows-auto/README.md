# Documentación de Workflows CI/CD - gp-api-balancer

## 📋 Tabla de Contenidos
- [Visión General](#visión-general)
- [Workflows Disponibles](#workflows-disponibles)
- [Flujos Automáticos](#flujos-automáticos)
- [Ejecución Manual](#ejecución-manual)
- [Configuración de Secrets](#configuración-de-secrets)
- [Guía de Modificación](#guía-de-modificación)
- [Solución de Problemas](#solución-de-problemas)

---

## 🎯 Visión General

Este proyecto utiliza GitHub Actions para automatizar el proceso de build y deploy. Los workflows están diseñados para:

1. **Compilar y construir** la imagen Docker cuando hay cambios en el código
2. **Desplegar automáticamente** a los entornos correspondientes según la rama
3. **Permitir deploys manuales** cuando sea necesario

---

## 🔄 Workflows Disponibles

### 1. GP BUILD (`build.yaml`)
**Propósito:** Compilar el código y construir la imagen Docker

**Se activa cuando:**
- Hay un `push` a las ramas `test` o `develop`
- Se ejecuta manualmente desde GitHub Actions

**Qué hace:**
1. Compila el código fuente Java con Maven
2. Construye la imagen Docker
3. Publica la imagen en el registry
4. Al finalizar exitosamente, dispara automáticamente el workflow de DEPLOY

### 2. GP DEPLOY (`deploy.yaml`)
**Propósito:** Desplegar la aplicación en los diferentes entornos

**Se activa cuando:**
- El workflow "GP BUILD" finaliza exitosamente
- Se ejecuta manualmente desde GitHub Actions

**Qué hace:**
1. Actualiza el repositorio de ArgoCD con la nueva versión
2. ArgoCD detecta el cambio y despliega automáticamente
3. Envía notificación a Teams sobre el resultado

---

## 🚀 Flujos Automáticos

### Flujo para rama `test`
```
Push a 'test' 
  → BUILD automático 
  → (si éxito) DEPLOY automático a ROSADEV
  → Notificación en Teams
```

### Flujo para rama `develop`
```
Push a 'develop' 
  → BUILD automático 
  → (si éxito) DEPLOY automático a ROSAPRE
  → Notificación en Teams
```

### Mapeo de Ramas a Entornos
| Rama    | Entorno  | Descripción           |
|---------|----------|-----------------------|
| test    | ROSADEV  | Entorno de desarrollo |
| develop | ROSAPRE  | Pre-producción        |
| *       | ROSA*    | Producción (futuro)   |

*_Aún no configurado_

---

## 🎮 Ejecución Manual

### Ejecutar BUILD manualmente
1. Ve a: `Actions` → `GP BUILD` → `Run workflow`
2. Selecciona la rama que quieres compilar
3. Click en `Run workflow`

### Ejecutar DEPLOY manualmente
1. Ve a: `Actions` → `GP DEPLOY` → `Run workflow`
2. Selecciona:
   - **Rama:** La rama que quieres desplegar
   - **Target:** El entorno destino (ROSADEV, ROSAPRE, ROSA)
3. Click en `Run workflow`

**Casos de uso para deploy manual:**
- Redesplegar una versión específica
- Desplegar a producción (ROSA)
- Probar un deploy en un entorno diferente al automático

---

## 🛠️ Guía de Modificación

### Agregar una nueva rama automática (ej: `rama-ejemplo` → `ROSA`)

#### 1. Modificar `build.yaml`:
```yaml
on:
  push:
    branches:
      - 'test'
      - 'develop'
      - 'rama-ejemplo'      # ← AGREGAR AQUÍ
```

#### 2. Modificar `deploy.yaml`:
```yaml
on:
  workflow_run:
    branches:
      - test
      - develop
      - rama-ejemplo        # ← AGREGAR AQUÍ
```

#### 3. Actualizar la lógica de TARGET en `deploy.yaml`:
```yaml
TARGET: ${{ 
  (github.event_name == 'workflow_run' && 
    (github.event.workflow_run.head_branch == 'test' && 'ROSADEV' || 
     github.event.workflow_run.head_branch == 'develop' && 'ROSAPRE' ||
     github.event.workflow_run.head_branch == 'rama-ejemplo' && 'ROSA')    # ← AGREGAR AQUÍ
  ) || 
  github.event.inputs.target 
}}
```

### Agregar un nuevo entorno de deploy

#### Modificar `deploy.yaml`:
```yaml
workflow_dispatch:
  inputs:
    target:
      options:
        - ROSADEV
        - ROSAPRE
        - ROSA
        - NUEVO_ENTORNO    # ← AGREGAR AQUÍ
```

### Cambiar el archivo de valores de Kubernetes

#### Modificar `deploy.yaml`:
```yaml
with:
  VALUES_FILE: ruta/a/tu/values.yaml    # ← CAMBIAR AQUÍ
```

---

## 🔍 Solución de Problemas

### El workflow no se ejecuta automáticamente
**Posibles causas:**
- La rama no está en la lista de `branches` en el workflow
- El commit fue hecho por un bot (algunos eventos de bot no disparan workflows)
- El archivo workflow tiene errores de sintaxis YAML

**Solución:**
1. Verifica que la rama esté configurada en `build.yaml`
2. Ejecuta el workflow manualmente para verificar que funciona
3. Revisa los logs en la pestaña `Actions`

### El BUILD fue exitoso pero no se desplegó
**Posibles causas:**
- La rama no está en la lista de `branches` en `deploy.yaml`
- El workflow de deploy tiene una condición `if` que no se cumple
- Falta algún secret requerido

**Solución:**
1. Verifica que la rama esté en `deploy.yaml` bajo `workflow_run.branches`
2. Revisa los logs del workflow de BUILD para ver si disparó el DEPLOY
3. Verifica que todos los secrets estén configurados

### Error: "Secret not found"
**Causa:** Falta configurar un secret requerido

**Solución:**
1. Consulta al equipo de DevOps para pedir revisión de los secrets necesarios

### El deploy se ejecutó pero la aplicación no se actualizó
**Posibles causas:**
- ArgoCD aún no ha terminado de sincronizar
- Hay un error en el archivo `values.yaml`
- La imagen Docker no se publicó correctamente

**Solución:**
1. Verifica en ArgoCD que la aplicación esté sincronizada
2. Revisa los logs del workflow de DEPLOY
3. Verifica que la imagen existe en el registry

---

## 📞 Contacto y Soporte

Para preguntas o problemas:
1. Revisa esta documentación y el documento general de migración a GitHub Actions
2. Consulta los logs en la pestaña `Actions` de GitHub
3. Contacta al equipo de DevOps
