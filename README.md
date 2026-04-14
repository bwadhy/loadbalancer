# 🚀 Arquitetura AWS - Alta Disponibilidade com ALB, EC2 e S3

## 📌 Visão Geral

Este projeto demonstra a criação de uma arquitetura web altamente disponível na AWS utilizando:

* Application Load Balancer (ALB)
* EC2 distribuídas em múltiplas AZs
* Integração com Amazon S3 via IAM Role
* Roteamento por path (/red e /blue)
* Health checks e failover automático

---

## 🧠 Arquitetura

* 2 Availability Zones (us-east-1a e us-east-1b)
* 4 instâncias EC2:

  * 2 RED
  * 2 BLUE
* 2 Target Groups:

  * tg-red
  * tg-blue
* 1 Application Load Balancer

### Fluxo:

```
Cliente → ALB → (Path /red ou /blue) → Target Group → EC2
```

---

## ⚙️ Tecnologias utilizadas

* Amazon EC2
* Application Load Balancer (ALB)
* Amazon S3
* IAM Roles
* Apache (httpd)
* Systems Manager (SSM) (teste)

---

## 🔐 Segurança

* Acesso às EC2 via SSM (sem uso de chave SSH)
* IAM Role para acesso ao S3 (sem credenciais hardcoded)
* Security Groups controlando acesso HTTP
  
  JSON da role para acessar o s3:
  {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::s3bucketlab13abril"
    },
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::s3bucketlab13abril/*"
    }
  ]
}
---

## 📦 Configuração das Instâncias

Cada EC2:

* Instala Apache automaticamente via User Data
* Baixa arquivos do S3
* Serve páginas distintas:
## 📦 Objetos armazenados no S3

Os seguintes arquivos foram utilizados para compor as páginas web servidas pelas instâncias EC2:

### 🔴 RED

* `red-index.html`
* `red-root-index.html`
* `hw-red.css`
* `hw-red-py.css`
* `python.png`
* `apache.svg`

### 🔵 BLUE

* `blue-index.html`
* `blue-root-index.html`
* `hw-blue.css`
* `hw-blue-py.css`
* `python.png`
* `apache.svg`

---

## ⚙️ User Data das Instâncias

As instâncias EC2 foram configuradas automaticamente via **User Data**, realizando:

* Instalação do Apache
* Download dos arquivos do S3
* Configuração das páginas web

---

### 🔴 User Data - RED

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
systemctl enable httpd

mkdir -p /var/www/html/red
cd /var/www/html/red

aws s3 cp s3://s3bucketlab13abril/red-index.html ./index.html
aws s3 cp s3://s3bucketlab13abril/hw-red.css ./
aws s3 cp s3://s3bucketlab13abril/hw-red-py.css ./
aws s3 cp s3://s3bucketlab13abril/python.png ./
aws s3 cp s3://s3bucketlab13abril/apache.svg ./

cd /var/www/html
aws s3 cp s3://s3bucketlab13abril/red-root-index.html ./index.html

systemctl restart httpd
```
<img width="1917" height="941" alt="image" src="https://github.com/user-attachments/assets/53af01f9-78ad-45b0-96f2-79fb0beb1430" />

---

### 🔵 User Data - BLUE

```bash
#!/bin/bash
yum install -y httpd
systemctl start httpd
systemctl enable httpd

mkdir -p /var/www/html/blue
cd /var/www/html/blue

aws s3 cp s3://s3bucketlab13abril/blue-index.html ./index.html
aws s3 cp s3://s3bucketlab13abril/hw-blue.css ./
aws s3 cp s3://s3bucketlab13abril/hw-blue-py.css ./
aws s3 cp s3://s3bucketlab13abril/python.png ./
aws s3 cp s3://s3bucketlab13abril/apache.svg ./

cd /var/www/html
aws s3 cp s3://s3bucketlab13abril/blue-root-index.html ./index.html

systemctl restart httpd

```
<img width="1908" height="933" alt="image" src="https://github.com/user-attachments/assets/e4286795-0bcd-4f50-9c25-ce9b029097fc" />

---

## 💡 Observação

O uso de **User Data** permitiu que as instâncias fossem configuradas automaticamente no momento do provisionamento, sem necessidade de acesso manual via SSH.

Além disso, a integração com o S3 foi realizada via **IAM Role**, eliminando o uso de credenciais estáticas.


---

## 🔄 Load Balancing
![alb](https://github.com/user-attachments/assets/7adbdcaf-df48-4268-a2cd-fb8c83a08665)


O ALB realiza:

* Roteamento baseado em path:

  * `/red*` → tg-red
  ![tg-red](https://github.com/user-attachments/assets/1f400884-7730-451c-b7c7-428349b14412)

  
  * `/blue*` → tg-blue
  ![tg-blue](https://github.com/user-attachments/assets/56364a1d-578e-41bf-817a-ee76db150a8e)

* Distribuição de carga entre múltiplas instâncias
* Health check automático

---

## 🧪 Testes realizados

* Acesso via:

  * `/red`
  ![dnsred](https://github.com/user-attachments/assets/d24d1523-8ea1-4604-a6c2-2c52fcdc11c6)

  * `/blue`
  <img width="1888" height="761" alt="image" src="https://github.com/user-attachments/assets/25225053-918e-452c-9f1a-6865671c534a" />

* Interrupção de instância para validação de failover
* Verificação de health check

---

## 💡 Aprendizados

* Diferença entre roteamento e balanceamento
* Funcionamento de health checks no ALB
* Importância de padronizar instâncias
* Uso de IAM Role para acesso seguro ao S3
* Comportamento do ALB com targets unhealthy (fail-open)

---

## 🚀 Próximos passos

* Implementar Auto Scaling
* Fazer Host Based Routing e usar o Route 53
* Automatizar infraestrutura com IaC
* Adicionar HTTPS


---


Bruno Wadhy Campos

Projeto desenvolvido para fins de aprendizado em Cloud (AWS)

