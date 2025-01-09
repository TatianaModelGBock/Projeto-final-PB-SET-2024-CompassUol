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

# Fase 1: Migração “As-Is” (Lift and Shift)

Nesta primeira fase, a meta é **migrar rapidamente** os servidores on-premises para a AWS, **sem** modificar a arquitetura ou refatorar código. No diagrama abaixo, ilustramos o fluxo de migração: o **AWS Replication Agent** envia os dados on-premises para uma **VPC Temporária (Staging)**, enquanto o **AWS MGN** converte as VMs em instâncias EC2. Paralelamente, o **AWS DMS** cuida da migração do banco de dados para **Amazon RDS**, garantindo integridade de dados.

## Visão Geral

- **AWS MGN (Application Migration Service)**: Replica os servidores (frontend, backend) para instâncias EC2 na AWS (Lift-and-Shift).  
- **AWS Replication Agent**: Agente responsável por enviar dados/arquivos do ambiente on-premises para a **VPC de staging**, onde o **Replication Server** processa e armazena em **EBS** temporariamente.  
- **AWS EBS (Elastic Block Store)**: Volumes que armazenam dados persistentes (tanto no staging quanto nas instâncias EC2 finais).  
- **AWS DMS (Database Migration Service)**: Migra o banco MySQL on-premises para **Amazon RDS** (MySQL).  
- **Load Balancer** (opcional): Distribui as requisições entre o Frontend (EC2) e o Backend (EC2), além de prover entrada segura (HTTP/HTTPS).  

## Passo a Passo de Migração

### 1. Planejamento

1. **Inventário**  
   - Catalogar os serviços rodando on-premises:  
     - Frontend (React)  
     - Backend (Nginx + APIs)  
     - Banco MySQL  
     - Versões de SO, bibliotecas, portas de rede (TCP 443, TCP 1500, TCP 3306 etc.).
2. **Janela de Manutenção e Riscos**  
   - Determinar quanto tempo de parada é tolerável.  
   - Planejar rollback (em caso de falha).

### 2. Provisionamento de Infraestrutura AWS

1. **Criar VPC de Staging (Temporária)**  
   - Subnet pública para receber o **Replication Server**.  
   - **Replication Server** irá receber dados on-prem (via AWS Replication Agent, porta TCP 1500).
2. **Criar VPC Final**  
   - **Subnets Públicas**: onde ficará o **Frontend EC2** e **Load Balancer**.  
   - **Subnets Privadas**: onde ficará o **Backend EC2** e o **RDS**.  
   - **Internet Gateway (IGW)** para as subnets públicas, **NAT Gateway** se as instâncias privadas precisarem acessar a internet.  
3. **Security Groups**  
   - Restringir porta 3306 (MySQL) para ser acessível somente pelo Backend.  
   - Permitir portas 80/443 (HTTP/HTTPS) vindas do Load Balancer para o Backend/Frontend.

### 3. Migração do Banco de Dados

1. **Provisionar Amazon RDS (MySQL)**  
   - Selecione a versão do MySQL compatível.  
   - Para um lift and shift TOTAL, será utilizado um RDS single-AZ
2. **Configurar AWS DMS**  
   - Criar um **endpoint de origem** (MySQL on-premises) e um **endpoint de destino** (RDS MySQL).  
   - Migrar esquemas, tabelas e dados (full load + CDC, replicação contínua).
3. **Testar**  
   - Validar performance e integridade do banco no RDS.

### 4. Migração de Frontend e Backend

1. **Instalar AWS Replication Agent on-premises**  
   - Configurar para enviar dados/VMs para o **Replication Server** na VPC Staging.  
2. **AWS MGN**  
   - Monitora e converte os dados armazenados no Replication Server em AMIs.  
   - Gera instâncias EC2 correspondentes (Frontend e Backend) na **VPC Final**.
3. **Volumes EBS**  
   - Os dados de cada servidor migrado ficam em **EBS** associados às instâncias EC2 resultantes.

### 5. Teste e Validação

1. **DNS Temporário**  
   - Aponte um subdomínio (ex.: wwww.fastengineering.com.br`) para o **Load Balancer** ou para o IP público do **EC2 Frontend** (caso não use LB).  
   - Verifique se o backend e o RDS estão respondendo corretamente.
2. **Checar Logs e Performance**  
   - Monitorar a saúde das instâncias EC2 (CPU, memória) e do RDS (latência, conexões).  
   - Verificar se APIs, frontend e DB estão funcionando sem erros.

---

## Conclusão

Nesta etapa de **Lift and Shift**, os servidores (Frontend/Backend) e o banco de dados MySQL foram migrados para **AWS EC2** e **Amazon RDS** respectivamente, **sem** grandes alterações na aplicação. O diagrama ilustra a transição via **Replication Server** (VPC Staging) e o **AWS MGN** para as instâncias EC2 finais, enquanto o **AWS DMS** garante a transferência segura dos dados do banco para o RDS.


## Diagrama “Lift-and-Shift"

![[diagrama2](https://github.com/TatianaModelGBock/Projeto-final-PB-SET-2024-CompassUol/blob/main/images/ArqLiftAndShift%20(1).drawio.png)](https://github.com/TatianaModelGBock/Projeto-final-PB-SET-2024-CompassUol/blob/main/images/ArqLiftAndShift.drawio%20(1).png)

---

## **Fase 2: Modernização**

---
## Visão Geral

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

## Objetivos

1. **Escalabilidade**: Escalar horizontalmente as aplicações através de pods (HPA) e aumentar/diminuir nós do EKS (Cluster Auto Scaling).  
2. **Disponibilidade**: Implantar o aplicativo em múltiplas zonas de disponibilidade (Multi-AZ), com RDS MySQL em modo Multi-AZ ou com réplicas para failover.  
3. **Automação**: Usar uma pipeline de CI/CD para **build**, **teste** e **deploy** contínuos.  
4. **Segurança**: Garantir criptografia e proteção em múltiplas camadas (WAF, ALB, subnets privadas, Secrets Manager etc.).  
5. **Observabilidade**: Centralizar logs e métricas no CloudWatch, facilitando troubleshooting e análise de performance.

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

