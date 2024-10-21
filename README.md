# Configuração de Infraestrutura AWS com Terraform

O código configura uma **infraestrutura completa na AWS**, criando uma rede, configurando a segurança e iniciando uma instância EC2 rodando **Debian 12**, além de fornecer as informações necessárias para acessar a máquina.

Este código é um **arquivo de configuração do Terraform**, uma ferramenta de Infraestrutura como Código (IaC) utilizada para gerenciar recursos em nuvem, neste caso, na AWS. Ele especifica a criação de uma **VPC (Virtual Private Cloud)**, sub-rede, gateway de internet, tabela de rotas, par de chaves, grupo de segurança e uma instância EC2 com Debian.

## Principais Componentes

- **provider "aws"**: Define a AWS como provedora de infraestrutura e utiliza a região "us-east-1".
- **variable "projeto" e "candidato"**: Declaram variáveis para o nome do projeto ("VExpenses") e do candidato ("SeuNome"), que serão reutilizadas dinamicamente na nomeação dos recursos.
- **resource "tls_private_key" "ec2_key"**: Gera uma chave privada usando o algoritmo RSA de 2048 bits, necessária para acessar a instância EC2.
- **resource "aws_key_pair" "ec2_key_pair"**: Cria um par de chaves na AWS com base na chave pública gerada, nomeando o par conforme as variáveis do projeto e candidato.
- **resource "aws_vpc" "main_vpc"**: Cria uma VPC com o bloco CIDR `10.0.0.0/16`, que comporta até 65.536 endereços IP, com suporte a DNS e resolução de hostnames.
- **resource "aws_subnet" "main_subnet"**: Define uma sub-rede dentro da VPC com o bloco CIDR `10.0.1.0/24` (256 endereços IP) na zona de disponibilidade "us-east-1a".
- **resource "aws_internet_gateway" "main_igw"**: Cria um gateway de internet, permitindo que a VPC tenha acesso externo.
- **resource "aws_route_table" "main_route_table"**: Cria uma tabela de rotas que direciona o tráfego da rede para fora, utilizando o gateway de internet.
- **resource "aws_route_table_association" "main_association"**: Associa a tabela de rotas à sub-rede, garantindo que as instâncias dentro dela tenham acesso à internet.
- **resource "aws_security_group" "main_sg"**: Define um grupo de segurança que controla o tráfego:
  - **Ingress (entrada)**: Permite conexões SSH (porta 22) de qualquer endereço IP.
  - **Egress (saída)**: Libera todo o tráfego de saída.
- **data "aws_ami" "debian12"**: Encontra a versão mais recente da AMI do Debian 12.
- **resource "aws_instance" "debian_ec2"**: Cria uma instância EC2 com a AMI do Debian 12, utilizando o tipo `t2.micro`, elegível para o nível gratuito da AWS. A instância é configurada com 20 GB de armazenamento e um script de inicialização que atualiza o sistema. Além disso, ela é provisionada com um IP público e associada ao grupo de segurança e à chave gerada.

## Alterações Implementadas

### 1. Restrição do Acesso SSH

- **Motivo**: O acesso SSH estava configurado para aceitar conexões de qualquer IP, aumentando os riscos de segurança.
- **Alteração**: Limitamos o acesso SSH a um IP específico ou a uma faixa de IPs confiável (`cidr_blocks = ["203.0.113.0/32"]`).
- **Benefício**: Reduz o risco, permitindo apenas conexões de locais confiáveis.

### 2. Habilitação de Logs de Tráfego na VPC (Flow Logs)

- **Motivo**: Não havia visibilidade sobre o tráfego dentro da VPC, dificultando a detecção de atividades suspeitas.
- **Alteração**: Adicionamos um recurso `aws_flow_log` para capturar o tráfego da VPC e enviá-lo ao CloudWatch Logs.
- **Benefício**: Facilita a auditoria e o monitoramento de possíveis atividades maliciosas.

### 3. Criptografia do Volume EBS

- **Motivo**: O volume da instância EC2 não estava criptografado, expondo os dados armazenados a riscos de segurança.
- **Alteração**: Ativamos a criptografia do volume EBS utilizando a propriedade `encrypted = true`.
- **Benefício**: Garante a proteção dos dados em repouso, prevenindo vazamentos em caso de acesso não autorizado.

### 4. Ativação de Monitoramento Detalhado na EC2

- **Motivo**: O monitoramento padrão coletava métricas a cada 5 minutos, o que não é suficiente para análises em tempo real.
- **Alteração**: Habilitamos o monitoramento detalhado, ajustando a frequência de coleta de métricas para cada 1 minuto (`monitoring = true`).
- **Benefício**: Aumenta a visibilidade sobre o desempenho e a segurança da instância.

Essas melhorias aumentam a **segurança** e a **capacidade de monitoramento** da infraestrutura na AWS, permitindo uma auditoria mais eficiente e proteção adicional aos dados.

## Configuração do Nginx

Utilizamos o campo `user_data` no recurso da instância EC2, permitindo a execução automática de um script durante a inicialização da instância.