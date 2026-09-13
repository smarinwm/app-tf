# Despliegue de una aplicación Node.js en AWS EC2 con Terraform y Docker

Proyecto de práctica para desplegar una aplicación **Node.js + Express** en una instancia **Amazon EC2** utilizando **Terraform** como infraestructura como código y **Docker** para empaquetar la aplicación.

El repositorio permite trabajar un flujo sencillo de aprovisionamiento en AWS: Terraform crea la infraestructura, la instancia EC2 ejecuta un script de inicialización y la aplicación se publica en un contenedor Docker accesible por HTTP.

> Este repositorio parte de un ejemplo previo cuyo `package.json` referencia el proyecto `piumsudhara/ECS-Terraform`. Se presenta aquí como material de práctica y adaptación técnica, no como proyecto original íntegramente propio.

## Tecnologías utilizadas

- **Terraform**
- **Amazon Web Services (AWS)**
- **Amazon EC2**
- **AWS Security Groups**
- **Docker**
- **Node.js**
- **Express**
- **Linux / Bash**
- **Infrastructure as Code (IaC)**

## Arquitectura

El flujo general del proyecto es:

```text
Terraform
   │
   ├── Security Group
   │      ├── HTTP :80
   │      └── SSH  :22
   │
   └── Amazon EC2
          │
          └── user_data
                 ├── Instala Docker y Git
                 ├── Clona el repositorio
                 ├── Construye la imagen Docker
                 └── Ejecuta la aplicación Node.js
```

La aplicación Express escucha en el puerto `80` y devuelve una respuesta JSON sencilla:

```json
{
  "response": "Hi, Amazon Web Service"
}
```

## Estructura del repositorio

```text
app-tf/
├── app/
│   ├── package.json
│   └── server.js
├── Dockerfile
├── main.tf
└── README.md
```

### `main.tf`

Define la infraestructura AWS con Terraform:

- Provider de AWS.
- Región `us-east-1`.
- Security Group.
- Acceso HTTP por el puerto `80`.
- Acceso SSH por el puerto `22`.
- Instancia EC2 `t3.micro`.
- Script `user_data`.
- Output con la IP pública de la instancia.

### `Dockerfile`

Construye una imagen para la aplicación Node.js:

```dockerfile
FROM node:14-alpine
WORKDIR /app
COPY app/package*.json ./
RUN npm install
COPY app/ ./
EXPOSE 80
CMD ["npm", "start"]
```

### `app/server.js`

Implementa una API mínima con Express y expone:

```text
GET /
```

## Requisitos

Para utilizar el proyecto necesitas:

- Una cuenta de **AWS**.
- **Terraform** instalado.
- Credenciales de AWS configuradas localmente.
- Permisos para crear instancias EC2 y Security Groups.
- Docker únicamente si quieres probar la imagen en local.

Comprueba Terraform con:

```bash
terraform version
```

y que tus credenciales AWS estén disponibles mediante el mecanismo que utilices habitualmente, por ejemplo AWS CLI o variables de entorno.

## Despliegue con Terraform

### 1. Clonar el repositorio

```bash
git clone https://github.com/smarinwm/app-tf.git
cd app-tf
```

### 2. Inicializar Terraform

```bash
terraform init
```

### 3. Revisar el plan

```bash
terraform plan
```

### 4. Crear la infraestructura

```bash
terraform apply
```

Terraform mostrará los cambios previstos antes de solicitar confirmación.

Cuando termine, el output:

```text
public_ip
```

mostrará la dirección pública de la instancia EC2.

La aplicación debería quedar accesible mediante:

```text
http://<IP_PUBLICA>
```

### 5. Destruir la infraestructura

Cuando hayas terminado las pruebas:

```bash
terraform destroy
```

Esto evita mantener recursos EC2 activos innecesariamente.

## Ejecución local con Docker

También puedes comprobar la aplicación sin desplegar AWS:

```bash
docker build -t app-tf .
docker run --rm -p 8080:80 app-tf
```

Después abre:

```text
http://localhost:8080
```

## Aspectos a revisar antes de utilizar el proyecto

El repositorio tiene finalidad **didáctica** y contiene varios puntos que conviene corregir antes de utilizarlo en un entorno real.

### Acceso SSH abierto

Actualmente el Security Group permite SSH desde cualquier dirección:

```hcl
cidr_blocks = ["0.0.0.0/0"]
```

Esto no es recomendable en producción.

Lo adecuado sería limitar el puerto `22` a una IP o red concreta, o evitar SSH público utilizando alternativas como **AWS Systems Manager Session Manager**.

### Ruta del repositorio en `user_data`

El script actual contiene:

```bash
git clone https://github.com/smarinwm/app-tf.git
cd mi-app-tf
```

El directorio creado por `git clone` normalmente será `app-tf`, por lo que esa ruta debería revisarse antes de ejecutar el despliegue:

```bash
cd app-tf
```

### Imagen de Node.js

El `Dockerfile` utiliza:

```dockerfile
FROM node:14-alpine
```

Node.js 14 está fuera de soporte. Para una actualización del proyecto conviene migrar a una versión LTS actual y verificar previamente la compatibilidad de la aplicación.

### AMI y comandos de inicialización

La AMI está fijada directamente en `main.tf`:

```hcl
ami = "ami-053a45fff0a704a47"
```

Los identificadores AMI son regionales y pueden quedar obsoletos.

También conviene comprobar que los comandos de instalación incluidos en `user_data`, como `amazon-linux-extras`, sean compatibles con la versión concreta de Amazon Linux utilizada.

Una mejora sería resolver la AMI dinámicamente mediante un `data source` de Terraform.

### Configuración de Terraform

Para evolucionar el proyecto sería recomendable separar valores configurables mediante variables, por ejemplo:

- Región AWS.
- Tipo de instancia.
- AMI.
- CIDR permitido para SSH.
- Nombre del proyecto.
- Tags.

También sería conveniente añadir:

```text
variables.tf
outputs.tf
versions.tf
terraform.tfvars.example
.gitignore
```

y no versionar nunca:

```text
.terraform/
*.tfstate
*.tfstate.*
```

## Seguridad y costes

Este proyecto crea infraestructura real en AWS y puede generar costes.

Antes de ejecutar `terraform apply`:

- Revisa siempre `terraform plan`.
- Verifica la región y el tipo de instancia.
- No almacenes claves AWS en el repositorio.
- Utiliza credenciales con permisos mínimos.
- Restringe el acceso SSH.
- Elimina los recursos al terminar con `terraform destroy`.
- Comprueba en la consola de AWS que no queden recursos activos.

## Objetivo didáctico

Este repositorio permite practicar:

- Infrastructure as Code con Terraform.
- Aprovisionamiento de EC2.
- Security Groups.
- Scripts de inicialización mediante `user_data`.
- Dockerización de aplicaciones.
- Node.js y Express.
- Despliegue básico de aplicaciones en AWS.
- Automatización de infraestructura.
- Gestión del ciclo de vida con `terraform init`, `plan`, `apply` y `destroy`.

## Perfil

**Silverio Marín** — Docente TIC en Valencia, especializado en cloud computing, automatización e infraestructura como código.

Más contenidos sobre **cloud computing, AWS y automatización**:

**[silveriomarin.com/cloud](https://silveriomarin.com/cloud/)**

GitHub: **[@smarinwm](https://github.com/smarinwm)**
