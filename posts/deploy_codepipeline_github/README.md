

### Criando o IAM Role para as instâncias EC2 para Deploy

- Acessar IAM, Roles, clicar em `CreateRole`
- `Trusted entity type`: `AWS service`
- Selecionar `Service or use case`: `EC2`, case `EC2`, clica em Next

![alt text](imagens/iam_role.png)


- Adiciona a Permissão `AmazonEC2RoleforAWSCodeDeploy`, clica em Next

![alt text](imagens/permission_policies_ec2.png)

- Atribua o nome da role `FlaskAppEC2DeployRole` 
- Clica em `Create role` 

![alt text](imagens/role_name_ec2.png)

- É necessario incluir as seguintes permissões para realizar o envio dos Logs da EC2 para o CloudWatch:

``` yaml
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "logs:PutLogEvents",
        "logs:DescribeLogStreams",
        "logs:DescribeLogGroups",
        "logs:CreateLogStream",
        "logs:CreateLogGroup"
      ],
      "Resource": "*"
    }
  ]
}
```

### Configurando uma instância EC2 Instance para o AWS CodeDeploy


#### Name and tags
- Acessar EC2, `Launch Instance`
- Na seção `Name and tags`, clicar em `Add addional tags`
- Atribuir `Key`: `Application`
- Atribuir `Value`: `My Flask App Web Server`

![alt text](imagens/names_and_tags_instance.png)

- Seleciona Amazon Machine Image (AMI): `Amazon Linux 2 AMI (HVM) (Free Tier)`
- Key pair (login): `Proceed without key pair`

![alt text](imagens/instance_type_key_pair.png)


#### Network settings
- VPC padrão: EC2 deve ser iniciada em uma sub-red pública
- Subnet: No preference
- Auto-assign pubic IP: Enable
- Seleciona Create security group
- Security group name - required: web-server-security-group
- Clica em Add security group rule
-- Type: HTTP
-- Source type: Anywhere

![alt text](imagens/security_group.png)

#### Advanced details 
- Na opção `IAM instance profile` seleciona a role que criamos anteriomente, `FlaskAppEC2DeployRole`
- Clica em `Launch Instance`

![alt text](imagens/advanced_details.png)

#### Iniciando a EC2

- Após iniciar a instancia, clica no `Instance ID`

![alt text](imagens/list_instances.png)

- É possível acessar a página na web utilizando o ipv4 DNS da instância e incluindo http:// no início.
- Clique em `Connect`

![alt text](imagens/instance_summary.png)

- Clique em `Connect`

![alt text](imagens/instance_connect.png)

- Conectado à instância

![alt text](image.png)


#### Instalando o CodeDeploy Agent

```	shell
# Instalando CodeDeploy Agent
$ sudo yum update -y
$ sudo yum install -y ruby wget

# Downloading CodeDeploy Agent
$ wget https://aws-codedeploy-sa-east-1.s3.sa-east-1.amazonaws.com/latest/install

$ ls -la 
$ chmod +x ./install
$ sudo ./install auto

# Verificando o status do CodeDeploy Agent
$ sudo service codedeploy-agent status
```

- Página de referência de download do agende codedeploy:
https://docs.aws.amazon.com/pt_br/codedeploy/latest/userguide/resource-kit.html#resource-kit-bucket-names

``` shell
# Downloading CodeDeploy Agent de outra região eu-west
$ wget https://aws-codedeploy-eu-west-1.s3.eu-west-1.amazonaws.com/latest/install 
```	

- Monstando o resulado do comando do status do CodeDeploy Agent

![alt text](image-1.png)

#### Instalando o Servidor Nginx


```	shell
# Instalando Nginx
$ sudo yum install -y nginx

# Checando Nginx status
$ sudo service nginx status

# Iniciando Nginx
$ sudo service nginx start

# Habilitando Nginx reiniciar
$ sudo systemctl enable nginx
```

- Servidor Nginx executando

![alt text](image-2.png)


#### Criando o diretório para as implantações

``` shell
$ sudo mkdir -p /var/www/my-flask-app
```

- Alterando a caminho do diretório inicial do servidor

``` shell
$ sudo nano /etc/nginx/nginx.conf
```

- Alterar o `root` para `/var/www/my-flask-app`


![alt text](image-3.png)

- Tecle `Ctrl + X` para sair, tecle no `Y` para salvar e clique no `Enter` para aprovar.

- No terminal da instância, executa os comandos definidos no arquivo ec2-instance-commands.txt
- Execute o seguinte comando para reiniciar os servidor

``` shell
$ sudo service nginx restart
```

- Acessando o servidor no navegador

![alt text](image-4.png)



### Criando a Service role de Deployment

- Acesse IAM, seção `Roles`, selecione para criar
- Selecione a opção de caso de uso `CodeDeploy`
![alt text](image-19.png)

- Atribua o nome `MyFirstServiceRoleFlaskApp`
- Finalize a criação da role

### Criando Aplicação

- Atribua nome `MyFirstFlaskApp`
- Selecione `EC2/On-premises`
- Clique em `Create Application`

![alt text](image-16.png)

#### Criando Grupo de Deploy

- Cliquei `Create Deployment Group`
![alt text](image-17.png)

- Atribua o nome `MyFirstFlaskAppDeployGroup`

![alt text](image-18.png)

- Atribuindo o Service Role `MyFirstServiceRoleFlaskApp`

![alt text](image-20.png)

- Na seção `Environment configuration`, selecione `Amazon EC2 instances`

![alt text](image-22.png)

- Atribuir os valores `Application` e `My Flask App Web Server`, esses dois valores são tags que foram criadas durante a criação da instância EC2, podem ser conferiadas na seção `Tags` da própria instância

![alt text](image-21.png)

- Desabilite o `Load balance`
- Cliquei em `Create deployment group`


### Criando o CodePipeline

- Acesse a página do CodePipeline, seção Pipelines e cliquei em `Create Pipeline`.

![alt text](image-5.png)

#### Selecionando o tipo de esteira

- Seleciona `Build custom pipeline`
- Cliquei em Next

![alt text](image-6.png)

- Atribua o nome `MyFirstFlaskAppPipeline`

![alt text](image-11.png)

- Selecinoe `Queued (Pipeline type V2 required)`
- Selecione `New Service Role`

![alt text](image-9.png)

#### Configurando Advanced Settings
- Selecione as seguintes opções na seção `Advanced Settings`

![alt text](image-10.png)

- Clique em Next

#### Definindo o Estágio de Origem

![alt text](image-12.png)

- Selecione GitHub App como o provedor de origem.
- Clique em `Connect to Github`
- Uma nova janele irá abri
- Atribua um nome para a conexão
- Siga as etapas seguinte para finalizar o login e conexão entre a aws e github

![alt text](image-13.png)

- Em seguinda, a `Connection` será preenchida
- Selecione o repositório e a branch que deseja conectar à pipeline

![alt text](image-14.png)

- Deixe habilitado o webhook
- Clique em Next

![alt text](image-15.png)

- Selecione `Skip build stage`

#### Definindo o Estágio Deploy
- Selecione a aplicação que foi criada `MyFirstFlaskApp`
- Selecione o `Deployment Group`
- Clique em Next e finalize a criação da pipeline

![alt text](image-23.png)


### Executando o deploy

- Logo que finaliza a criação a esteira começou a executar porém deu erro:

![alt text](image-24.png)

- Vamos realizar algumas instalações na instância:
    - python
    - ambiente virutal
    - flask
    - gunicorn

#### Instalando o Python
``` shell
$ sudo yum install python3 -y
$ python3 --version
```	

![alt text](image-25.png)

#### Instalando o Ambiente virtual e ativando

#### Instalando o Ambiente virtual e ativando

``` shell
$ sudo yum install python3 -y
$ sudo yum install python3-pip -y
$ pip3 install virtualenv
$ python3 -m venv myenv
$ source myenv/bin/activate
$ pip install flask
$ pip install gunicorn
```	

![alt text](image-26.png)

### Instalando o Log no EC2

- Precisa atribuir a política `CloudWatchAgentServerPolicy`à role da instância

![alt text](image-27.png)

#### Configurando o arquivo de log

```	shell
$ wget https://s3.amazonaws.com/amazoncloudwatch-agent/amazon_linux/amd64/latest/amazon-cloudwatch-agent.rpm
$ sudo rpm -U ./amazon-cloudwatch-agent.rpm
$ sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-config-wizard
```

![alt text](image-28.png)

![alt text](image-29.png)

![alt text](image-30.png)

![alt text](image-31.png)

![alt text](image-32.png)

![alt text](image-33.png)

- Quando finalizar a criação do arquivo, execute o comando e inclua o trecho: `"timestamp_format": "[%Y-%m-%d %H:%M:%S.%f]"`

``` shell
$ sudo nano /opt/aws/amazon-cloudwatch-agent/bin/config.json
```

![alt text](image-34.png)

- Demais comandos para iniciar/parar a execução do agente

``` shell
# Start CloudWatch agent 
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c file:/opt/aws/amazon-cloudwatch-agent/bin/config.json -s

# Stop CloudWatch agent
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a stop

# Check CloudWatch agent status
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```	


# Direcionar a página inicial

Abrir o arquivo de config e incluir o trecho no seção `server`:

``` shell
$ sudo nano /etc/nginx/nginx.conf
```

``` config
server { 
    listen 80; 
    server_name _; 

    location / { 
        proxy_pass http://127.0.0.1:5000; 
        proxy_set_header Host $host; 
        proxy_set_header X-Real-IP $remote_addr; 
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for; 
        proxy_set_header X-Forwarded-Proto $scheme; 
    }
}
```

``` shell
# Testar a configuração
$ sudo nginx -t
# Reinicializar o servidor
$ sudo systemctl restart nginx
```

- Acesse o link da EC2 para veficiar a página que está sendo exibida:

![alt text](image-35.png)