# INFORME DE PRÁCTICA: IMPLEMENTACIÓN DE INFRAESTRUCTURA WEB CON DOCKER COMPOSE

## Título

**Despliegue de un CMS WordPress mediante Microservicios Orquestados: Integración de Servidor Web, Base de Datos y Gestor de DB en Entornos de Contenedores con Docker Compose**

---

## Tiempo de duración

**120 minutos**

### Plataforma de Simulación
Killercoda (Escenario: Port-forwarding in Docker)

---

# 3. Fundamentos

La tecnología de contenedores ha transformado significativamente el ciclo de vida del desarrollo y despliegue de software moderno. Los contenedores permiten empaquetar aplicaciones junto con todas sus dependencias, librerías y configuraciones necesarias, garantizando que el software funcione de manera consistente en diferentes entornos. Esta característica reduce problemas relacionados con incompatibilidades de versiones y diferencias entre sistemas operativos.

Docker es una de las plataformas más utilizadas para la administración de contenedores debido a su facilidad de uso, eficiencia y capacidad de automatización. A diferencia de las máquinas virtuales tradicionales, Docker comparte el kernel del sistema operativo anfitrión, permitiendo un menor consumo de recursos y una mayor velocidad de ejecución. Gracias a ello, múltiples servicios pueden ejecutarse de manera aislada dentro de un mismo equipo físico sin afectar el rendimiento general del sistema.

## Orquestación Local con Docker Compose

A medida que las aplicaciones escalan y se dividen en microservicios (Frontend, Backend y Bases de Datos), la ejecución manual e individual mediante comandos independientes (`docker run`) se vuelve ineficiente y propensa a errores humanos. Docker Compose soluciona esta problemática centralizando toda la arquitectura técnica (servicios, variables de entorno, redes dedicadas y volúmenes persistentes) en un único archivo declarativo estructurado en formato YAML (`YML`). Esto permite levantar o detener toda la infraestructura multi-contenedor con un solo comando unificado.

En esta práctica se implementó una arquitectura basada en microservicios utilizando tres contenedores independientes gestionados a través de un orquestador YAML:

- **Docker Engine / Compose:** Motor principal encargado de interpretar el archivo YML y coordinar el ciclo de vida de los microservicios.
- **MySQL (Servicio `mysql`):** Motor de bases de datos relacionales versión 8.0 asignado para almacenar tablas, posts, taxonomías y configuraciones del CMS.
- **WordPress (Servicio `wordpress`):** CMS principal que integra nativamente el entorno de ejecución PHP y el servidor web Apache en su imagen oficial de Docker Hub.
- **phpMyAdmin (Servicio `phpmyadmin`):** Interfaz gráfica web que simplifica las auditorías, consultas y gestión de la base de datos MySQL desde el navegador.

La comunicación interna se garantizó mediante el aislamiento en una red virtual personalizada (`Bridge Network`) gestionada internamente por nombres de host (servicios). Asimismo, la persistencia se garantizó mapeando volúmenes locales independientes para evitar la pérdida de información ante la destrucción o recreación de contenedores.

---

# 4. Conocimientos previos

- Estructuración de archivos en formato YAML.
- Administración y manejo de comandos Linux mediante terminal Bash.
- Diferencias entre comandos individuales (`Docker V1`) y orquestación unificada (`Docker Compose / V2`).
- Conceptos fundamentales de Redes Virtuales (`Bridge`), mapeo de puertos de red locales y persistencia por volúmenes.
- Administración y credenciales en bases de datos relacionales MySQL.

---

# 5. Objetivos a alcanzar

## Objetivo General

Implementar una infraestructura web multi-servicio mediante un archivo declarativo de Docker Compose (formato YML) para desplegar de forma integrada un CMS WordPress, una Base de Datos MySQL y un gestor phpMyAdmin dentro de la plataforma Killercoda.

## Objetivos Específicos

- Construir un archivo `docker-compose.yml` respetando de manera estricta la indentación por espacios obligatoria de la sintaxis YAML.
- Estructurar e integrar de forma jerárquica 3 servicios específicos: `mysql`, `wordpress` y `phpmyadmin`.
- Definir explícitamente una red puente personalizada de aislamiento interno para la intercomunicación directa por alias.
- Definir y asignar volúmenes independientes para asegurar la persistencia permanente de los datos de MySQL y WordPress.
- Homologar variables de entorno de seguridad con contraseña simplificada (`12345`) adaptadas para despliegue de laboratorio.
- Solucionar conflictos de redes y de versiones locales detectadas en la consola de comandos de Ubuntu.

---

# 6. Equipo necesario

- Computador con navegador web actualizado y acceso a Internet.
- Cuenta activa en la plataforma de aprendizaje práctico Killercoda.
- Playground virtualizado de **Port-forwarding in Docker** con Docker y Docker Compose preinstalados.

---

# 7. Material de apoyo

- Manual oficial de Docker Compose Specification.
- Esquemas de configuración de imágenes y variables de entorno oficiales en Docker Hub.
- Historial de comandos y resolución de problemas de redes y DNS en Ubuntu Linux.

---

# 8. Procedimiento y Comandos Mandados

A continuación se detalla la secuencia exacta de las acciones ejecutadas y comandos ingresados en la terminal derecha del playground de Killercoda.

---

## Paso 1: Diagnóstico inicial y selección del escenario

Se accedió a la plataforma Killercoda seleccionando el escenario avanzado de **Port-forwarding in Docker** con el fin de disponer de la pestaña técnica de desvío de tráfico de red remota.

---

## Paso 2: Generación automática del archivo de configuración declarativo

Para construir el entorno unificado respetando las reglas YAML, se utilizó el redireccionador de consola Linux `cat << 'EOF'`. Este comando generó el archivo `docker-compose.yml` inyectándole los 3 servicios, la red y los volúmenes sin necesidad de recurrir a editores visuales.

```bash
cat << 'EOF' > docker-compose.yml
version: '3.8'

# 1. Estructuración de los 3 servicios requeridos
services:

  # Servicio: Base de datos MySQL
  mysql:
    image: mysql:8.0
    container_name: servidor_mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: "12345"
      MYSQL_DATABASE: wordpress_db
      MYSQL_USER: wp_user
      MYSQL_PASSWORD: "12345"
    volumes:
      - datos_web:/var/lib/mysql
    networks:
      - red_practica

  # Servicio: Plataforma WordPress
  wordpress:
    image: wordpress:latest
    container_name: sitio_wordpress
    restart: always
    ports:
      - "8080:80"
    environment:
      WORDPRESS_DB_HOST: mysql:3306
      WORDPRESS_DB_USER: wp_user
      WORDPRESS_DB_PASSWORD: "12345"
      WORDPRESS_DB_NAME: wordpress_db
    volumes:
      - archivos_web:/var/www/html
    networks:
      - red_practica
    depends_on:
      - mysql

  # Servicio: phpMyAdmin
  phpmyadmin:
    image: phpmyadmin:latest
    container_name: panel_phpmyadmin
    restart: always
    ports:
      - "8081:80"
    environment:
      PMA_HOST: mysql
      MYSQL_ROOT_PASSWORD: "12345"
    networks:
      - red_practica
    depends_on:
      - mysql

# 2. Definición de una red personalizada
networks:
  red_practica:
    driver: bridge

# 3. Definición de volúmenes persistentes
volumes:
  datos_web:
  archivos_web:
EOF
```

---

## Paso 3: Verificación del contenido del archivo estructurado

Con el fin de asegurar que el bloque YAML se grabó íntegramente y sin corrupciones de caracteres, se ejecutó la lectura del archivo mediante:

```bash
cat docker-compose.yml
```

---

## Paso 4: Inicialización y Orquestación Multi-Contenedor

Para encender el ecosistema, se procedió a mandar el comando de ejecución en segundo plano (modo detached):

```bash
docker-compose up -d
```

---

## Nota de Incidencias Tecnológicas Solucionadas durante la Práctica

### Ajuste de Sintaxis por Versión de Docker

Al intentar lanzar inicialmente el comando moderno:

```bash
docker compose up -d
```

el sistema operativo notificó un error de interpretación de parámetros. Se detectó que el playground operaba bajo la arquitectura V1 clásica, lo cual se subsanó exitosamente mandando el comando:

```bash
docker-compose up -d
```

### Fallo Temporal de Red e Internet en VM Externa

En pruebas paralelas en una máquina virtual Ubuntu limpia, se identificó un bloqueo por falta de resolución DNS (`Temporary failure in name resolution`) al intentar instalar servicios locales.

Se solucionó forzando un servidor DNS global y reiniciando el servicio de red:

```bash
sudo bash -c 'echo "nameserver 8.8.8.8" > /etc/resolv.conf'
sudo systemctl restart systemd-resolved
```

---

## Paso 5: Auditoría y Verificación de Estado

Se mandó el siguiente comando para verificar el estado de los contenedores:

```bash
docker ps
```

---

# 9. Diagrama de la Solución

```text
┌────────────────────────┐
│ Interfaz de Killercoda │
└───────────┬────────────┘
            │
┌───────────▼────────────┐
│     Pestaña Traffic    │
├────────────────────────┤
│   Puerto 8080 | 8081   │
└───────┬────────┬───────┘
        │        │
   8080:80    8081:80
        │        │
┌───────▼───┐ ┌──▼──────────┐
│ WordPress │ │ phpMyAdmin  │
└───────┬───┘ └────┬────────┘
        │           │
        └────┬──────┘
             │
      red_practica
        (bridge)
             │
      Resolución DNS
          mysql
             │
       ┌─────▼─────┐
       │   MySQL   │
       └─────┬─────┘
             │
 ┌───────────▼───────────┐
 │ Volúmenes Persistentes│
 ├───────────────────────┤
 │ datos_web             │
 │ archivos_web          │
 └───────────────────────┘
```

**Figura 1-0.** Topología de microservicios, mapeos de puerto y persistencia local de la práctica.

---

# 10. Resultados Esperados y Evidencias

## A. Creación y Estado de Contenedores

Al mandar el comando:

```bash
docker ps
```

se evidenció la correcta orquestación arrojando en pantalla las siguientes entidades virtuales en estado estable:

- `sitio_wordpress` → Puerto local `8080`
- `panel_phpmyadmin` → Puerto local `8081`
- `servidor_mysql` → Puerto interno `3306`

Todos los servicios expusieron el estado:

```text
STATUS: Up
```

---

## B. Validación de Servicios Web mediante Redirección Remota

| Servicio | Puerto | Evidencia |
|---|---|---|
| WordPress | 8080 | Pantalla de instalación del CMS cargada correctamente |
| phpMyAdmin | 8081 | Panel de autenticación funcional y acceso exitoso |

### Credenciales utilizadas

```text
Usuario: root
Contraseña: 12345
```

---
<img width="589" height="276" alt="Imagen5" src="https://github.com/user-attachments/assets/c9e689dc-0c73-4978-b5ce-fadb58db3c69" />
<img width="524" height="594" alt="Imagen1" src="https://github.com/user-attachments/assets/206f553c-3a58-4e03-9121-af71cf139072" />
<img width="538" height="600" alt="Imagen2" src="https://github.com/user-attachments/assets/b9d56ad3-b785-4d0b-8c74-9741c7756dea" />
<img width="589" height="418" alt="Imagen3" src="https://github.com/user-attachments/assets/fbdf35b7-8b7e-422b-841d-91526430940d" />
<img width="589" height="483" alt="Imagen4" src="https://github.com/user-attachments/assets/6f5c4c55-c70b-4dc0-a39c-72be45727468" />


# 11. Conclusiones

- La utilización de Docker Compose permitió centralizar toda la infraestructura en un único archivo YAML.
- Se comprendió que WordPress integra internamente PHP y Apache.
- La red `bridge` permitió la resolución de nombres entre contenedores utilizando aliases como `mysql`.
- Los volúmenes persistentes aseguraron la conservación de datos aun después de reiniciar los contenedores.

---

# 12. Bibliografía

- Docker Documentation. (2024). *Compose file version 3 reference.*

https://docs.docker.com/compose/compose-file/compose-version3/

- Docker Hub Registry. (2024). *Official Repository for WordPress and MySQL Images.*

https://hub.docker.com/

- Méndez, A. R. (2021). *Contenedores de software: una alternativa para el despliegue de aplicaciones web. Revista Cubana de Ciencias Informáticas.*
