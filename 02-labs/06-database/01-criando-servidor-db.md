# 🧪 Criação e Uso de Banco de Dados no Amazon RDS

## 🔍 Visão Geral

- *Data:* 27/11/2025
- *Nome do Lab:* 160--Lab - Crie seu servidor de banco de dados e interaja usando um web app
- *Plataformas*: AWS re/start (Canvas) 
- *Serviços AWS:*  Amazon RDS
- *Objetivo:* 
	- Executar uma instância de banco de dados do Amazon RDS com alta disponibilidade.
	- Configurar a instância de banco de dados para permitir conexões do seu servidor web.
	- Abrir um aplicativo web e interagir com seu banco de dados.

---

## 🧩 Problema a Ser Resolvido

Uma aplicação web precisa armazenar dados de forma persistente e confiável, mas não pode depender de um banco de dados instalado manualmente em um servidor, pois isso aumenta a complexidade operacional, o risco de falhas e o tempo de indisponibilidade. Além disso, a aplicação precisa continuar funcionando mesmo se ocorrer uma falha em uma Zona de Disponibilidade.

---
## 🏗️ Arquitetura da Solução

- **Arquitetura da aplicação**
    - Aplicação web rodando em uma instância **EC2**
    - Acesso a um **banco de dados relacional** gerenciado pelo **Amazon RDS**.
    - Toda a infraestrutura está dentro da **mesma VPC**.

- **Banco de dados**
    - Implantado em **subnets privadas** distribuídas em **duas Zonas de Disponibilidade**.
    - Configuração **Multi-AZ** para **alta disponibilidade**.
    - **Replicação automática** dos dados entre as Zonas de Disponibilidade.

- **Segurança e acesso**
    - **Security Groups** controlam o acesso ao banco de dados.
    - Conexões permitidas **apenas a partir da instância EC2** do servidor web.

- **Conectividade**
    - Aplicação conecta ao banco via **endpoint fornecido pelo RDS**.
    - Endpoint abstrai a instância física e garante **continuidade de conexão em caso de failover**.

---
## 🖼️ Diagrama de Arquitetura

- Como a internet acessa apenas o Web Server na EC2, por estar numa subnet pública e ter o SG que permite entrada através da porta 80 de qualquer origem;

- O banco está na subnet privada, só acessível internamente, e nesse caso, só acessível pelo SG do Web Server, para ainda mais segurança;

- O Nat Gateway na subnet pública permite que os recursos das subnets privadas acessam a internet sem se expor, ideal para que o RDS realize seus gerenciamentos automáticos;

- O banco de dados é implantado em configuração Multi-AZ, com uma instância primária em uma Zona de Disponibilidade e uma instância de espera em outra, garantindo alta disponibilidade com failover automático.

	![](06-lab-rds-diagrama.jpg)

---

## 🧰 Serviços Utilizados e Justificativa

### ==Serviço AWS #1 - Amazon RDS (MySQL, Multi-AZ)==
- **Função:** Fornecer um banco de dados relacional gerenciado 
- **Por que foi escolhido:** Elimina a necessidade de gerenciar servidor, backups e failover, além alta disponibilidade.
- **Benefício principal:**  Alta disponibilidade com failover automático e simplicidade operacional.
### ==Serviço AWS #2 - Amazon VPC==
- **Função:**  Isolar e controlar a rede onde a aplicação e o banco de dados estão implantados.
- **Por que foi escolhido:** Permite segmentação por subnets e controle via Security Groups.
- **Benefício principal:** Segurança e controle de acesso em nível de rede.
### ==Serviço AWS #3 - Amazon EC2==
- **Função:**  Hospedar a aplicação web que interage com o banco de dados.
- **Por que foi escolhido:** Permite executar uma aplicação de forma controlada dentro da VPC, com acesso direto ao banco de dados via rede privada.
- **Benefício principal:**  Flexibilidade para executar aplicações personalizadas e integração direta com serviços como RDS por meio de Security Groups.
---
## 🪜 Passo a Passo 

1. Criei um Security Group:
	- Esse será aplicado a instância do RDS
	- Ele só aceita acessos de origem do Security Group da Aplicação Web
	![](01-create-db-sg.png)

2. Criei um grupo de subnet para o Banco de Dados
	- Selecionei as duas AZs
	- Selecionei as duas subnets privadas, uma de cada AZ
	- É onde fica definido que o banco não será exposto a internet
	![](02-create-db-subnet.png)

3. Criei uma instância de um Banco de Dados Amazon RDS
	- Associei para conectar a nossa Lab VPC com nossas subnets criadas
	![](03-create-rds-db.png)

4. Acessei uma EC2 para conectar o banco a aplicação
	- A instância de EC2 permite acesso HTTP disponibilizando um Web App
	- Nele conseguimos usar as credenciais do banco criado para testar uma conexão
	![](04-test-db-conn.png)

	![](05-crud-db-example.png)

---
## 🔐 Segurança

A solução protege os dados e recursos principalmente por meio de isolamento de rede dentro da VPC. O banco de dados do Amazon RDS é implantado em subnets privadas, não sendo acessível diretamente pela internet. O acesso ao banco é controlado por Security Groups, permitindo conexões apenas da instância EC2 que executa a aplicação web.

As credenciais do banco de dados são utilizadas apenas pela aplicação, enquanto o gerenciamento do serviço é controlado por permissões do IAM. A comunicação entre a aplicação e o banco ocorre dentro da rede privada da AWS, reduzindo a superfície de ataque. Além disso, o Amazon RDS oferece suporte a criptografia dos dados em repouso e em trânsito, seguindo boas práticas de segurança da AWS.

---
## 💰 Custos

O custo da solução é composto principalmente pela instância do Amazon RDS e pela instância EC2 utilizada pela aplicação. No caso do RDS, o uso de configuração Multi-AZ aumenta o custo, pois mantém uma instância de banco de dados em espera em outra Zona de Disponibilidade.

Outros fatores que influenciam o custo incluem a classe da instância, o tipo e a quantidade de armazenamento provisionado, o tráfego de rede e o tempo em que os recursos permanecem ativos. Para ambientes de laboratório e teste, é comum desativar backups automatizados e utilizar instâncias de menor porte para reduzir custos.

---
