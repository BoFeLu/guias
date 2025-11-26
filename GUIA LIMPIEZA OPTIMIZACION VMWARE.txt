# Guía Completa de Limpieza y Optimización de VMware Ubuntu

## Introducción

Esta guía documenta el proceso completo para limpiar, reorganizar y optimizar una máquina virtual Ubuntu en VMware Workstation, recuperando espacio significativo tanto en el guest como en el host. El proceso está diseñado para desarrolladores y administradores de sistemas que necesitan mantener un entorno de trabajo eficiente.

**Contexto**: Después de completar proyectos intensivos de desarrollo (como prácticas FCT), es común acumular archivos temporales, imágenes Docker obsoletas, y estructuras de directorios caóticas que consumen espacio innecesariamente.

## Tabla de Contenidos

1. [Análisis Inicial del Sistema](#análisis-inicial)
2. [Limpieza de Docker](#limpieza-docker)
3. [Reorganización de Directorios](#reorganización)
4. [Compactación de Disco VMware](#compactación-vmware)
5. [Solución de Problemas Comunes](#troubleshooting)
6. [Mejores Prácticas](#mejores-prácticas)

## Análisis Inicial del Sistema {#análisis-inicial}

### Comandos de Diagnóstico Fundamentales

```bash
# Estado general del sistema de archivos
df -h

# Análisis detallado por directorios del usuario
sudo du -h --max-depth=1 ~/ | sort -hr

# Estado específico de Docker
docker system df -v

# Localización de archivos grandes (>500MB)
find ~/ -type f -size +500M -exec ls -lh {} \; 2>/dev/null | sort -k5 -hr
```

### Interpretación de Resultados

Los comandos anteriores te proporcionarán:
- **df -h**: Uso general del disco, porcentaje ocupado
- **du -h**: Identificación de directorios que consumen más espacio
- **docker system df**: Imágenes, contenedores y caché Docker
- **find**: Archivos individuales grandes que pueden eliminarse

**Ejemplo de análisis típico**:
```
/home/usuario/respaldo-docker/     4.7GB
/home/usuario/Escritorio/          3.6GB  
/home/usuario/.minikube/           1.5GB
/home/usuario/.cache/              713MB
```

## Limpieza de Docker {#limpieza-docker}

### Limpieza Segura (sin afectar proyectos activos)

```bash
# Verificar contenedores en ejecución
docker ps

# Eliminar imágenes no utilizadas (dangling)
docker image prune -f

# Limpiar caché de build
docker buildx prune -f

# Limpieza completa del sistema Docker (conservando contenedores activos)
docker system prune -f
```

### Limpieza Selectiva de Imágenes Obsoletas

```bash
# Listar todas las imágenes
docker images

# Eliminar imágenes específicas obsoletas (ejemplo)
docker rmi php-app-base:latest
docker rmi proyecto-obsoleto:latest

# Eliminar imágenes huérfanas
docker rmi $(docker images -f "dangling=true" -q) 2>/dev/null
```

### Casos Especiales

**Minikube corrompido**:
```bash
# Verificar estado
minikube status

# Si está corrupto, eliminación completa
minikube delete 2>/dev/null
rm -rf ~/.minikube
```

## Reorganización de Directorios {#reorganización}

### Principios de Organización

Implementamos una metodología de "orden, claridad y limpieza" donde cada elemento tiene un propósito específico y una ubicación lógica.

### Estructura Propuesta para Escritorio de Desarrollador

```
~/Escritorio/
├── PROYECTOS-ACTIVOS/           # Trabajo en curso
├── PORTFOLIO/                   # Proyectos terminados/destacados  
├── LABORATORIOS/               # Experimentos y prácticas
└── RECURSOS/                   # Documentación y referencias
```

### Comandos de Reorganización

```bash
# Crear estructura
mkdir -p ~/Escritorio/{PROYECTOS-ACTIVOS,PORTFOLIO,LABORATORIOS,RECURSOS}

# Movimiento con preservación de permisos
sudo mv ~/Escritorio/proyecto-principal ~/Escritorio/PROYECTOS-ACTIVOS/
sudo mv ~/Escritorio/app-terminada ~/Escritorio/PORTFOLIO/
sudo mv ~/Escritorio/practicas-* ~/Escritorio/LABORATORIOS/
```

### Gestión de Permisos MySQL/Docker

Los directorios `db/data/` creados por Docker pueden tener permisos restrictivos:

```bash
# Identificar directorios problemáticos
sudo ls -la ~/proyecto/db/data/

# Cambiar propietario recursivamente
sudo chown -R $USER:$USER ~/proyecto/db/

# Verificar cambio
ls -la ~/proyecto/db/data/
```

## Compactación de Disco VMware {#compactación-vmware}

### Proceso de Preparación en Guest

La compactación requiere "llenar" el espacio libre con ceros para que VMware pueda identificarlo como espacio recuperable:

```bash
# Llenar espacio libre con ceros (proceso lento, 30-90 minutos)
sudo dd if=/dev/zero of=/tmp/zero.img bs=1M

# Monitorear progreso en otra terminal
watch -n 5 'ls -lh /tmp/zero.img 2>/dev/null'

# El proceso terminará automáticamente con "No space left on device"
```

### Limpieza Post-Preparación

```bash
# Eliminar archivo temporal (liberará espacio en guest)
sudo rm -f /tmp/zero.img

# Verificar espacio restaurado
df -h /
```

### Compactación Desde Guest (VMware Tools)

```bash
# Intentar compactación automática
sudo vmware-toolbox-cmd disk shrink /
```

### Compactación Manual Desde Host

Si la compactación automática falla:

1. **Apagar la VM completamente**
2. **Eliminar snapshots existentes**:
   - VMware Workstation → VM → Snapshot Manager
   - Delete All Snapshots
3. **Compactación manual**:
   - Edit VM Settings → Hard Disk → Utilities → Compact

### Método Alternativo con vmware-vdiskmanager

```cmd
# En Windows (CMD como administrador)
cd "C:\Program Files (x86)\VMware\VMware Workstation"
vmware-vdiskmanager.exe -k "ruta\completa\a\tu\archivo.vmdk"
```

## Solución de Problemas Comunes {#troubleshooting}

### Error: "La reducción de disco está deshabilitada"

**Causas comunes**:
- Snapshots existentes
- VM configurada como clone vinculado
- Discos previamente asignados

**Soluciones**:
1. Eliminar todas las instantáneas
2. Verificar configuración de VM (no debe ser clone vinculado)
3. Usar compactación manual desde host

### Permisos Denegados Durante Limpieza

```bash
# Usar sudo para operaciones de limpieza
sudo rm -rf directorio_problematico/

# Cambiar propietarios antes de operaciones normales
sudo chown -R $USER:$USER directorio/
```

### Proceso dd Lento

Es normal que el proceso tome tiempo. Para monitorear:

```bash
# Ver progreso en tiempo real
watch -n 5 'df -h /'

# Enviar señal para mostrar estadísticas
pidof dd | xargs -I {} sudo kill -USR1 {}
```

## Mejores Prácticas {#mejores-prácticas}

### Mantenimiento Preventivo

```bash
# Limpieza mensual recomendada
sudo apt clean
sudo apt autoremove
docker system prune -f
rm -rf ~/.cache/pip ~/.cache/npm
```

### Gestión de Snapshots

- **Crear snapshot después de limpieza**: Estado "golden" optimizado
- **Eliminar snapshots antiguos**: Previene problemas de compactación
- **Nombrado descriptivo**: "Sistema-Limpio-Post-Proyecto-YYYY-MM"

### Organización Continua

- **Principio de categorización**: Cada archivo en su lugar lógico
- **Eliminación regular**: No acumular archivos temporales
- **Documentación**: README en cada directorio principal

### Monitoreo de Espacio

```bash
# Script simple para alertas de espacio
df -h / | awk 'NR==2 {if($5+0 > 80) print "Alerta: Disco al " $5}'
```

## Resultados Esperados

Después de seguir esta guía completa:

- **Recuperación de espacio**: 5-10GB típico en proyectos de desarrollo
- **Organización mejorada**: Estructura clara y mantenible
- **Rendimiento optimizado**: VM más rápida, menor fragmentación
- **Host optimizado**: Archivo .vmdk compactado físicamente

## Conclusión

Este proceso de limpieza y optimización no solo recupera espacio, sino que establece una base sólida para un entorno de desarrollo mantenible. La combinación de técnicas de limpieza del sistema operativo, reorganización lógica y optimización de VMware proporciona un flujo de trabajo eficiente para proyectos futuros.

La metodología presentada es escalable y puede adaptarse a diferentes tipos de proyectos y entornos de desarrollo, manteniendo siempre los principios fundamentales de orden, claridad y eficiencia.

---

**Nota**: Esta guía ha sido probada en VMware Workstation con Ubuntu 24.04. Los comandos pueden requerir ajustes menores para otras versiones o distribuciones.