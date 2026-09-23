| id    | casos_de_uso_arquiteturais |
|-------|------------------------------|
| title | Casos de Uso Arquiteturais   |

# Casos de Uso Arquiteturais

## Plataforma de E-commerce com Arquitetura de Nuvem Híbrida e Logística Last-Mile Inteligente

*Modelo de Casos de Uso Arquiteturais — Projeto de Cloud*

| Informações | |
|---|---|
| **Projeto** | Plataforma de E-commerce com Logística Last-Mile Inteligente |
| **Documento** | Modelo de Casos de Uso Arquiteturais |
| **Versão** | 1.0 |
| **Data** | [23/09/2026] |
| **Status** | Em Desenvolvimento |
| **Responsável** | [Grupo E-commerce] |
| **Disciplina** | Projeto de Cloud |

---

## 1. Introdução

### 1.1. Propósito

Este documento descreve os Casos de Uso Arquiteturais para a infraestrutura em nuvem AWS da plataforma de e-commerce. Os casos de uso arquiteturais focam em requisitos de infraestrutura, segurança, operações e governança, diferentemente dos casos de uso funcionais, que descrevem interações de usuários finais (compradores, lojistas) com o sistema.

### 1.2. Escopo

Os casos de uso arquiteturais abrangem a configuração, operação e manutenção da infraestrutura AWS, incluindo:

- Rede (VPC, sub-redes, rotas, endpoints)
- Segurança (Security Groups, NACLs, IAM)
- Conectividade entre camadas (ALB, EC2, RDS, Lambda)
- Integração com serviços gerenciados (DynamoDB, S3, Secrets Manager)
- Automação e deploy (CI/CD)

### 1.3. Referências

- Documento de Visão — Plataforma de E-commerce (v1.0)
- Documento de Requisitos Suplementares — Plataforma de E-commerce (v1.0)
- AWS Well-Architected Framework
- Amazon VPC Documentation

---

## 2. Visão Geral dos Casos de Uso

### 2.1. Atores

| Ator | Descrição | Responsabilidades |
|---|---|---|
| Administrador de Infraestrutura | Profissional responsável por configurar e gerenciar a infraestrutura AWS. | Criar VPC, sub-redes, security groups, endpoints. |
| Engenheiro de DevOps | Profissional responsável por automação e CI/CD. | Configurar pipelines, deploys, rollbacks. |
| Sistema AWS | Serviços gerenciados da AWS (DynamoDB, S3, etc.). | Prover serviços, endpoints, logs. |
| Arquiteto de Segurança | Profissional responsável por políticas de segurança e compliance. | Definir políticas IAM, criptografia, auditoria. |

### 2.2. Diagrama de Casos de Uso

![Diagrama de Casos de Uso Arquiteturais](casos_uso_diagrama.png)

---

## 3. Especificação dos Casos de Uso

### UC-ARQ-001: Configurar VPC e Rede do E-commerce

| Elemento | Especificação |
|---|---|
| **Identificador** | UC-ARQ-001 |
| **Nome** | Configurar VPC e Rede do E-commerce |
| **Versão** | 1.0 |
| **Data** | [DD/MM/AAAA] |
| **Status** | Aprovado |
| **Ator Principal** | Administrador de Infraestrutura |
| **Ator Secundário** | Sistema AWS |
| **Pré-condição** | 1. Conta AWS ativa.<br>2. Permissões IAM para criar VPC, sub-redes, IGW, NAT Gateway, Route Tables. |
| **Pós-condição** | 1. VPC criada com CIDR 10.0.0.0/16.<br>2. 4 sub-redes criadas (2 públicas, 2 privadas) em 2 AZs.<br>3. Internet Gateway anexado.<br>4. NAT Gateways configurados.<br>5. Route Tables configuradas. |

**Fluxo Principal**

| # | Passo |
|---|---|
| 1 | Administrador acessa o Console AWS. |
| 2 | Navega até o serviço VPC. |
| 3 | Cria VPC com CIDR 10.0.0.0/16 e habilita DNS hostnames. |
| 4 | Cria 2 sub-redes públicas (uma por AZ) com CIDR 10.0.1.0/24 e 10.0.2.0/24, para o Load Balancer. |
| 5 | Cria 2 sub-redes privadas (uma por AZ) com CIDR 10.0.3.0/24 e 10.0.4.0/24, para EC2 (Django) e RDS. |
| 6 | Cria e anexa Internet Gateway à VPC. |
| 7 | Cria NAT Gateways (um por AZ) nas sub-redes públicas. |
| 8 | Configura Route Table pública: 10.0.0.0/16 → local, 0.0.0.0/0 → IGW. |
| 9 | Configura Route Table privada: 10.0.0.0/16 → local, 0.0.0.0/0 → NAT Gateway. |
| 10 | Associa sub-redes públicas à Route Table pública. |
| 11 | Associa sub-redes privadas à Route Table privada. |
| 12 | Valida conectividade: instância pública → Internet, instância privada → NAT. |

**Fluxos Alternativos**

| Alt. | Descrição |
|---|---|
| Alt 1 | CIDR da VPC conflita com outra VPC ou rede on-premises (erro de criação). |
| Alt 2 | Limite de VPCs por região é atingido (5 VPCs por região). |
| Alt 3 | Permissões IAM insuficientes (erro de autorização). |
| Alt 4 | NAT Gateway falha (instância privada perde acesso à Internet). |

**Requisitos Não-Funcionais**

| Dimensão | Métrica |
|---|---|
| Desempenho | VPC criada e configurada em < 15 min. |
| Disponibilidade | Multi-AZ com 2 AZs (redundância). |
| Segurança | Sub-redes privadas sem rota direta para Internet. |
| Custo | NAT Gateway: ~US$ 0,045/hora por AZ (~US$ 64/mês para 2 AZs). |

**Riscos e Mitigação**

| Risco | Mitigação |
|---|---|
| Endereçamento conflitante | Planejar CIDR com folga (10.0.0.0/16). |
| Custo elevado do NAT Gateway | Avaliar uso de NAT Instance (mais barato, menos gerenciado). |
| Erro de roteamento | Testar conectividade com ping ou traceroute. |

---

### UC-ARQ-002: Configurar Segurança de Rede do E-commerce

| Elemento | Especificação |
|---|---|
| **Identificador** | UC-ARQ-002 |
| **Nome** | Configurar Segurança de Rede do E-commerce |
| **Versão** | 1.0 |
| **Data** | [DD/MM/AAAA] |
| **Status** | Aprovado |
| **Ator Principal** | Administrador de Infraestrutura |
| **Ator Secundário** | Arquiteto de Segurança, Sistema AWS |
| **Pré-condição** | 1. VPC e sub-redes criadas (UC-ARQ-001).<br>2. IAM configurado. |
| **Pós-condição** | 1. Security Groups configurados para cada camada.<br>2. NACLs configuradas.<br>3. IAM roles definidas para serviços. |

**Fluxo Principal**

| # | Passo |
|---|---|
| 1 | Administrador acessa o Console AWS. |
| 2 | Navega até Security Groups. |
| 3 | Cria SG-ALB com regras de entrada: portas 80, 443 (0.0.0.0/0). |
| 4 | Cria SG-EC2-Django com regras de entrada: porta 8000 (origem: SG-ALB). |
| 5 | Cria SG-RDS com regras de entrada: porta 5432 (origem: SG-EC2-Django). |
| 6 | Cria SG-Lambda com regras de saída: DynamoDB, S3, Secrets Manager. |
| 7 | Cria NACLs para sub-redes públicas e privadas (camada extra de segurança). |
| 8 | Define IAM roles para EC2, Lambda e RDS. |

**Fluxos Alternativos**

| Alt. | Descrição |
|---|---|
| Alt 1 | Porta 8000 exposta acidentalmente → revisão das regras. |
| Alt 2 | IAM role com permissões excessivas → aplicar princípio de menor privilégio. |

**Requisitos Não-Funcionais**

| Dimensão | Métrica |
|---|---|
| Segurança | Princípio de menor privilégio implementado. |
| Segurança | Criptografia em trânsito (TLS 1.3). |
| Compliance | Conformidade com a LGPD. |

**Riscos e Mitigação**

| Risco | Mitigação |
|---|---|
| Security Groups permissivos | Revisão periódica das regras. |
| Acesso não autorizado | MFA obrigatório para administradores. |

---

### UC-ARQ-003: Configurar VPC Endpoints do E-commerce

| Elemento | Especificação |
|---|---|
| **Identificador** | UC-ARQ-003 |
| **Nome** | Configurar VPC Endpoints do E-commerce |
| **Versão** | 1.0 |
| **Data** | [DD/MM/AAAA] |
| **Status** | Aprovado |
| **Ator Principal** | Administrador de Infraestrutura |
| **Ator Secundário** | Sistema AWS (DynamoDB, S3, Secrets Manager) |
| **Pré-condição** | 1. VPC e sub-redes privadas criadas (UC-ARQ-001).<br>2. IAM configurado. |
| **Pós-condição** | 1. DynamoDB Gateway Endpoint configurado.<br>2. S3 Interface Endpoint configurado.<br>3. Secrets Manager Interface Endpoint configurado.<br>4. Políticas de endpoint restritivas aplicadas. |

**Fluxo Principal**

| # | Passo |
|---|---|
| 1 | Administrador acessa o Console AWS. |
| 2 | Navega até VPC Endpoints. |
| 3 | Cria Gateway Endpoint para DynamoDB (tipo: Gateway), usado para os eventos de rastreamento logístico. |
| 4 | Associa às sub-redes privadas (10.0.3.0/24, 10.0.4.0/24). |
| 5 | Atualiza Route Table privada com rota para o DynamoDB. |
| 6 | Cria Interface Endpoint para S3 (tipo: Interface), usado para imagens de produtos e canhotos de entrega. |
| 7 | Cria Interface Endpoint para Secrets Manager (tipo: Interface). |
| 8 | Aplica políticas de endpoint para restringir acesso a buckets/segredos específicos. |
| 9 | Valida conectividade: EC2 → DynamoDB, Lambda → S3. |

**Fluxos Alternativos**

| Alt. | Descrição |
|---|---|
| Alt 1 | Serviço não suporta Gateway Endpoint (ex.: Secrets Manager) → usar Interface Endpoint. |
| Alt 2 | Política de endpoint bloqueia acesso → revisar e ajustar. |

**Requisitos Não-Funcionais**

| Dimensão | Métrica |
|---|---|
| Segurança | Tráfego não sai da VPC. |
| Custo | Interface Endpoints: US$ 0,01/hora cada (~US$ 14/mês). |
| Desempenho | Latência reduzida em comparação ao acesso via Internet. |

**Riscos e Mitigação**

| Risco | Mitigação |
|---|---|
| Custo elevado | Usar Gateway Endpoint para serviços suportados (DynamoDB, S3). |
| Políticas restritivas | Testar políticas
