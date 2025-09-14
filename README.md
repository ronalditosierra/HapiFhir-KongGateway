# 🏥 Proyecto de Interoperabilidad con HAPI FHIR + Kong API Gateway

Este proyecto implementa un servidor **HAPI FHIR** para manejar recursos médicos y un **API Gateway con Kong** para exponer los endpoints y aplicar políticas como **Rate Limiting**.

---

## 📌 Requisitos previos

Antes de iniciar, asegúrate de tener instalado:

- **Docker** y **Docker Compose**
- **Minikube**
- **kubectl**
- **cURL** o **Postman** para pruebas

---

## ⚙️ Instalación de dependencias en Ubuntu/Debian

```bash
# Actualizar sistema
sudo apt update && sudo apt upgrade -y

# Instalar dependencias
sudo apt install ca-certificates curl gnupg lsb-release -y

# Añadir repositorio de Docker
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Instalar Docker y plugins
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Verificar instalación
docker --version
docker compose version
```

---

## 🚀 Levantar los servicios

Archivo `docker-compose.yml`:

```yaml
services:
  hapi-fhir:
    image: hapiproject/hapi:latest
    container_name: hapi-fhir
    restart: unless-stopped
    ports:
      - "8081:8080"

  kong:
    image: kong/kong-gateway:latest   # si no tienes licencia, usa 'kong:latest'
    container_name: kong
    restart: unless-stopped
    network_mode: "host"
    environment:
      KONG_DATABASE: "off"
      KONG_DECLARATIVE_CONFIG: /etc/kong/kong.yml
      KONG_PROXY_LISTEN: "0.0.0.0:8000"
      KONG_ADMIN_LISTEN: "0.0.0.0:8001"
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
    volumes:
      - ./kong.yml:/etc/kong/kong.yml:ro
```

Levantar contenedores:

```bash
docker compose up -d
```

Ver logs:

```bash
docker compose logs -f
```

---

## 🔗 Probar API HAPI FHIR

Crear un paciente válido:

```bash
curl -X POST http://localhost:8081/fhir/Patient   -H "Content-Type: application/json"   -d '{
        "resourceType": "Patient",
        "name": [{ "family": "Sierra", "given": ["Ronal"] }],
        "gender": "male",
        "birthDate": "1992-05-10"
      }'
```

Crear un **Observation válido**:

```json
{
  "resourceType": "Observation",
  "status": "final",
  "code": {
    "coding": [{
      "system": "http://loinc.org",
      "code": "3141-9",
      "display": "Body weight Measured"
    }]
  },
  "subject": {
    "reference": "Patient/1"
  },
  "valueQuantity": {
    "value": 70,
    "unit": "kg",
    "system": "http://unitsofmeasure.org",
    "code": "kg"
  }
}
```

Enviar con:

```bash
curl -X POST http://localhost:8081/fhir/Observation   -H "Content-Type: application/json"   -d @valid-observation.json
```

---

## ⚠️ Probar un **Observation inválido**

Archivo `invalid-observation.json`:

```json
{
  "resourceType": "Observation",
  "status": "invalid",
  "code": {
    "coding": [{
      "system": "http://loinc.org",
      "code": "XXXX",
      "display": "Invalid Code"
    }]
  },
  "subject": {
    "reference": "Patient/9999"
  },
  "valueQuantity": {
    "value": "not-a-number",
    "unit": "??",
    "system": "http://unitsofmeasure.org",
    "code": "??"
  }
}
```

Ejecutar:

```bash
curl -X POST http://localhost:8081/fhir/Observation   -H "Content-Type: application/json"   -d @invalid-observation.json
```

---

## 📊 Probar Rate Limiting con Kong

Ejemplo en **bash** con un ciclo `for` para enviar múltiples requests y gatillar el límite:

```bash
for i in {1..20}
do
  echo "Request $i"
  curl -s -o /dev/null -w "%{http_code}\n" http://localhost:8000/fhir/Patient/1
done
```

👉 Verás que después de cierto número de requests, Kong devolverá **429 Too Many Requests**.

---

## 🛠️ Comandos útiles

Ver logs de Kong:

```bash
docker logs -f kong
```

Ver logs de HAPI FHIR:

```bash
docker logs -f hapi-fhir
```

Listar contenedores activos:

```bash
docker ps
```

Parar servicios:

```bash
docker compose down
```

---

## 📌 Notas

- El puerto de HAPI FHIR se expone en **8081** (internamente corre en 8080).
- Kong expone:
  - **8000** → Proxy
  - **8001** → Admin API
- Puedes modificar la política de **Rate Limiting** desde el archivo `kong.yml`.

---

## ✨ Autor

Proyecto creado por **Ronal Sierra** 👨‍💻  
📍 Sincelejo, Sucre – Colombia  
