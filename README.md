# Infraestructura de Odoo 19 - Despliegue en Entorno Docker

## Especificaciones de Arquitectura
Este repositorio contiene los manifiestos de despliegue para inicializar un ecosistema de Odoo 19 utilizando contenedores Docker. La arquitectura está diseñada para operar con alta eficiencia sobre un sistema anfitrión Ubuntu, integrando un proxy inverso y gestión de bases de datos.

### Componentes de la Unidad
* **Odoo (Versión 19):** Interfaz de aplicación y procesamiento lógico.
* **PostgreSQL (Versión 18):** Motor de base de datos relacional.
* **Nginx Proxy Manager:** Gestión centralizada de enrutamiento web y terminación SSL.

## Topología de Red
El sistema utiliza una red puente (bridge) denominada `odoo-net` para aislar las comunicaciones internas. 
Para entornos que operan con túneles de seguridad perimetral (como Cloudflare Tunnel) y hardware de enrutamiento avanzado (ej. MikroTik RB4011), el tráfico externo debe ser dirigido hacia los puertos 80/443 de Nginx Proxy Manager. El proxy enrutará las peticiones HTTP de forma segura hacia el puerto 8069 del contenedor Odoo. La resolución DNS interna está configurada para consultar directamente al enrutador local.

## Protocolo de Inicialización

### 1. Preparación de Parámetros
Copie el archivo de ejemplo para establecer sus variables de entorno.
```bash
cp env.example .env
```
**Directiva de Seguridad Crítica:** Usted debe modificar el archivo `.env` para asignar secuencias criptográficas seguras a las variables `PASSWORD` e `INITIAL_ADMIN_PASSWORD`. Mantener las credenciales predeterminadas es un fallo de seguridad inaceptable.

### 2. Configuración del Sistema Odoo
Asegúrese de que el archivo `./etc/odoo.conf` esté presente y configurado, incluyendo la directiva `admin_passwd` para la creación de bases de datos, y los parámetros `proxy_mode = True` y `http_interface = 0.0.0.0` para garantizar la conectividad con el proxy.

### 3. Ejecución de Despliegue
El ecosistema está fragmentado en perfiles operativos para garantizar la modularidad y la adaptación a diferentes infraestructuras.

**Inicialización Completa (Odoo + Proxy Inverso):**
Ejecute este comando si la red no posee un proxy activo.
```bash
docker compose --profile odoo --profile proxy up -d
```

**Inicialización Parcial (Solo Odoo):**
Ejecute este comando si la infraestructura corporativa ya posee un nodo de Nginx Proxy Manager externo gestionando la red.
```bash
docker compose --profile odoo up -d
```

## Estructura de Volúmenes
La preservación estructural es obligatoria. Los directorios `./addons` (módulos adicionales), `./etc` (configuraciones), `./data` (datos del proxy) y `./letsencrypt` (certificados SSL) deben mantenerse en la jerarquía del repositorio.
