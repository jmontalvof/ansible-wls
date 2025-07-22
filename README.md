# 🛠️ Ansible WebLogic Deployment Pipeline

Este repositorio contiene una estructura automatizada para desplegar aplicaciones WAR en servidores WebLogic, usando Ansible, Jenkins y una arquitectura de carpetas modular.

---

## 📁 Estructura del repositorio
ansible-wls/ ├── inventory/            # Inventarios por entorno │   └── pre.yml ├── Jenkinsfile           # Pipeline universal ├── logs/                 # Logs de ejecución ├── playbooks/            # Playbooks reutilizables │   ├── prepare.yml │   ├── deploy.yml │   └── postdeploy.yml ├── README.md             # Documentación del proyecto └── vars/                 # Variables por aplicación └── pedidos-api.yml


---

## 🚀 Flujo de despliegue

1. **Prepare**: crea carpetas, verifica usuario y descarga artefacto
2. **Deploy**: usa `weblogic.Deployer` para instalar el WAR en el servidor
3. **Postdeploy**: realiza chequeo HTTP de la aplicación en WebLogic

El Jenkinsfile es genérico, adaptable a cualquier aplicación mediante `vars/<app>.yml`.

---

## 🧪 Requisitos del entorno de pruebas

- Servidor WebLogic accesible
- Usuario de despliegue (`jorge` o `ansible_desp`) con:
  - Clave SSH configurada en Jenkins
  - Permisos `sudo` sin contraseña (`NOPASSWD:ALL`)
- Carpetas locales organizadas:

/opt/ ├── deploy/ │   └── pedidos-api/         # WARs, plantillas, configuraciones ├── logs/ │   └── pedidos-api/         # Logs técnicos y de verificació


---

## 🗄️ Estructura en NAS (producción)

En entornos reales, los servidores WebLogic pueden montar una carpeta NFS exportada desde una NAS central:

/export/weblogic_domain/ ├── domains/                 # Archivos del dominio WebLogic ├── deploy/                 # Artefactos por aplicación (WAR/EAR) ├── logs/                   # Logs centralizados


Todos los nodos del cluster acceden a esta estructura común.

---

## 📘 Ejemplo de configuración (vars/pedidos-api.yml)

```yaml
app_name: pedidos-api
war_path: /opt/deploy/pedidos-api/pedidos-api-1.0.0.war
dominio: 192.168.1.143
targets: Cluster01
wl_user: weblogic
wl_pass: admin123

🔐 Seguridad y buenas prácticas
- Gestiona credenciales en Jenkins, no en texto plano
- Usa .gitignore para excluir claves, .env o secretos
- Versiona únicamente artefactos y configuraciones estables
- No subir .pem, .key ni contraseñas al repositorio

🎯 Mejoras futuras
- Integración con NAS vía NFS (o EFS si escala a la nube)
- Separación por entornos (dev, pre, pro) con inventarios y estructuras propias
- Validación previa del estado de la aplicación (listapps)
- Notificaciones automáticas vía correo o Slack desde Jenkin

🙌 Autor
Configurado y gestionado por Jorge — entorno de pruebas replicable para despliegues automatizados en WebLogic usando herramientas libres, buenas prácticas y sentido común 


