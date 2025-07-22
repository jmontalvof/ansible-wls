# 🛠️ Ansible WebLogic Deployment Pipeline

Este repositorio contiene una estructura modular y automatizada para desplegar aplicaciones WAR en servidores WebLogic usando Ansible y Jenkins.

---

## 📦 Estructura del repositorio
# ansible-wls
ansible-wls/
├── inventory/           # Inventarios por entorno (pre, pro, etc.)
│   └── pre.yml
├── Jenkinsfile          # Pipeline universal para todas las apps
├── logs/                # Carpeta opcional para registros de ejecución
├── playbooks/           # Playbooks reutilizables
│   ├── prepare.yml
│   ├── deploy.yml
│   └── postdeploy.yml
├── README.md            # Este documento
└── vars/                # Variables por aplicación
    └── pedidos-api.yml

---

## 🚀 Flujo de despliegue automatizado

1. **Prepare**: Verifica usuario, crea carpetas y copia artefactos
2. **Deploy**: Ejecuta `weblogic.Deployer` para instalar el WAR
3. **Postdeploy**: Verifica la URL pública del app vía HTTP

El Jenkinsfile es universal. Sólo necesitas definir variables por app en el archivo correspondiente dentro de `vars/`.

---

## 🧪 Requisitos para el entorno de pruebas

- 🐧 Servidor WebLogic corriendo localmente
- 🔐 Usuario de despliegue (`jorge` o `ansible_desp`) con:
  - Acceso SSH
  - Permisos `NOPASSWD:ALL` en `/etc/sudoers`
- 📂 Carpetas por aplicación:
  - `/opt/deploy/<app>/`
  - `/opt/logs/<app>/`

---

## 🌐 Estructura recomendada de montaje en WebLogic

Para cada aplicación:
/opt/
├── deploy/
│   └── pedidos-api/         # WARs, plantillas, configuraciones específicas
├── logs/
│   └── pedidos-api/         # Logs técnicos, respuesta HTTP, trazas de ejecución

Y en la NAS (producción real):

/export/weblogic_domain/
├── domains/                 # Dominios WebLogic centralizados
├── deploy/                  # Repositorio de artefactos por aplicación
├── logs/                    # Logs centralizados accesibles por Jenkins / Sysadmin

---

## 📚 Ejemplo de configuración por aplicación

# vars/pedidos-api.yml

app_name: pedidos-api
war_path: /opt/deploy/pedidos-api/pedidos-api-1.0.0.war
dominio: 192.168.1.143
targets: Cluster01
wl_user: weblogic
wl_pass: admin123

🔐 Consideraciones de seguridad
- Las claves SSH deben estar gestionadas por Jenkins Credentials
- Nunca subir .pem, .key o contraseñas al repositorio
- Usa .gitignore para evitar archivos sensibles

📘 Mejora futura
Este entorno está pensado para pruebas. Para producción se recomienda:
- Uso de NAS compartida (TrueNAS / OpenMediaVault)
- Centralización de logs y configuraciones
- Despliegue bajo Jenkins CI con notificacione
- Validación previa con listapps antes de cada deploy

🙌 Autor
Proyecto gestionado por [Jorge] — despliegue automatizado sobre WebLogic con herramientas libres 


