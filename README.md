🧪 Ejercicio práctico: OpenSearch con instalación local y Faceted Search
📍 Contexto

OpenSearch es una plataforma open source para búsqueda y analítica de datos, derivada de Elasticsearch.
El objetivo del ejercicio fue instalar OpenSearch localmente con Docker Compose, cargar datos de ejemplo, crear un dashboard con visualizaciones y habilitar filtros dinámicos tipo faceted search.

🎯 Objetivo

Implementar un entorno de OpenSearch en local, cargar un dataset de ejemplo y construir un dashboard interactivo con búsqueda facetada para explorar los datos.

🔧 Requisitos técnicos

Sistema operativo: Linux Ubuntu

Herramientas: Docker y Docker Compose

Contenedores:

opensearch

opensearch-dashboards

Datos de ejemplo: Sample web logs (dataset oficial de OpenSearch Dashboards)

🧱 1️⃣ Instalación local con Docker Compose
Paso 1. Instalar Docker y Docker Compose
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


Verificar instalación:

docker --version
docker compose version

Paso 2. Crear carpeta del proyecto
mkdir -p /home/alan_osorio/Documentos/BLEND/opensearch_lab
cd /home/alan_osorio/Documentos/BLEND/opensearch_lab

Paso 3. Crear archivo .env
cat > .env << 'EOF'
OPENSEARCH_INITIAL_ADMIN_PASSWORD=Lobo.1014#
EOF

Paso 4. Crear archivo docker-compose.yml
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

Paso 5. Levantar los contenedores
docker compose pull
docker compose up -d
docker compose ps


Verificar acceso:

curl -sk -u admin:Lobo.1014# https://localhost:9200 | jq .


Entrar al panel:
👉 http://localhost:5601

Usuario: admin
Contraseña: Lobo.1014#

📊 2️⃣ Ingesta de datos de ejemplo (Dataset Quickstart)

Dentro de OpenSearch Dashboards:

Ir a Home → Add sample data

Seleccionar Sample web logs → Add data

Esto crea el índice:
opensearch_dashboards_sample_data_logs

Verificar datos en Discover (ajustar el tiempo a Last 7 days).

🧮 3️⃣ Creación de visualizaciones
Visualización 1 — Barras (Visitas por país)

Tipo: Vertical Bar

Y-axis: Count

X-axis: Terms del campo geo.src

Guardar: Visitas por país

Visualización 2 — Línea (Promedio de bytes en el tiempo)

Tipo: Line

Y-axis: Average de bytes

X-axis: Date histogram con @timestamp

Guardar: Promedio de bytes

Visualización 3 — Métrica (Visitantes únicos)

Tipo: Metric

Métrica: Unique count de clientip

Guardar: Visitantes únicos

📈 4️⃣ Creación del Dashboard

Menú lateral → Dashboards → Create dashboard

Add from library → agregar las 3 visualizaciones anteriores

Save → Mi Dashboard Logs

🔍 5️⃣ Configurar Faceted Search

En el dashboard → Edit → Add panel → Controls
Agregar:

Options list: campo geo.src → “País”

Options list: campo machine.os.keyword → “Sistema Operativo”

Range slider: campo bytes → “Tamaño de Bytes”

Guardar.
Ahora puedes filtrar combinando varias facetas (por país, SO y rango de bytes).
