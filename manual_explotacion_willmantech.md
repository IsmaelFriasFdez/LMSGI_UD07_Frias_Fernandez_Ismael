# Manual de Explotación

# **1\. Introducción y Arquitectura**

## **1.1 Módulos Activados**

El sistema parte de la instalación del módulo base de Odoo, sobre el cual se articulan los procesos de la empresa:

* **Gestión de Ventas y Facturación:** Ciclo de vida desde el presupuesto hasta la emisión de la factura.  
* **Contabilidad y Finanzas:** Gestión de asientos y control tributario.  
* **Inventario:** Control de existencias.

## **1.2 Topología Lógica y Arquitectura**

El sistema se despliega mediante **Docker Compose**, dividiendo la arquitectura en dos servicios interdependientes:

* **Capa de Aplicación (odoo):** Contenedor basado en la imagen odoo:latest que expone el servicio web en el puerto **8200** del servidor host (mapeado al 8069 del contenedor).  
* **Capa de Datos (db):** SGBD relacional basado en postgres:latest.  
* **Persistencia:** Se utilizan volúmenes gestionados (odoo\_data, db\_data) y mapeos locales (./config, ./addons) para garantizar que la información de la base de datos, las configuraciones y los módulos extra sobrevivan al ciclo de vida de los contenedores.
# **2\. Guía de Instalación y Reinstalación**

Este procedimiento detalla cómo levantar el entorno Odoo desde cero, basándose en la configuración declarativa del archivo **docker-compose.yml** proporcionado.

## **2.1 Requisitos Previos**

* Servidor con **Docker** y **Docker Compose** instalados.  
* Estructura de directorios creada en la raíz del proyecto:  
  * Directorio ./config (para el archivo odoo.conf).  
  * Directorio ./addons (para módulos personalizados de WillmanTech).

## **2.2 Despliegue del Entorno**

El sistema utiliza un archivo **docker-compose.yml** que define automáticamente las variables de entorno para la conexión (USER=odoo, PASSWORD=odoo, POSTGRES\_DB=odoo).

1. Ubicar el archivo docker-compose.yml en el servidor.  
2. Ejecutar el siguiente comando para descargar las imágenes y levantar los servicios en segundo plano: ***docker compose up \-d***  
     
     
3. Verificar que los contenedores odoo y db están en ejecución: ***docker compose ps***  
4. Acceder a la plataforma a través del navegador web usando la IP del servidor y el puerto definido: ***http://\<IP\_SERVIDOR\>:8200*** o ***localhost:8200***
