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
# **3\. Seguridad y Control de Acceso**

La gestión de accesos se fundamenta en el modelo de Control de Acceso Basado en Roles (RBAC) propio de Odoo.

## **3.1 Roles y Privilegios**

| Rol | Privilegios sobre la Información |
| :---- | :---- |
| **Administrador** | Acceso total a los ajustes de Odoo, instalación de *addons*, gestión de usuarios y acceso a todos los módulos. |
| **Contable** | Lectura/Escritura en módulos financieros. Acceso en modo lectura a Ventas e Inventario para validación de ingresos/costes. |
| **Comercial** | Lectura/Escritura limitada a sus propios clientes y presupuestos. Sin acceso a la configuración del sistema ni a la contabilidad general. |

#### 

## **3.2 Políticas de Contraseñas**

Para proteger el acceso a la interfaz web (puerto 8200), se deben aplicar las siguientes reglas organizativas para todos los usuarios:

* **Longitud mínima:** 12 caracteres.  
* **Complejidad:** Uso combinado de mayúsculas, minúsculas, números y símbolos.  
* **Rotación:** Cambio cada 90 días.
# **4\. Procedimiento de Backup y Restauración**

Dado que la base de datos PostgreSQL se encuentra contenerizada bajo el nombre db y el usuario/base de datos están definidos como odoo, las operaciones de volcado de datos deben realizarse a través del cliente de Docker.

## **4.1 Creación de Backup (Respaldo)**

Para extraer una copia de seguridad en caliente (sin detener el ERP):

docker exec \-t \<nombre\_contenedor\> pg\_dump \-U \<usuario\> \<nombre\_bd\> \> /ruta/en/tu/host/backup.sql

Ejemplo:  
docker exec \-t postgres-db pg\_dump \-U postgres mi\_base\_datos \> ./backup/mi\_base\_datos.sql

## **4.2 Restauración del Sistema (Restore)**

Para restaurar una copia de seguridad ante un desastre:

1\. Eliminar la base de datos actual (requiere detener el contenedor Odoo primero)

docker compose stop odoo  
docker exec \-it db dropdb \-U odoo odoo  
docker exec \-it db createdb \-U odoo odoo

2\. Restaurar el backup:  
docker exec \-i db pg\_restore \-U odoo \-d odoo \-1 \< /ruta/segura/backups/archivo\_backup.dump

3\. Reiniciar el ERP  
docker compose start odoo
# **5\. Flujo Operativo de Facturación e Informes**

Esta sección explica la arquitectura de renderizado de Odoo para la depuración técnica y el flujo de trabajo del usuario final.

## **5.1 Generación de Facturas en Interfaz**

1. **Navegación:** En el menú principal de Odoo, acceder al módulo de *Facturación* y seleccionar *Nueva Factura*.  
2. **Cabecera:** Seleccionar el cliente. Odoo autocompletará la posición fiscal, NIF y términos de pago.  
3. **Líneas de Factura:** Añadir los productos, cantidades y precios. Los impuestos se calculan dinámicamente.  
4. **Confirmación:** Al pulsar **Confirmar**, el asiento contable se publica y se asigna un número de factura inmutable.  
5. **Descarga:** Hacer clic en **Imprimir \> Factura**.

## **5.2 El Pipeline de Renderizado a PDF (wkhtmltopdf)**

Odoo utiliza una arquitectura específica para transformar los datos de la base de datos en un documento PDF imprimible:

1. **Extracción (QWeb):** El motor de plantillas de Odoo (QWeb) extrae los datos de PostgreSQL y genera un documento XML/HTML dinámico.  
2. **Renderizado HTML:** El sistema combina estos datos con los estilos CSS de la compañía.  
3. **Conversión Binaria (wkhtmltopdf):** Odoo envía este HTML renderizado a la librería del sistema wkhtmltopdf (incluida dentro de la imagen oficial de Odoo). Esta herramienta utiliza el motor WebKit para renderizar el HTML exactamente como lo haría un navegador web y lo empaqueta en un archivo PDF que se descarga en el equipo del usuario.
