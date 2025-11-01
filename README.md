# Desafio de Projeto DIO: Infraestrutura como Código com AWS CloudFormation

Este repositório documenta a solução do Desafio de Projeto da [Digital Innovation One (DIO)](https://www.dio.me/) sobre AWS CloudFormation.

O objetivo foi aplicar os conceitos de **Infraestrutura como Código (IaC)** para provisionar (criar) e gerenciar uma infraestrutura na nuvem AWS de forma automatizada, documentando o processo e os aprendizados adquiridos.

---

## 🎯 Objetivo

O desafio consistiu em implementar uma "Stack" no AWS CloudFormation a partir de um template. Este repositório serve como a documentação central do projeto, contendo o template utilizado e os insights sobre o processo.

---

## 📖 O que é AWS CloudFormation? (Conceitos Aprendidos)

Durante o estudo, estes foram meus principais aprendizados sobre o serviço:

* **O que é?** É o serviço de **Infraestrutura como Código (IaC)** nativo da AWS. Ele permite que você defina e provisione recursos da AWS de forma previsível e repetível.
* **Template:** É o "coração" do CloudFormation. É um arquivo de texto (em formato **YAML** ou **JSON**) que descreve *quais* recursos você quer criar (ex: instâncias EC2, Security Groups, Buckets S3). Este arquivo é a "planta" da sua infraestrutura.
* **Stack:** É a unidade de gerenciamento. Quando o CloudFormation "executa" um template, ele cria os recursos definidos como uma única unidade, chamada *Stack*. Atualizar ou deletar a Stack atualiza ou deleta todos os recursos associados.
* **Vantagens (O "Porquê"):**
    * **Automatização:** Cria e deleta ambientes complexos com um único comando.
    * **Padronização:** Garante que os ambientes de Desenvolvimento, Teste e Produção sejam idênticos.
    * **Versionamento:** O template (o `.yaml`) pode ser versionado no Git, como qualquer outro código, permitindo rastrear mudanças na infraestrutura.
    * **Gerenciamento de Dependências:** O CloudFormation entende as dependências (ex: "é preciso criar o Security Group *antes* de criar a instância EC2") e executa na ordem correta.

---

## ⚙️ O Projeto: A "Stack" Implementada

O template implementado neste projeto (baseado nas aulas) cria uma infraestrutura básica, mas fundamental: **um servidor web Apache rodando em uma instância EC2.**

### Recursos Provisionados:

O template (descrito abaixo) define os seguintes recursos:

1.  **`AWS::EC2::Instance` (Instância EC2):**
    * Define a máquina virtual (Tipo: `t2.micro`).
    * Especifica a Imagem (AMI) a ser usada (ex: Amazon Linux 2).
    * Contém um script `UserData` que é executado na inicialização para instalar o servidor web Apache e criar uma página de teste (`index.html`).
2.  **`AWS::EC2::SecurityGroup` (Grupo de Segurança):**
    * Atua como um "firewall virtual" para a instância.
    * Libera a porta **80 (HTTP)** para permitir acesso ao servidor web a partir de qualquer IP.
    * Libera a porta **22 (SSH)** para permitir acesso administrativo.

### 1. O Template (Código-Fonte)

Abaixo está o código-fonte do template (em YAML) que define toda a infraestrutura:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: Desafio DIO - Stack CloudFormation para criar um servidor web Apache.

Resources:
  # 1. Define o Grupo de Segurança (Firewall)
  WebServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Libera portas 80 (HTTP) e 22 (SSH)
      SecurityGroupIngress:
        # Regra para HTTP (Porta 80)
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        # Regra para SSH (Porta 22)
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0 # ATENÇÃO: Em produção, restrinja isso ao seu IP!

  # 2. Define a Instância EC2 (Servidor)
  EC2Instance:
    Type: AWS::EC2::Instance
    Properties:
      InstanceType: t2.micro
      # IMPORTANTE: Este ID de AMI (Amazon Linux 2) é da região us-east-1 (N. Virginia).
      # Se a aula usou outra região, você pode precisar atualizar este ID.
      ImageId: ami-0c7217cdde317cfec 
      SecurityGroupIds:
        # Associa o Grupo de Segurança criado acima a esta instância
        - !Ref WebServerSecurityGroup
      
      # Script que executa na primeira inicialização da máquina
      UserData:
        Fn::Base64: |
          #!/bin/bash
          # Atualiza os pacotes
          yum update -y
          # Instala o servidor web Apache
          yum install -y httpd
          # Inicia o serviço do Apache
          systemctl start httpd
          # Garante que o Apache inicie com o sistema
          systemctl enable httpd
          # Cria uma página web simples de teste
          echo "<html><h1>Servidor Web (Desafio DIO CloudFormation) - OK</h1></html>" > /var/www/html/index.html

Outputs:
  # Cria uma "saída" que mostra o IP público do servidor web
  WebsiteURL:
    Description: URL do servidor web Apache
    Value: !GetAtt [EC2Instance, PublicDnsName]
