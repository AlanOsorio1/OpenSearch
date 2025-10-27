# 🧪 Ejercicio práctico: OpenSearch con instalación local y Faceted Search

## 📍 Contexto

OpenSearch es una plataforma **open source** para búsqueda y analítica de datos, derivada de Elasticsearch.  
El objetivo de este ejercicio fue **instalar OpenSearch localmente con Docker Compose**, cargar datos de ejemplo, crear un dashboard con visualizaciones y habilitar filtros dinámicos tipo *faceted search*.

---

## 🧱 1️⃣ Instalación local con Docker Compose

En esta sección se explica cómo instalar Docker, configurar el entorno y levantar los servicios de OpenSearch y OpenSearch Dashboards.

---

### 🔹 Paso 1. Instalar Docker y Docker Compose


```bash
sudo apt update
sudo apt install ca-certificates curl gnupg lsb-release -y
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
Verificar instalación:

```bash
docker --version
docker compose version
```

### 🔹 Paso 2. Crear carpeta del proyecto
Creamos una carpeta para contener los archivos:

```bash
mkdir -p /home/alan_osorio/Documentos/BLEND/opensearch_lab
cd /home/alan_osorio/Documentos/BLEND/opensearch_lab
```

### 🔹 Paso 3. Crear archivo .env
El archivo .env se utiliza para almacenar variables de entorno sensibles, como la contraseña del usuario administrador.

```bash
cat > .env << 'EOF'
OPENSEARCH_INITIAL_ADMIN_PASSWORD=******
EOF
```

### 🔹 Paso 4. Crear archivo docker-compose.yml
Este archivo define los servicios de Docker que se ejecutarán: el motor de OpenSearch y la interfaz de Dashboards.
```bash
yaml
services:
  opensearch:
    image: opensearchproject/opensearch:2
    container_name: opensearch
    environment:
      - cluster.name=opensearch-docker-cluster
      - node.name=node-1
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g"
      - OPENSEARCH_INITIAL_ADMIN_PASSWORD=${OPENSEARCH_INITIAL_ADMIN_PASSWORD}
    ulimits:
      memlock:
        soft: -1
        hard: -1
    ports:
      - "9200:9200"
      - "9600:9600"

  opensearch-dashboards:
    image: opensearchproject/opensearch-dashboards:2
    container_name: opensearch-dashboards
    environment:
      - OPENSEARCH_HOSTS=["https://opensearch:9200"]
      - OPENSEARCH_USERNAME=admin
      - OPENSEARCH_PASSWORD=${OPENSEARCH_INITIAL_ADMIN_PASSWORD}
      - SERVER_HOST=0.0.0.0
    ports:
      - "5601:5601"
    depends_on:
      - opensearch
```

📘 Explicación del código docker-compose.yml
🔹 Servicio opensearch
- image: especifica la imagen oficial de OpenSearch versión 2.x.
- container_name: nombre identificador del contenedor.
- environment:
  - cluster.name → define el nombre del clúster.
  - node.name → nombre del nodo único.
  - discovery.type=single-node → indica que se ejecutará en modo de un solo nodo.
  - bootstrap.memory_lock=true → bloquea la memoria para evitar que el sistema use swap.
  - "OPENSEARCH_JAVA_OPTS=-Xms1g -Xmx1g" → asigna 1 GB de memoria a la JVM.
  - OPENSEARCH_INITIAL_ADMIN_PASSWORD=${...} → toma la contraseña desde el archivo .env.

- ulimits.memlock: evita limitaciones de memoria.
- ports:
    - 9200 → puerto principal de la API REST de OpenSearch.
    - 9600 → puerto para métricas internas.

🔹 Servicio "opensearch-dashboards"
- image: imagen oficial de la interfaz gráfica (similar a Kibana).
- environment:
  - OPENSEARCH_HOSTS → URL del servicio principal.
  - OPENSEARCH_USERNAME / OPENSEARCH_PASSWORD → credenciales para conectarse.
  - SERVER_HOST=0.0.0.0 → expone el servicio en todas las interfaces.

- ports:
  - 5601 → puerto del panel web de Dashboards.

- depends_on: garantiza que el contenedor opensearch se inicie primero.

### 🔹 Paso 5. Levantar los contenedores
```bash
docker compose pull
docker compose up -d
docker compose ps
```
Verificar que OpenSearch está corriendo:

```bash
curl -sk -u admin:Lobo.1014# https://localhost:9200 | jq .
```
Acceder al panel de control:
```bash
👉 http://localhost:5601
```
