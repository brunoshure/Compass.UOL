# Atividade 1 Linux


**Objetivo**: Criar um ambiente AWS com uma instância EC2 e configurar o NFS para armazenar dados.

**Escopo**: A atividade incluirá a geração de uma chave pública de acesso, criação de uma instância EC2 com o sistema operacional Amazon Linux 2, geração de um endereço IP elástico e anexá-lo à instância EC2, liberação de portas de comunicação para acesso público, configuração do NFS, criação de um diretório com o nome do usuário no filesystem do NFS, instalação e configuração do Apache, criação de um script para validar se o serviço está online e enviar o resultado para o diretório NFS, e configuração da execução automatizada do script a cada 5 minutos.

---
## Requisitos

### Instância AWS:
- Chave pública para acesso ao ambiente
- Amazon Linux 2
    - t3.small
    - 16 GB SSD
- 1 Elastic IP associado a instância
- Portas de comunicação liberadas
    - 22/TCP (SSH)
    - 111/TCP e UDP (RPC)
    - 2049/TCP/UDP (NFS)
    - 80/TCP (HTTP)
    - 443/TCP (HTTPS)

### Configurações Linux:

- Configurar o NFS entregue;
- Criar um diretório dentro do filesystem do NFS com seu nome;
- Subir um apache no servidor - o apache deve está online e rodando;
- Criar um script que valide se o serviço esta online e envie o resultado da validação para o seu diretório no nfs;
    - O script deve conter - Data HORA + nome do serviço + Status + mensagem personalizada de ONLINE ou offline;
    - O script deve gerar 2 arquivos de saída: 1 para o serviço online e 1 para o serviço OFFLINE;
    - Execução automatizada do script a cada 5 minutos.
---
## Passo a Passo



### Criando uma VPC
- Na AWS busque por VPC.
- Clique em `Criar VPC`.
- Selecione no menu de criação a opção `VPC e muito mais`.
- Crie a VPC com as configurações automáticas e pré-estabelecidas.


### Criando um Grupo de Segurança
- No menu EC2 procure por Grupo de Segurança.
- Acesse e clique em `Criar novo grupo de segurança`.
- Dê um nome e descrição.
- Selecione a VPC criada anteriormente.
- Em regras de entrada, adicione as regras para liberar as portas (conforme requerido na atividade):

| TCP personalizado | TCP | 80 | 0.0.0.0/0 | HTTP |
| TCP personalizado | TCP | 443 | 0.0.0.0/0 | HTTPS |
| TCP personalizado | TCP | 111 | 0.0.0.0/0 | RPC |
| TCP personalizado | TCP | 2049 | 0.0.0.0/0 | NFS |
| UDP personalizado | UDP | 111 | 0.0.0.0/0 | RPC |
| UDP personalizado | UDP | 2049 | 0.0.0.0/0 | NFS |

- Para finalizar clique em `Criar grupo de segurança`.


### Criando uma instancia EC2 na AWS
- Na AWS busque pelo serviço de EC2.
- Clique em `executar uma instância`.
- Na aba de Configuração Nome e Tags adione o nome e tags necessarias.
- Em imagens de aplicação e de sistema operacional, busque e selecione Amazon Linux 2.
- Em Tipo de instância selecione a instância t3.small.
- Selecione a opção pra criar um par de chaves (.pem para openSSH e .ppk para PuTTY).
- Em Configuração de Rede Selecione a VPC criada anteriormente.
- Ainda em Configuração de Rede selecione o grupo de segurança criado anteriormente.
- Na aba Configurar armazenamento selecione um disco de 16GB padrão gp2.
- Ao terminar as configurações basta clicar em `Executar Instância` para iniciar a criação.

### Criando um Elastic IP
- No menu EC2 procure no menu de serviços a esquerda pelo serviços Elastic IP.
- No menu de Elastic IP clique em alocar endereço IP elástico.
- No menu de criação verifique se a região está correta e IPv4 está selecionado, e clique em alocar.
- No menu de Elastic IP clique em `Ações` selecione Associar endereço IP elástico, marque instância e selecione a instância criada previamente após isso clique em `Associar endereço de IP elástico`.

### Acessando a instância via Putty
- Baixe e instale o [Putty].
- Em `Session` no campo Host Name, insira o public DNS(IPv4) da instância EC2.
- Acesse `Connection` > `Data` e no campo `Auto-login username`, digite `ec2-user`.
- Acesse `Connection` > `SSH` > `Auth` > `Credentials` e no campo `Private key file for authentication` selecione o arquivo .ppk da chave criada previamente.


### Instalando Apache em uma instância EC2
- Execute o comando para instalar o apache server e suas dependências:
```
sudo yum install -y httpd
```
- Executando depois disso o comando para iniciar o apache server:
```
sudo systemctl enable --now httpd.service
```
- Execute para verificar o status do apache server:
```
sudo systemctl status httpd
```
- Caso o servidor esteja rodando corretamente agora deve ser possível acessar a página de teste do apache através do ip elástico anexado a instância.

### Criando um EFS(NSF SERVER)
- Busque por EFS na Amazon AWS o serviço de arquivos de NFS escalável da AWS.
- Na Página de EFS clique em `Criar sistema de arquivos`.
- Escolha um nome para o EFS e selecione uma VPC(a mesma da instância) e clique em `Criar`.
- Acesse a pagina de configuração do EFS
- Uma vez criado o EFS navegue até network, e acesse gerenciar dentro dos detalhes do EFS e altere os security group para aquele que foi criado anteriormente.


### Criando uma pasta compartilhada EFS EC2
- Acesse a instância, e digite o comando para instalar as dependencias necessárias para acessar o EFS na instância do EC2:
```
sudo yum install -y amazon-efs-utils
```
- Uma vez que a instalação foi concluida, crie um diretorio para compartilhar entre a instância EC2 e o EFS através do comando:
```
sudo mkdir /home/ec2-user/efs
```
- É necessário montar o EFS no diretório através do comando:
```
sudo mount -t efs <EFS_FILE_SYSTEM_ID>:/ /home/ec2-user/efs
```

- Uma vez concluído o processo de montagem do EFS crie um diretório para armazenar os logs dentro do EFS com o comando:
```
sudo mkdir /home/ec2-user/efs/logs
```

### Criando um Script
- Crie um arquivo .sh para criar o script e abra-o com o editor de texto padrão através do comando:
```
sudo vi check.sh
```
- Copie o script feito anteriormente no editor de texto.
- Feche o editor de texto, para isso pressione "esc" para entrar no modo de comando, digite :wq e pressione enter.
- Habilite as permissões necessárias para o script, para isso execute o comando:
```
sudo chmod +x check.sh
```
- Teste a execução do script executando-o pelo comando:
```
sudo /home/ec2-user/check.sh
```

### Utilizando Crontab para automatizar a execução do Script
- Abra o crontab através do comando: 
```
crontab -e
```
- Pressione "i" para entrar no modo edição do editor.
- Digite `*/5 * * * * sudo /home/ec2-user/check.sh`.
- Verifique se o serviço cron esta sendo executado:
```
sudo systemctl status crond
```
- Caso o serviço não esteja em exeução, inicie-o com o comando abaixo: 
```
sudo systemctl start crond
```
