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
