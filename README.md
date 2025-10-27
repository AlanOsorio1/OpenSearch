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
OPENSEARCH_INITIAL_ADMIN_PASSWORD=PASSWORD
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
### 🔹 Paso 5. Levantar los contenedores
```bash
docker compose pull
docker compose up -d
docker compose ps
```
Verificar que OpenSearch está corriendo:

```bash
curl -sk -u admin:PASSWORD# https://localhost:9200 | jq .
```
Acceder al panel de control:
```bash
👉 http://localhost:5601
```
### 🔹 Paso 6. Creación del dataset personalizado
🔹 Descripción general

Se construyó un nuevo conjunto de datos llamado eventos_ventas, que simula registros de una empresa de tecnología con operaciones en varias ciudades de Colombia.
Cada documento representa un evento o transacción con atributos como:

- Fecha (@timestamp)
- Categoría (categoria)
- Producto (producto)
- Ciudad (lugar)
- Canal de venta (canal)
- Monto (monto)
- Detalle (detalle)

Este dataset permite visualizar comportamientos de ventas, devoluciones y servicios de soporte a lo largo del tiempo.

1️⃣ Crear el índice con estructura (mapping):

```bash
curl -sk -u admin:'PASSWORD' \
  -H "Content-Type: application/json" \
  -X PUT "https://localhost:9200/eventos_ventas" \
  -d '{
    "settings": { "number_of_shards": 1 },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "categoria":  { "type": "keyword" },
        "producto":   { "type": "keyword" },
        "lugar":      { "type": "keyword" },
        "canal":      { "type": "keyword" },
        "monto":      { "type": "float" },
        "detalle":    { "type": "text" }
      }
    }
  }'
```

Esto define la estructura del índice, indicando los tipos de datos y campos que se podrán consultar o filtrar.

2️⃣ Cargar datos simulados con Bulk (NDJSON)

Se creó un archivo eventos_ventas.ndjson con 30 registros simulados de diferentes tipos de evento (ventas, devoluciones, soporte, actualizaciones, etc.).

Ejemplo de las primeras líneas:
```bash
{ "index" : { "_index" : "eventos_ventas" } }
{ "@timestamp":"2025-10-01T09:15:00Z","categoria":"venta","producto":"Laptop","lugar":"Bogotá","canal":"Online","monto":850.5,"detalle":"Compra de laptop gama media" }
{ "index" : { "_index" : "eventos_ventas" } }
{ "@timestamp":"2025-10-02T14:00:00Z","categoria":"devolucion","producto":"Celular","lugar":"Cartagena","canal":"Tienda","monto":-620.0,"detalle":"Reembolso por garantía" }
```

Carga masiva en OpenSearch:
```bash
curl -sk -u admin:'PASSWORD' \
  -H "Content-Type: application/json" \
  -X POST "https://localhost:9200/_bulk" \
  --data-binary @eventos_ventas.ndjson
```

Verificar cantidad de documentos:
```bash
curl -sk -u admin:'PASSWORD' "https://localhost:9200/eventos_ventas/_count"
```
3️⃣ Crear Data View en OpenSearch Dashboards


### 🔹 Paso 6. Creación de visualizaciones y dashboard

Visualizaciones creadas:

- Barras	Transacciones por categoría	X: categoria, Y: Count
- Pastel	Distribución por producto	Campo: producto
- Línea	Evolución del monto total en el tiempo	X: @timestamp, Y: Sum(monto)
- Métrica	Total de ingresos acumulado	Sum(monto)

🎛️ Configuración de búsqueda facetada (Faceted Search)
Objetivo:
Permitir filtrar dinámicamente las visualizaciones combinando distintas categorías de datos, sin necesidad de modificar consultas manualmente.

