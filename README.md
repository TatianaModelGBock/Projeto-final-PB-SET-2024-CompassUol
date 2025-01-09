# Projeto Final PB - Compass Uol

## Cloud Pratictioner - PB - SET 2024

## Contexto
A **Fast Engineering S/A** enfrenta desafios de escalabilidade e disponibilidade para seu e-commerce. Atualmente, todo o stack roda **on-premises**, mas o crescimento da demanda exige uma arquitetura mais flexível e resiliente.  
A proposta é migrar o ambiente para a **AWS** em duas fases:

1. **Fase 1**: Migração “Lift and Shift” (o mais rápido possível, mantendo a aplicação como está, com o mínimo de modernização).  
2. **Fase 2**: Modernização com contêineres (EKS), banco gerenciado (RDS), armazenamento de objetos (S3), reforço de segurança etc.

---

## Arquitetura Atual (On-Premises)

- **db01**: Servidor MySQL (500GB, 10GB RAM, 3 vCPUs)  
- **front01**: Frontend em React (5GB, 2GB RAM, 1 vCPU)  
- **back01**: Backend com 3 APIs + Nginx (5GB, 4GB RAM, 2 vCPUs)  
- **Nginx** também atua como balanceador de carga e armazenamento de estáticos (imagens, arquivos, etc.).

## Diagrama da arquitetura:
![diagrama](https://github.com/TatianaModelGBock/Projeto-final-PB-SET-2024-CompassUol/blob/main/images/arq-on-premise.jpg)

---

## Fase 1: Migração “As-Is” (Lift and Shift)

### Visão Geral
Na primeira fase, a ideia é levar rapidamente os servidores on-premises para a AWS, **sem grandes refatorações** de código ou mudança de arquitetura.  
Ferramentas utilizadas:
- **AWS MGN (Application Migration Service)**: realiza a replicação do servidor (sistema operacional, aplicações) para instâncias EC2.  
- **AWS Replication Agent**: realiza uma migração segura de dados para dentro de uma subrede migratória.
- **AWS EBS (Elastic Block System)**: armzenará os arquivos persistentes do banco de dados.
- **AWS DMS (Database Migration Service)**: Migrará o banco de dados para uma estrutura RDS.

### Passo a Passo de Migração
1. **Planejamento**  
   - Inventariar serviços e dependências (versão do SO, portas, bibliotecas, integrações externas).

2. **Provisionamento de Infraestrutura AWS**  
   - **Criar VPC** com subnets públicas e privadas.  
   - **Internet Gateway** (IGW) para subnets públicas. 
   - **Security Groups**:  
     - Acesso restrito ao MySQL (porta 3306) só a partir do backend.  
     - Acesso ao backend (porta 80/443).

3. **Migração do Banco de Dados**
   - **DMS** Serviço de migração de banco de dados.
   - **RDS MySQL** (Lift and Shift com modernização mínima para preparação de ambiente).  
   - Copiar dados via **AWS Replication Agent** (TCP 1500).  
   - Testar **integridade** e **performance** do banco no ambiente de destino.

5. **Migração de Frontend/Backend**  
   - Usar **AWS MGN** para replicar as máquinas on-premises em instâncias EC2.  
   - Usar **AWS EBS** para hospedar os arquivos advindos das máquinas

6. **Teste e Validação**  
   - Apontar subdomínio (ex.: `test.minhaempresa.com`) para o IP ou ALB da aplicação na AWS.  
   - Verificar logs, monitorar performance.


## Diagrama “Lift-and-Shift"

![[diagrama2](https://github.com/TatianaModelGBock/Projeto-final-PB-SET-2024-CompassUol/blob/main/images/ArqLiftAndShift%20(1).drawio.png)](https://github.com/TatianaModelGBock/Projeto-final-PB-SET-2024-CompassUol/blob/main/images/ArqLiftAndShift.drawio%20(1).png)

---

## **Fase 2: Modernização**

### **Objetivo**
Após a migração inicial, modernizar a arquitetura seguindo boas práticas de nuvem, com foco em escalabilidade, segurança e facilidade de gerenciamento.

---
# Componentes Principais

### DNS e Distribuição de Conteúdo:
- **Route 53**: Gerenciamento de DNS.
- **CloudFront**: Distribuição de conteúdo para otimizar o acesso global.
- **WAF (Web Application Firewall)**: Proteção contra ataques como SQL Injection e Cross-Site Scripting.

### Orquestração de Contêineres:
- **EKS Managed Cluster**: Gerencia os clusters Kubernetes, reduzindo a necessidade de configuração manual.
- **Auto Scaling Group**: Escala os nós do cluster automaticamente com base na demanda.
- **Horizontal Pod Autoscaler**: Ajusta a quantidade de pods com base no consumo de recursos, como CPU e memória.

### Redes:
- **Subnets Públicas e Privadas**:
- **NAT Gateway**: Permite que instâncias privadas acessem a Internet.
- **Load Balancer (ALB)**: Distribui o tráfego entre os pods e garante alta disponibilidade.

### Banco de Dados:
- **RDS MySQL com Réplica**: Banco de dados relacional para armazenar dados persistentes com replicação Multi-AZ para alta disponibilidade.

### Segurança:
- **AWS Secrets Manager**: Gerencia segredos, como credenciais de banco de dados.
- **AWS KMS**: Gerenciamento de chaves para criptografia.
- **IAM Roles**: Configuração de roles e policies para controle de acesso.

### Armazenamento de Objetos:
- **S3**: Armazena artefatos e arquivos estáticos, como imagens e vídeos.

### Monitoramento e Armazenamento:
- **CloudWatch**: Monitora métricas de infraestrutura e aplicações.
- **S3**: Armazena artefatos e backups.

### CI/CD e Controle de Código:
- **CodeCommit**: Repositório de código fonte.
- **CodeBuild e CodePipeline**: Automação do processo de construção e implantação.
- **IAM Roles**: Controle de permissões para desenvolvedores e equipes.

# Passo a Passo da Modernização

## 1. Planejamento
- **Mapeamento de serviços**:
  - Identificar componentes da aplicação e suas dependências.
  - Validar compatibilidade com contêineres (Docker).
- **Desenho de arquitetura**:
  - Criar diagramas e fluxos com a nova infraestrutura.
  - Definir requisitos de rede, segurança e escalabilidade.

## 2. Criação do Cluster EKS
- Configurar um cluster gerenciado pela AWS utilizando o EKS:
  - Criar subnets públicas e privadas para os nós do cluster.
  - Associar um NAT Gateway para permitir comunicação segura com serviços externos.
- **Configurar permissões**:
  - Associar roles do IAM para nós do cluster e pods.

## 3. Contêinerização da Aplicação
- Criar Dockerfiles para cada serviço da aplicação:
  - Backend (API).
  - Frontend (interface).
- Armazenar as imagens no **Amazon ECR (Elastic Container Registry)**.

## 4. Configuração do Kubernetes
- Criar manifests YAML para os seguintes recursos:
  - **Deployments**:
    - Configurar réplicas para alta disponibilidade.
    - Associar pods a volumes persistentes para dados críticos.
  - **Services**:
    - Configurar serviços para expor os pods internamente (ClusterIP) e externamente (LoadBalancer).
  - **Ingress**:
    - Utilizar o ALB para rotear tráfego HTTP/HTTPS.
  - **Secrets e ConfigMaps**:
    - Gerenciar credenciais e configurações externas.

## 5. Implementação de Melhoria de Segurança
- **Reforço de Segurança**:
  - Configuração de IAM roles e policies.
  - Implementação de WAF para proteção contra ataques.

## 6. Backup Automatizado
- Utilizar snapshots do RDS e backups automáticos do S3.

## 7. CI/CD e Automação
- **Pipeline CI/CD**:
  - Configurar o **CodePipeline** para automatizar a construção e implantação das imagens Docker.
  - Utilizar o **CodeBuild** para criar e publicar imagens no ECR.
  - Implantar as atualizações no cluster EKS automaticamente.
- **CloudFormation**:
  - Utilizar CloudFormation para automatizar a infraestrutura e deployments.

## 8. Monitoramento e Logs
- Configurar **CloudWatch Logs**:
  - Capturar logs dos pods e métricas do cluster.
- Utilizar ferramentas de monitoramento como **Prometheus** e **Grafana**:
  - Visualizar métricas personalizadas de aplicação e infraestrutura.

