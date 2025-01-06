# Projeto Final PB - Compass Uol

## Cloud Pratictioner - PB - SET 2024

## 1. Contexto

Nós somos da empresa "Fast Engineering S/A" e gostaríamos de uma solução dos
senhores(as), que fazem parte da empresa terceira "TI SOLUÇÕES INCRÍVEIS".
Nosso eCommerce está crescendo e a solução atual não está atendendo mais a alta
demanda de acessos e compras que estamos tendo.
Atualmente usamos:
•
•
•
01 servidor para Banco de Dados Mysql (500GB de dados, 10Gb de RAM, 3 Core
CPU);
01 servidor para a aplicação utilizando REACT – frontend (5GB de dados, 2Gb de
RAM, 1 Core CPU);
01 servidor de backend com 3 APIs, com o Nginx servindo de balanceador de
carga e que armazena estáticos como fotos e links. (5GB de dados, 4Gb de RAM,
2 Core CPU);



Queremos modernizar esse sistema para a AWS, precisamos seguir as melhores
práticas arquitetura em Cloud AWS, a nova arquitetura deve seguir as seguintes
diretrizes:

• Ambiente Kubernetes;

• Banco de dados gerenciado (PaaS e Multi AZ);

• Backup de dados;

• Sistema para persistência de objetos (imagens, vídeos etc.);

• Segurança; 

Porém antes da migração acontecer para a nova estrutura, precisamos fazer uma
migração “lift-and-shift” ou “as-is”, o mais rápido possível, só depois que iremos
promover a modificação para a nova estrutura em Kubernetes.

