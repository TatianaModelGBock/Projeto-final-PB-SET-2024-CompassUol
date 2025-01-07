# Projeto Final PB - Compass Uol

## Cloud Pratictioner - PB - SET 2024

## Contexto
A **Fast Engineering S/A** enfrenta desafios de escalabilidade e disponibilidade para seu e-commerce. Atualmente, todo o stack roda **on-premises**, mas o crescimento da demanda exige uma arquitetura mais flexível e resiliente.  
A proposta é migrar o ambiente para a **AWS** em duas fases:

1. **Fase 1**: Migração “Lift and Shift” (o mais rápido possível, mantendo a aplicação como está).  
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
   - Criar uma instância **EC2 MySQL** (Lift and Shift total).  
   - Copiar dados via **AWS Replication Agent** (TCP 1500).  
   - Testar **integridade** e **performance** do banco no ambiente de destino.

4. **Migração de Frontend/Backend**  
   - Usar **AWS MGN** para replicar as máquinas on-premises em instâncias EC2.  
   - Usar **AWS EBS** para hospedar os arquivos advindos das máquinas

5. **Teste e Validação**  
   - Apontar subdomínio (ex.: `test.minhaempresa.com`) para o IP ou ALB da aplicação na AWS.  
   - Verificar logs, monitorar performance.


## Diagrama “Lift-and-Shift"

![diagrama2](https://github.com/TatianaModelGBock/Projeto-final-PB-SET-2024-CompassUol/blob/main/images/ArqLiftAndShift.drawio.png)

---

## **Fase 2: Modernização**

### **Objetivo**
Após a migração inicial, modernizar a arquitetura seguindo boas práticas de nuvem, com foco em escalabilidade, segurança e facilidade de gerenciamento.

---

### **Melhorias Planejadas**

1. **Kubernetes (EKS):**  
   - Gerenciamento de contêineres para as APIs e o frontend.  

2. **Banco de Dados Gerenciado (RDS):**  
   - Multi-AZ para alta disponibilidade.  

3. **Armazenamento de Objetos (S3):**  
   - Para arquivos estáticos, como imagens e vídeos.  

4. **Reforço de Segurança:**  
   - Configuração de IAM roles e policies.  
   - Implementação de WAF (Web Application Firewall).  

5. **Backup Automatizado:**  
   - Utilizar snapshots do RDS e backups automáticos do S3.  

---

### **Passo a Passo da Modernização**

1. Refatorar a aplicação para rodar em contêineres (Docker).  
2. Configurar o EKS e realizar o deploy dos contêineres.  
3. Migrar o banco de dados para o RDS com replicação Multi-AZ.  
4. Migrar os arquivos estáticos para o S3.  
5. Implementar monitoramento com **CloudWatch** e automação com **CloudFormation**.  

