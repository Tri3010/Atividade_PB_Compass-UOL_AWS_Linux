# Atividade Amazon AWS e Linux - PB Compass UOL

## Introdução

Este projeto visa automatizar a configuração de um ambiente AWS, a implementação de um servidor Apache e a criação de um script para monitorar o status do serviço.

## Pré-requisitos

- Conta AWS válida.
- Chave privada correspondente à chave pública a ser gerada (PB).
- Conhecimento básico de AWS, Linux, Apache, NFS, e scripts Bash.

## Configuração do Ambiente AWS

- Feito o login em minha conta AWS;
- Certificado que eu estava na região correta; 
- Criado par de chaves;
- Criada Instância Amazon EC2;
- Gerado um Elastic IP e anexei-o à minha instância;
- Liberei as portas de comunicação para acesso público no grupo de segurança da instância criada;
Já configurado em minha VPC:
- Criado Gateway de Internet;
- Criado rotas para acesso público.

1 - Criação do par de chaves

- Abri o console do EC2;
- Selecionei "Pares de Chaves" no painel de navegação;
- Escolhi "Criar Pares de chaves";
- Coloquei um "Name";
- Tipo de par de chaves foi RSA;
- Formato de arquivo foi Pem;
- Criado o par de chaves;
- O arquivo de chave privada é baixado automaticamente pelo navegador. Arquivo salvo em local seguro.

2 - Criação do Amazon Elastic File System (EFS)

- No console do EFS, cliquei em  "Criar sistemas de arquivo";
- Dei um nome para o sistema de arquivo;
- Selecionada a minha VPC padrão;
- Clicado em "Criar"


3 - Criação da instância Amazon EC2

- No console do EC2, cliquei em  "Executar Instância";
- Coloquei chave e valor nas tags (Name, CostCenter, Project);
- Escolhi imagem da instância - Amazon Linux 2;
- Escolhi tipo da instância - t3.small;
- Selecionei par de chaves criado anteriormente;
- Em "Rede", escolhido opção "Criar Grupo de Segurança", com allow SSH trafic from "Meu IP";
- Em "Configurar armazenamento" preenchi - 16 GiB, gp2 volume raiz;
- Em sistemas de arquivos, cliquei em "Editar";
- Selecinei "EFS";
- Selecionei o EFS que foi criado anteriormente com seu ponto de montagem.
- Cliquei em "Executar Instância"

* Sempre que a inastância não estava em uso, ela foi deixada em estado "Interrompido".

4 - Alocação de um endereço de IP Elástico e anexação a minha instância

- No console da VPC, procurei no painel lateral "IPs Elásticos";
- Cliquei nele e escolhi "Alocar IPs Elásticos";
- Deixei tudo como padrão e cliquei em "Alocar";
- Depois de alocado, selecionei-o e cliquei em "Associar endereço IP Elástico";
- Em tipo de recurso, selecionei "Instância"e selecionei a minha instância criada para a atividade;
- Clicado em "Associar".
 
5 - Liberação das portas de comunicação para acesso público

- No console do EC2, em Instâncias, selecionei minha instância;
- Na aba "Segurança", selecionei meu security group e editei as regras de entrada deixando da seguinte maneira:
Intervalo de Portas - 80     Protocolo - TCP   
Intervalo de Portas - 111    Protocolo - UDP
Intervalo de Portas - 2049   Protocolo - TCP
Intervalo de Portas - 443    Protocolo - TCP
Intervalo de Portas - 2049   Protocolo - UDP
Intervalo de Portas - 22     Protocolo - TCP
Intervalo de Portas - 111    Protocolo - TCP

Todos com Origem - 0.0.0.0/0

Os próximos passos, eu já havia configurado para comunicação pública. 
Mesmo não sendo necessário devido a anexação do IP elástico a minha instância, optei por colocar nessa documentação.

6 - Criação de um Gateway da Internet e associação dele a minha VPC

- No console da VPC, no painel lateral cliquei em "Gateways da Internet";
- Cliquei em "Criar Gateway da Internet";
- Coloquei chave e valor nas tags (Name, CostCenter, Project);
- Cliquei em "Criar Gateway da Internet";
- Selecionei o Gateway da Internet criado, cliquei em "Associar à VPC";
- Selecionei minha VPC;
- Meu Gateway da Internet foi associado à minha VPC.

7 - Criação Rotas para acesso público

- No console da VPC, no painel lateral cliquei em "Tabela de Rotas";
- Cliquei em "Tabela de Rotas";
- Coloquei chave e valor nas tags (Name, CostCenter, Project);
- Associei à minha VPC;
- Cliquei em criar tabela de rotas;
- Selecionei minha tabela de rotas criada.
- Na aba Rotas, adicionei uma rota com destino 0.0.0.0/0 e alvo meu gateway da internet.



## Requisitos no Linux

- Configurado EFS;
- Criado diretório patriciaMariaMoura dentro do EFS;
- Subido Apache no servidor - online e rodando;
- Criado script que valida o status do serviço e envia o resultado para patriciaMariaMoura no EFS;
- Script com DataHORA + nome do serviço + Status + mensagem personalizada de online ou offline;
- Script gera duas saidas, uma para serviço online e outra para offline;
- Execução automatizada a cada 5 minutos.


1 - Configuração EFS e criação do diretório patriciaMariaMoura

- Primeiro passo é a instalação do pacote amazon-efs-utils. 
Comando: sudo yum install -y amazon-efs-utils;
- Minha montagem de sistema de arquivo foi automática, pois anexei o EFS quando criei minha instância EC2 Linux2;
- Atualizei o arquivo /etc/fstab do EC2 com uma entrada para o sistema de arquivos do EFS;
- Criado diretório dentro efs. 
Comando: mkdir mnt/efs/patriciaMariaMoura


2 - Subido Apache no servidor

- Conectada a minha instância EC2, instalei o servidor Web Apache.
Comando: sudo yum -y install httpd

- Iniciei o serviço.
Comando: sudo service httpd start  

- Certificado que o serviço estava operando através da "Test Page" do Apache, colocando o IP elástico no browser. 


3 - Criado script para validar serviço.

- Conectada em minha instância EC2, criei arquivo nano dentro do meu diretório patriciaMariaMoura
Comando: nano /mnt/efs/patriciaMariaMoura/scriptApache
- Código do script foi escrito nesse arquivo. 

4 - Preparação da execução automatizada do script a cada 5 minutos.
 
- Aberto o arquivo cronjobs;
Comando: crontab -e
- Agendar a execução a cada 5 minutos. Adicionei uma linha ao arquivo.
Comando: */5 * * * * /mnt/efs/patriciaMariaMoura/scriptApache


Versionamento

O versionamento deste projeto é gerenciado utilizando Git. Cada versão está marcada com uma tag correspondente.

  


 

