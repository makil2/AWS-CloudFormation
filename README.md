# ☁️ AWS CloudFormation — Primeiros Passos com Infraestrutura como Código

> Repositório criado como entrega do desafio **"Implementando sua Primeira Stack com AWS CloudFormation"** da [Digital Innovation One (DIO)](https://www.dio.me/).

---

## 📋 Sobre o Projeto

Este repositório documenta a implementação prática de stacks AWS utilizando **AWS CloudFormation**, serviço que permite provisionar e gerenciar recursos de infraestrutura na AWS através de arquivos de configuração (IaC — *Infrastructure as Code*).

Os templates foram desenvolvidos de forma progressiva, partindo de uma instância EC2 simples até uma stack completa com EC2, S3, IAM User e Security Group.

---

## 🎯 Objetivos de Aprendizagem

- ✅ Compreender os conceitos de **Infraestrutura como Código (IaC)**
- ✅ Criar e implantar **stacks no AWS CloudFormation**
- ✅ Configurar instâncias **EC2** com e sem scripts de inicialização
- ✅ Instalar e configurar um **servidor web Apache** via `UserData`
- ✅ Criar e associar **Security Groups** para controle de tráfego
- ✅ Provisionar recursos integrados: **EC2 + S3 + IAM User + IAM Group**
- ✅ Documentar processos técnicos de forma clara no **GitHub**

---

## 📁 Estrutura do Repositório

```
.
├── templates/
│   ├── 01-EC2.yaml                # EC2 simples
│   ├── 02-Apache.yaml             # EC2 com Apache via UserData
│   ├── 03-Firewall.yaml           # EC2 + Apache + Security Group
│   └── 04-EC2_S3_UserGroup.yaml   # Stack completa com S3 e IAM
├── images/                        # Capturas de tela da implementação
└── README.md
```

---

## 🛠️ Templates CloudFormation

### 01 — EC2 Simples (`01-EC2.yaml`)

Cria uma instância Amazon EC2 básica na região `us-east-1a`, utilizando a AMI Amazon Linux 2 e o tipo `t2.micro` (elegível ao Free Tier).

**Recursos criados:**
| Recurso | Tipo | Detalhes |
|---|---|---|
| `MinhaInstancia` | `AWS::EC2::Instance` | t2.micro · ami-0ed9277fb7eb570c9 · us-east-1a |

```yaml
Resources:
  MinhaInstancia:
    Type: AWS::EC2::Instance
    Properties:
      AvailabilityZone: us-east-1a
      ImageId: ami-0ed9277fb7eb570c9
      InstanceType: t2.micro
      Tags:
        - Key: "Name"
          Value: "EC2"
```

---

### 02 — Servidor Apache (`02-Apache.yaml`)

Evolução do template anterior: adiciona um bloco `UserData` que executa um script de inicialização para instalar e ativar o Apache HTTP Server automaticamente ao subir a instância.

**Recursos criados:**
| Recurso | Tipo | Detalhes |
|---|---|---|
| `MinhaInstancia` | `AWS::EC2::Instance` | t2.micro + Apache instalado via UserData |

**Script `UserData` executado na inicialização:**
```bash
#!/bin/bash -xe
yum install -y httpd.x86_64
systemctl start httpd.service
systemctl enable httpd.service
echo "<h1>OLA AWS FOUNDATIONS do $(hostname -f)</h1>" > /var/www/html/index.html
```

> ⚠️ **Nota:** Neste ponto ainda não há Security Group configurado, portanto o acesso HTTP externo ainda não está liberado.

---

### 03 — Firewall / Security Group (`03-Firewall.yaml`)

Adiciona um **Security Group** à stack, liberando a porta **80 (HTTP)** para acesso público. A instância EC2 é associada ao grupo de segurança criado no mesmo template.

**Recursos criados:**
| Recurso | Tipo | Detalhes |
|---|---|---|
| `MinhaInstancia` | `AWS::EC2::Instance` | EC2 com Apache + Security Group associado |
| `GrupoSeguranca` | `AWS::EC2::SecurityGroup` | Libera TCP porta 80 de qualquer origem (0.0.0.0/0) |

**Regra de entrada configurada:**
```yaml
SecurityGroupIngress:
  - IpProtocol: tcp
    FromPort: 80
    ToPort: 80
    CidrIp: 0.0.0.0/0
```

> ✅ Com esta configuração, é possível acessar a página do Apache pelo IP público da instância no navegador.

---

### 04 — Stack Completa: EC2 + S3 + IAM (`04-EC2_S3_UserGroup.yaml`)

Template mais completo do projeto, demonstrando como provisionar múltiplos recursos integrados em uma única stack CloudFormation.

**Recursos criados:**
| Recurso | Tipo | Detalhes |
|---|---|---|
| `S3Bucket` | `AWS::S3::Bucket` | Bucket chamado `S3-FOUNDATION` |
| `IAMGroup` | `AWS::IAM::Group` | Grupo `GPO-ADMIN-LAB` |
| `IAMUser` | `AWS::IAM::User` | Usuário `alexsandro.lechner` vinculado ao grupo IAM |
| `EC2Instance` | `AWS::EC2::Instance` | Ubuntu + Python3 via UserData |
| `EC2SecurityGroup` | `AWS::EC2::SecurityGroup` | Libera SSH (porta 22) |

**Diferenciais deste template:**

- Uso de **`Parameters`** para permitir escolha do tipo de instância no momento do deploy
- Uso de **`Mappings`** para selecionar automaticamente a AMI correta por região (us-east-1 / us-west-2)
- Uso de **`Outputs`** para expor o ID da instância, nome do bucket e nome do usuário IAM após a criação

```yaml
Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues:
      - t2.micro
      - t3.micro
```

---

## 🚀 Como Implantar os Templates

### Pré-requisitos

- Conta AWS ativa
- Permissões para criar EC2, S3, IAM e CloudFormation
- AWS CLI configurado (opcional, para deploy via terminal)

### Via Console AWS

1. Acesse o serviço **CloudFormation** no [Console AWS](https://console.aws.amazon.com/cloudformation/)
2. Clique em **"Criar stack" → "Com novos recursos"**
3. Selecione **"Fazer upload de um arquivo de modelo"** e envie o arquivo `.yaml` desejado
4. Preencha os parâmetros solicitados (quando aplicável)
5. Avance pelas etapas e clique em **"Criar stack"**
6. Aguarde o status mudar para `CREATE_COMPLETE`

### Via AWS CLI

```bash
aws cloudformation create-stack \
  --stack-name minha-stack \
  --template-body file://templates/01-EC2.yaml \
  --region us-east-1
```

Para o template com recursos IAM, adicione a flag de capacidades:
```bash
aws cloudformation create-stack \
  --stack-name stack-completa \
  --template-body file://templates/04-EC2_S3_UserGroup.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --region us-east-1
```

---

## 🔐 Boas Práticas e Pontos de Atenção

- **Nunca exponha chaves de acesso** (`AccessKey`, `SecretKey`) em templates públicos
- **Restrinja o CidrIp do SSH** (`0.0.0.0/0`) ao seu IP real em ambientes de produção
- **Exclua as stacks** após os testes para evitar cobranças inesperadas na AWS
- **AMIs são específicas por região** — sempre verifique o ID correto antes do deploy
- O campo `KeyName` no template 04 deve ser substituído pelo nome do seu par de chaves existente

---

## 📚 Referências

- [AWS CloudFormation — Documentação Oficial](https://docs.aws.amazon.com/pt_br/AWSCloudFormation/latest/UserGuide/Welcome.html)
- [AWS CloudFormation — Guia de Início Rápido](https://docs.aws.amazon.com/pt_br/AWSCloudFormation/latest/UserGuide/gettingstarted.walkthrough.html)
- [Tipos de Recursos AWS para CloudFormation](https://docs.aws.amazon.com/pt_br/AWSCloudFormation/latest/UserGuide/aws-template-resource-type-ref.html)
- [Digital Innovation One](https://www.dio.me/)
