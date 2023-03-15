---
layout: default
title: Prácticas 13.5
parent: Prácticas 13
nav_order: 5
---
# IAW - Práctica 13.5 Terraform
![](../assets/pr135/terraform.png)
# ¿Qué es Terraform?
[Terraform](https://www.terraform.io/) es una herramienta de [infraestructura como código](https://es.wikipedia.org/wiki/Infraestructura_como_c%C3%B3digo) (Infraestructure as Code, IaC) que permite crear, modificar y eliminar infraestructura de forma automática.

[Terraform](https://www.terraform.io/) puede gestionar los recursos de diferentes proveedores de servicios en la nube, como [AWS](https://aws.amazon.com/es/), [Google Cloud](https://cloud.google.com/), [Azure](https://azure.microsoft.com/es-es/), etc.

# Comandos Básicos
## Inicializar el directorio de trabajo
En primer lugar hay que descargar los plugins necesarios del proveedor que se especifica en el archivo de configuración.

    terraform init
## Formatear y validar el archivo de configuración
Terraform proporciona un comando para formatear el archivo de configuración para que sea más legible. Algunas de las tareas que realiza este comando son, ajustar la indentación, ordenar los argumentos de los bloques de configuración, etc.
```
terraform fmt
```



Para validar la sintaxis del archivo de configuración es posible utilizar el siguiente comando.
```
terraform validate
```
## Mostrar los cambios que se van a realizar
Compara la configuración del archivo de Terraform con la que existe actualmente en el proveedor de infraestructura y muestra las acciones que se tienen que realizar para conseguir la configuración deseada. Permite al usuario verificar los cambios antes de aplicarlos en el proveedor.
```
terraform plan
```
## Aplicar los cambios
Crea los recursos del archivo de configuración en su cuenta de AWS.
```
terraform apply
```
Si se desea crear los recursos sin tener que escribir yes para confirmar la ejecución del comando, podemos utilizar la opción -auto-approve.
```
terraform apply -auto-approve
```
## Mostrar el estado actual de los recursos
Muestra los recursos creados en el proveedor y su estado actual.
```
terraform show
```

## Eliminar los recursos
Elimina los recursos indicados en el proveedor.
```
terraform destroy
```
# Estructura
En esta práctica se realizó la creación de la infraestructura necesaria para desplegar las aplicaciones web propuetas en la práctica 7 y 9 utilizando las funcionalidades de **Terraform**.

```txt
.
├── imgs
│   └── terraform.png
├── practica7
│   ├── main.tf
│   ├── terraform.tfstate
│   └── terraform.tfstate.backup
├── practica9
│   ├── main.tf
│   └── terraform.tfstate
└── README.md
```
Ambos directorios, **practica7** y **practica9** contienen un main.tf  para montar la infrastructura necesaria.

# Práctica 7
```terraform
# Configuramos el proveedor de AWS
provider "aws" {
  region = "us-east-1"
}

# Creamos un grupo de seguridad para el frontend
resource "aws_security_group" "frontend_sg" {
  name        = "FrontEndSecurityGroup"
  description = "Grupo de seguridad para las maquinas FrontEnd"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Creamos un grupo de seguridad para el backend
resource "aws_security_group" "backend_sg" {
  name        = "BackEndSecurityGroup"
  description = "Grupo de seguridad para las maquinas BackEnd"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Creamos una instancia EC2 Frontend
resource "aws_instance" "instancia_frontend" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.small"
  key_name        = "vockey"
  security_groups = [aws_security_group.frontend_sg.name]

  tags = {
    Name = "FrontEnd"
  }
}

# Creamos una instancia EC2 BackEnd
resource "aws_instance" "instancia_backend" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.small"
  key_name        = "vockey"
  security_groups = [aws_security_group.backend_sg.name]

  tags = {
    Name = "BackEnd"
  }
}

# Creamos una IP elástica y la asociamos a la instancia
resource "aws_eip" "ip_elastica" {
  instance = aws_instance.instancia_backend.id
}

# Mostramos la IP pública de la instancia
output "elastic_ip" {
  value = aws_eip.ip_elastica.public_ip
}
```
# Práctica 9
```terraform
# Configuramos el proveedor de AWS
provider "aws" {
  region = "us-east-1"
}

# Creamos un grupo de seguridad para el frontend
resource "aws_security_group" "frontend_sg" {
  name        = "FrontEndSecurityGroup"
  description = "Grupo de seguridad para las maquinas FrontEnd"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Creamos un grupo de seguridad para el backend
resource "aws_security_group" "backend_sg" {
  name        = "BackEndSecurityGroup"
  description = "Grupo de seguridad para las maquinas BackEnd"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Creamos un grupo de seguridad para el load balancer
resource "aws_security_group" "balancer_sg" {
  name        = "LoadBalancerSecurityGroup"
  description = "Grupo de seguridad para el balanceador de carga"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Creamos un grupo de seguridad para el servidor NFS
resource "aws_security_group" "nfs_sg" {
  name        = "NFSServerSecurityGroup"
  description = "Grupo de seguridad para el servidor NFS"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 2049
    to_port     = 2049
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

# Creamos una instancia EC2 balanceador de carga
resource "aws_instance" "instancia_balancer" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.medium"
  key_name        = "vockey"
  security_groups = [aws_security_group.balancer_sg.name]

  tags = {
    Name = "LoadBalancer"
  }
}

# Creamos una instancia EC2 Frontend01
resource "aws_instance" "instancia_frontend01" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.small"
  key_name        = "vockey"
  security_groups = [aws_security_group.frontend_sg.name]

  tags = {
    Name = "FrontEnd_01"
  }
}

# Creamos una instancia EC2 Frontend02
resource "aws_instance" "instancia_frontend02" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.small"
  key_name        = "vockey"
  security_groups = [aws_security_group.frontend_sg.name]

  tags = {
    Name = "FrontEnd_02"
  }
}


# Creamos una instancia EC2 BackEnd
resource "aws_instance" "instancia_backend" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.small"
  key_name        = "vockey"
  security_groups = [aws_security_group.backend_sg.name]

  tags = {
    Name = "BackEnd"
  }
}

# Creamos una instancia EC2 servidor NFS
resource "aws_instance" "instancia_nfs" {
  ami             = "ami-00874d747dde814fa"
  instance_type   = "t2.medium"
  key_name        = "vockey"
  security_groups = [aws_security_group.nfs_sg.name]

  tags = {
    Name = "NFSServer"
  }
}

# Creamos una IP elástica y la asociamos a la instancia
resource "aws_eip" "ip_elastica" {
  instance = aws_instance.instancia_balancer.id
}

# Mostramos la IP pública de la instancia
output "elastic_ip" {
  value = aws_eip.ip_elastica.public_ip
}
```
