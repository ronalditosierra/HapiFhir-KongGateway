# 🏥 API Gateway con Kong y HAPI FHIR

Este proyecto implementa un entorno de pruebas para servicios **FHIR (Fast Healthcare Interoperability Resources)** usando **HAPI FHIR** como servidor de datos clínicos y **Kong Gateway** como API Gateway.

## 🚀 Objetivos principales

* Desplegar un servidor FHIR (HAPI FHIR) en contenedores con Docker.
* Integrar un API Gateway con **Kong** para controlar el acceso.
* Configurar políticas de **rate limiting** para proteger los recursos.
* Realizar pruebas con peticiones válidas e inválidas para validar la seguridad y estabilidad del sistema.

## 🛠️ Tecnologías utilizadas

* **Docker & Docker Compose** 🐳
* **Kong Gateway** ⚡
* **HAPI FHIR** 🏥
* **cURL** para pruebas de endpoints

---

## 📂 Estructura básica del proyecto

```bash
Project/
├── docker-compose.yml
├── kong.yml
├── invalid-observation.json
├── valid-observation.json
├── patient.json
└── README.md
```

---

## ⚙️ Pasos de instalación y despliegue

### 1. Clonar el repositorio

```bash
git clone <url-del-repo>
cd Project
```

### 2. Levantar los contenedores

```bash
docker compose up -d
```

### 3. Verificar servicios activos

```bash
docker ps
```

Debes ver al menos:

* `kong`
* `hapi-fhir`

---

## 🔬 Pruebas con recursos FHIR

### 1. Crear un paciente

El archivo `patient.json`:

```json
{
  "resourceType": "Patient",
  "id": "1",
  "name": [
    {
      "use": "official",
      "family": "Doe",
      "given": ["John"]
    }
  ],
  "gender": "male",
  "birthDate": "1980-01-01"
}
```

Ejecuta:

```bash
curl -X POST "http://localhost:8000/fhir/Patient" \
  -H "Content-Type: application/fhir+json" \
  --data-binary @patient.json
```

### 2. Crear una Observation válida

El archivo `valid-observation.json`:

```json
{
  "resourceType": "Observation",
  "status": "final",
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "8480-6",
        "display": "Systolic blood pressure"
      }
    ]
  },
  "subject": {
    "reference": "Patient/1"
  },
  "valueQuantity": {
    "value": 120,
    "unit": "mmHg",
    "system": "http://unitsofmeasure.org",
    "code": "mm[Hg]"
  }
}
```

Ejecuta:

```bash
curl -X POST "http://localhost:8000/fhir/Observation" \
  -H "Content-Type: application/fhir+json" \
  --data-binary @valid-observation.json
```

### 3. Crear una Observation inválida

El archivo `invalid-observation.json` (valor incorrecto en `valueQuantity`):

```json
{
  "resourceType": "Observation",
  "status": "final",
  "code": {
    "coding": [
      {
        "system": "http://loinc.org",
        "code": "8480-6",
        "display": "Systolic BP"
      }
    ]
  },
  "valueQuantity": {
    "value": "ABC"
  }
}
```

Ejecuta:

```bash
curl -X POST "http://localhost:8000/fhir/Observation/$validate" \
  -H "Content-Type: application/fhir+json" \
  --data-binary @invalid-observation.json
```

---

## 🔄 Ciclos de prueba con Rate Limiting

### Prueba con Observation inválida

```bash
for i in {1..20}
do
  echo "Request $i"
  curl -s -o /dev/null -w "%{http_code}\\n" \
    -X POST "http://localhost:8000/fhir/Observation/$validate" \
    -H "Content-Type: application/fhir+json" \
    --data-binary @invalid-observation.json
done
```

### Prueba con Patient existente

```bash
for i in {1..20}
do
  echo "Request $i"
  curl -s -o /dev/null -w "%{http_code}\\n" http://localhost:8000/fhir/Patient/1
done
```

---

## 📖 Notas

* Si ves errores de permisos con Docker, asegúrate de que tu usuario está en el grupo `docker`:

```bash
sudo usermod -aG docker $USER && newgrp docker
```

* Los endpoints de prueba se acceden a través de Kong en **[http://localhost:8000](http://localhost:8000)**.

---

## 📜 Licencia

Este proyecto es de uso libre con fines educativos y de prueba.

