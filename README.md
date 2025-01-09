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

# Fase 2: Modernização com EKS (Elastic Kubernetes Service)

## Visão Geral

- **Amazon EKS** (Elastic Kubernetes Service): Orquestração de contêineres, facilitando implantação de múltiplos microserviços.  
- **Amazon RDS** (MySQL) em Multi-AZ: Banco de dados gerenciado com backup e alta disponibilidade.  
- **Amazon S3**: Armazenamento de objetos estáticos (imagens, PDFs etc.) e/ou backups.  
- **CI/CD com CodePipeline, CodeBuild, CodeCommit, ECR**: Fluxo de entrega contínua para as imagens Docker das aplicações.  
- **Security**: AWS WAF, Security Groups, IAM Roles, Secrets Manager e KMS para criptografia.

---

## Passo a Passo

### 1. Preparar o Ambiente de Contêineres

1. **Criar Cluster EKS**
   - Usar **eksctl**, **Terraform**, ou **CloudFormation** para criar o cluster.  
   - Definir subnets privadas (worker nodes) e subnets públicas (para o Load Balancer).  
   - Habilitar **Cluster Autoscaler** para redimensionar nós conforme a demanda.

2. **Configurar VPC e Subnets**
   - As **subnets privadas** recebem os pods do Kubernetes (sem IP público).  
   - As **subnets públicas** hospedam o Application Load Balancer (ALB) ou NAT Gateways (se for preciso).  
   - **Internet Gateway (IGW)** conectado às subnets públicas para tráfego externo.

3. **Criar e Configurar os Worker Nodes**
   - Crie um **Auto Scaling Group** para os nós do EKS (EC2).  
   - Defina o tipo de instância (ex.: `m5.large`), capacidade mínima e máxima (ex.: 2 a 6 nós).  
   - Vincule a **IAM Role** que permita operações do Kubernetes (por exemplo, criar logs em CloudWatch).

---

### 2. Containerizar as Aplicações

1. **Dockerfiles**
   - Criar Dockerfile(s) para o **Backend** (APIs, Nginx) e, caso exista, para o **Frontend** (React).
   - 
2. **Armazenar Imagens no Amazon ECR**
   - Criar repositório ECR (`backend-api`, `frontend-app`, etc.).  
   - Executar `docker build` e `docker push` para enviar as imagens.  
   - Usar tags para versionamento (`v1.0.0`, `latest`, etc.).

3. **Configurar Variáveis de Ambiente**
   - Endpoints do banco (RDS), credenciais do Secrets Manager, URLs de APIs etc.  
   - Evite armazenar senhas direto no Dockerfile ou no código.


### 3. Pipeline de CI/CD

1. **AWS CodeCommit / Git**  
   - Repositório de código para as aplicações.  
   - Os desenvolvedores fazem **push** no branch principal (ex.: `main`).

2. **AWS CodeBuild**  
   - Etapa de build que:  
     1. Faz `docker build` das imagens,  
     2. Sobe para o ECR,  
     3. (Opcional) Executa testes de unidade/integração.

3. **AWS CodePipeline**  
   - Orquestra o fluxo **Source** (CodeCommit) → **Build** (CodeBuild) → **Deploy** (CloudFormation ou `kubectl apply`).  
   - Ao final, o EKS recebe a nova versão do deployment de contêiner.


### 4. Implantar Aplicações no EKS

1. **Manifests Kubernetes**  
   - Criar arquivos YAML para `Deployment`, `Service`, `Ingress`, etc.  

2. **Ingress Controller (ALB)**  
   - Se quiser expor via ALB Ingress Controller, criar um `Ingress` definindo rotas (e.g., `/api` -> backend service).  
   - O Ingress Controller gera o Application Load Balancer associado, colocando-o em subnets públicas.

3. **Horizontal Pod Autoscaler (HPA)**  
   - Configurar o HPA para cada deployment, definindo escalonamento baseado em CPU/Memória ou métricas customizadas.


### 5. Integração com RDS e S3

1. **Banco de Dados**  
   - Use o RDS MySQL migrado na Fase 1 ou crie um novo (Multi-AZ, backups automáticos).  
   - Ajuste o **Security Group** para aceitar conexões somente do EKS (via IAM Roles for Service Accounts ou SGs).  
   - Ativar **SSL** (opcional) para criptografia em trânsito.

2. **Armazenamento de Arquivos (S3)**  
   - Se a aplicação manipula uploads de usuários (imagens, PDFs), salve no **Amazon S3**.  
   - Configure um bucket e use **bucket policies** ou IAM roles para permitir acesso somente pelos pods.

3. **Secrets Manager**  
   - Armazene credenciais (usuário/senha do RDS, tokens de API etc.)  
   - Use IRSA (IAM Roles for Service Accounts) para que os pods possam ler segredos de forma segura.

### 6. Segurança e Observabilidade

1. **CloudWatch Logs e Metrics**  
   - Instale o **CloudWatch Agent** ou configure **Container Insights** para coletar logs e métricas de pods.  
   - Crie alarmes no CloudWatch (ex.: CPU, memória, erro 5xx no ALB).

2. **WAF + CloudFront**  
   - Se a aplicação exigir conteúdo estático e cache global, use o **CloudFront** como CDN.  
   - Ative **AWS WAF** para mitigar ameaças em camada 7 (SQLi, XSS etc.).

3. **IAM Roles e KMS**  
   - Restrinja acessos (princípio de menor privilégio).  
   - Se for necessário armazenar dados sensíveis, encriptar com **AWS KMS**.

4. **GuardDuty, AWS Config** (opcional)  
   - Para detecção de ameaças e conformidade.  
   - Receber alertas de configurações indevidas ou acessos suspeitos.


### 7. Teste, Monitoramento e Otimização

1. **Testar Aplicação**  
   - Validar rotas do Ingress, ver se o DNS está corretamente apontando para o ALB.  
   - Checar se pods escalam e o cluster autoscaling funciona.

2. **Monitorar Custos**  
   - Acompanhar uso das instâncias (muito sub-utilizadas ou saturadas?).  
   - Considerar Savings Plans ou Reserved Instances se houver workloads de longa duração.

3. **Refinar Deploy**  
   - Ajustar políticas de release (blue/green, canary) caso queiram zero downtime nas implantações.  
   - Adicionar testes automatizados no pipeline.


## Conclusão

Nesta fase, o ambiente deixa de ser “apenas Lift and Shift” e passa a adotar **princípios cloud-native**:
- **EKS** para orquestração de contêineres.
- **RDS** para banco gerenciado.
- **S3** para arquivos e backups.
- **CI/CD** robusto com CodePipeline e CodeBuild.
- **Segurança** em múltiplas camadas (WAF, SG, IAM, KMS).

## Diagrama "Modernização EKS"

![diagrama](https://github.com/TatianaModelGBock/Projeto-final-PB-SET-2024-CompassUol/blob/main/images/Moderniza%C3%A7%C3%A3o_Kubernetes.drawio%20(5).png)

**Benefícios esperados** incluem **melhor escalabilidade**, **redução de custos de manutenção** e **agilidade** na entrega de novas features (devops).
