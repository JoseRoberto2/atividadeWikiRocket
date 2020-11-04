Primeiro prepare o ambiente kubernets e docker para seguir os próximos passos.

Este projeto desenvolvi no SO Windows 10 com o docker 19.03 kubernets 1.19 do minikube 1.14.2

Dividi os arquivos de configuração para facilitar o processo pela ordem de execurção:

## NAMESPACE ##
Crie primeiro o namespace "atividade" que servirá para este projeto.

``kuberctl.exe apply -f .\namespace.yaml``

## VOLUME PERSISTENTE ##
Crie o volume persistet e o volume persistent claim que servirá para armazenar os dados persistente do banco postgres. Use o arquivo ``storagepostgres.yaml``.

``kuberctl.exe apply -f .\storage-postgres.yaml``

---
# WIKI.JS COM O BANCO POSTGRES #
---

## BANCO POSTGRES ##

* **CONFIMAPS** 
Primeiro crie o configmpas com o arquivo ``configmap-postgres.yaml`` para definir algumas variaveis de ambiente do postgress.

``kubectl.exe applly -f .\configmap-postgres.yaml``

* **DEPLOYMENT** 
Agora vamos criar o deploy do banco para vincular as variaveis, o volume e definir a porta, definir as replicas para o nosso nodes e a imagem para o nosso pod.

``kubectl.exe applly -f .\deploy-postgres.yaml ``


* **SERVICE**
Por ultimo vamos definir o serviço da aplicação do banco, definindo a porta padrão 5432 para consumir externamete e vamos deixar tipo como loadBalance:

``kuberctl.exe applly -f .\service-postgres.yaml``

## PARA O WIKI.JS ##

Já temos o banco preparado para a aplicação wiki.js. Vamos criar as nossas API para o wiki na ordem:

* **CONFIGMAPS**
Primeiro vamos colocar a nossa configMaps para a variaveis da aplicação com o arquivos ``configmapswiki.yaml``

``kubectl.exe applly -f .\configmap-wiki.yaml``

O kubernet consegue resolver internamente o nome DNS do serviço postgres para o host do banco da aplicação.

* ** DEPLOYMENTS**
Vamos definir o deploy da aplicação no arquivo ``deploy-wiki.yaml``. Vamos usar a versão 2.

``kubectl.exe applly -f .\deploy-wiki.yaml ``

* **SERVICE**
Finalizamos a instalação no kubenets criando o serviço do WIKI para acessar na porta 3000 com o arquivo.

``kubectl.exe applly -f .\service-wiki.yaml``


** CONFIGURAÇÃO INICIAL DO WIKI.JS **
* Instalado o ambiente, precisamos definir o e-mail do administrador, senha, url e se vamos pemitir a coleta de dados anonimos para para o wiki.js.

* Depois precisamos criar a primeira página para o home. Temos as opções de criar a página em ``Markdown RichText  HTML``


* Para criar a página temos que preencher o Título, o subtitulo , a linguagem, a pasta e a categoria. Podemos definir também a data de publicação.

* Nas configuração podemos adicionar idioma, adicionar metodo de autenticação, adicionar usuário local e outras opções.

---
# ROCKET.CHAT COM O MONGO #
---

O projeto do RocketChat não foi concluido. Tive algum problemas com o replicas set e com a persistencia do banco. Realizei a implementação dessa formas, mas na primeira falha do nosso pod, o sistema não ficou operacional e não conseguiu recuperar. 
## PARA O BANCO MONGO ##

* **STATESFULL** 
Agora vamos criar o deploy do banco para vincular as variaveis, o volume e definir a porta, definir as replicas para o nosso nodes e a imagem para o nosso pod.

Vamos usar um conceito diferente para o mongo. Vamos usar o statesfull para garantir a ordem e exclusividade do pods e por ser a recomendada para esse serviço de banco de dados.

O stefull não usa o VolumePersistent precisamos criar um StorageClass. Atenção para o campo ``provisioner``. Pode mudar esse plugin para outros tipos de volume.

``kubectl.exe applly -f .\storage-mongo.yaml``

Crie o Statesfull do mongo.

``kubectl.exe applly -f .\statesfull-mongo.yaml ``


* **SERVICE**

Definimos o serviço também na porta padrão 27017.

``kuberctl.exe applly -f .\service-mongo.yaml``

Precise entrar no pod e definir algums paramêtros no mongo. Execute o seguinte comando para entrar no container:

``kubectl.exe exec -it mongo-0 -- bash``

Entre no mongo com o comando ``mongo`` e depois digite os comandos para definir os menbros do replicaSet do mongo:

```
rs.initiate()
var cfg = rs.config()
cfg.members[0].host="mongo-0.mongo:27017"
rs.reconfig(cfg)
rs.add("mongo-1.mongo:27017")
```

## PARA O ROCKET.CHAT ##

Vamos seguir na ordem abaixo.

* **CONFIGMAPS**
Vamos definir algumas variaveis do mongo para executar a nossa aplicação ``configmaps-rocketc.yaml``

``kubectl.exe applly -f .\configmap-rocket.yaml``

Temos o serviço do banco rodando com o nome ``mongo-1.mongo mongo-0.mongo``.

* ** DEPLOYMENTS**
Crie o nosso deploy da aplicação.

``kubectl.exe applly -f .\deploy-rocket.yaml ``

* **SERVICE**
Por padrão o service roda na porta 3000, mas para não ter conflito com o wiki vamos usar a porta 3001 no nosso service.

``kubectl apply -f .\service-roket.yaml``


** CONFIGURAÇÃO INICIAL DO ROCKET CHAT **

* O primeiro acesso é necessário o Nome, nome do usuario, e-mail e senha para o administrador.

* Mais algumas informações são necesárias na página 2 referente a organização ou a empresa. Tipo da organização, nome, tamanho, país e site.

* A página 3 tem informações do servidor, nome, idioma, tipo do servidor e se deseja autenticação em duas etapas.

* Na última opção pegunta se deseja compartilhar os dados e receber notificações.

Pronto, finalizado a configuração com as pendências descrita acima do mongo.