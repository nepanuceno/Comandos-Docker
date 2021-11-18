# Comandos-Docker
Tutorial explicando sobre os pricipais comandos da ferramenta Docker e como utiliza-los na prática.


# Primeiros passos com o Docker

## Contêineres
### Lista de contêineres que estão sendo executados
``` # docker ps ```

### Lista de todos contêineres, inclusive os parados
```# docker ps -a```

### Como executar um contêiner
```# docker run -it <image>```
O comando acima executa a imagem e abre um terminal interativo de acesso ao contêiner iniciado.
A imagem pode ser obtida a partir do [HubDocker](https://hub.docker.com/search?type=image) ou ser uma imagem "*buildada*" por você a partir de um *Dockerfile*. Daqui a pouco vamos aprender a gerar nossa própria imagem.

### Como executar um contêiner em segundo plano
``` # docker run -d <imagem>```
a *flag* -d informa ao *docker* que o contêiner deverá ser iniciado em segundo plano, ou seja, sem a necessidade de um terminal de acesso.

### Acessando o contêiner via porta específica
Na grande maioria dos casos, contêineres oferecem um determinado tipo de serviço, tais serviços são acessados por meio de rede, que por sua vez utiliza uma determinada porta. Por exemplos o apache, que utiliza a porta 80, ou a *MySql*, que utiliza a porta 3306, dentro outros.
Para acessar um contêiner via porta de rede, a porta deverá ser informada no ato a inicialização do mesmo. Exemplo:
``` # docker run -d -p 8080:80 nginx``` 

O exemplo que temos aqui é de um contêiner que fornece o serviço de servidor web. Comumente, o serviço está disponível na porta 80, que é a porta que está exposta no contêiner. Para obter acesso, devemos especificar a porta que irá se comunicar com o contêiner, que neste caso é a porta 8080.
No navegador, podemos acessar o serviço web da seguinte forma:
http://localhost:8080.

### Nomeando um contêiner
Você deve ter observado que após botar o contêiner pra rodar, ao listar os contêineres, eles têm um nome aleatório na a ver, mas, caso isso te incomode, você pode simplesmente criar seus contêineres com um nome mais relacionado à tarefa que ele vai servir. Para isso, utilize a *flag* ``--name`` seguida do nome que mais lhe agradar. Ex:
``` # docker run -d -p 8080:80 --name meu_nginx --rm nginx``` 

Nesse exemplo a cima, aproveitei a oportunidade para mostrar uma *flag* bastante útil, o `` --rm``. Esta opção serve para indicar que o contêiner deve ser destruído ao ser parado, isso evita que você, ao testar suas imagens e contêineres, tenha que ficar parando e depois excluindo o contêiner.

### Parando e reiniciando um contêiner
Por diversos motivos um contêiner deve ser parado e posteriormente reiniciado, perincipalmente durante a sua implementação. Para isso o *docker* oferece os comandos de *start* e *stop*

```# docker star <ID do contêiner> ```
```# docker stop <ID do contêiner> ```

Para descobrir o ID do contêiner, você pode executar o comando ``docker ps -a``, identificar o contêiner que deseja para e pegar o CONTAINER ID dele.

### Logs de um contêiner

Para observar o que está se passando com o contêiner, o *docker* oferece uma forma de exibir os *logs* pro usuário.
```# docker logs -f <ID>```
A *flag* ``-f`` força o *docker logs* a ficar observando continuamente o comportamento do contêiner e exibindo no terminal.

### Adicinando usuário ao grupo do docker

Neste ponto do tutorial, você já deve ter reparado que para utilizar os comandos do *docker* é exigido permissão de usuário. Para resolver essa chatice, adicione o seu nome de usuário ao grupo de usuários do *docker*.

```# sudo usermode -aG docker $(whoami) ```
Para efetivar a operação sem precisar reiniciar sua sessão de usuário, use o comando a seguir:
``` # sg docker```

### Removendo um contêiner

Para remover um contêiner, simplesmente execute o comando ``` # docker -rm <ID>```.
Certifique-se que o contêiner esteja parado antes de executar o comando de remoção, mas caso queira pará-lo e removê-lo de uma só vez, utilize o comando ``` # docker -rm -f <ID>```.  A *flag* -f forçará a parada para que a exclusão aconteça.

Até agora, vimos como gerenciar as principais ações em um contêiner, porem só utilizamos imagens pré prontas do repositório do *Docker*. Na próxima sessão vamos descobrir como podemos construir nossa própria imagem personalizada do *Docker*.

## Criando imagens
### O que é uma imagem *Docker*?
Uma imagem *Docker* é um conjunto de características e instruções que serão responsáveis pela construção e funcionamento dos seus contêineres de serviços.
Vamos entender melhor durante o desenrolar dessa sessão.

Uma imagem *docker* é definida por um arquivo chamado *Dockerfile*, que fica localizado na raiz do projeto que está sendo desenvolvido. Esse arquivo contem todas as instruções de como o contêiner deverá se comportar. A seguir, temos um exemplo de uma imagem baseada na imagem oficial do *nginx*.
```mermaid
	FROM nginx #imagem na qual a sua imagem irá se basear
	WORKDIR /var/www/html # diretório onde o seus projeto será hospedado
	COPY ./ ./ # copia o conteúdo do diretório corrente para o WORKDIR
	RUN npm install # executa um comando
	COPY ./ ./ # copia o conteúdo do diretório corrente para o WORKDIR
	EXPOSE 3000 # define a porta pela qual o container irá rodar o serviço
	# lista de comandos a serem executados
	CMD ["apt-get update", "apt-get upgrade","composer install"] 
```

Após definido o *Dockerfile*, já podemos construir a imagem. Para isso, vamos utilizar o comando:
```# docker build . ```
Esse comando procura um arquivo *Dockerfile* no diretório indicado e monta a imagem com as definições contidas nele.

Para executar a imagem gerada, vamos utilizar o nosso já conhecido comando:
 ``` # docker run <imagem>``` 
 > O contêiner será gerado e ficará disponível na porta definida no *Dockerfile*
 > Se a imagem for alterada, o *build* deverá ser executado novamente e o contêiner regerado.

### Baixar uma imagem sem gerar um contêiner
Às vezes precisamos deixar algumas imagens disponíveis para uma eventual necessidade. Neste caso, podemos baixá-las sem a necessidade de gerarmos um contêiner. Para isso temos o comando a seguir:
```# docker pull <imagem> ```

Quando for necessário gerar um contêiner dessa imagem, pode-se usar o ``` # docker run <imagem>``` para gerá-lo.


### Renomear imagem
Para renomear uma imagem, podemos usar o comando 
```# docker tag <imagem>:<versão> <nome_desejado>:<versão>```

Exemplo: ``` # docker tag nginx:latest meu_nginx:producao```

### Listar imagens
```# docker images``` para listar as imagens que estão sendo utilizadas 
```# docker images -a ``` para listar todas as imagens, mesmo as que não estão sendo utilizadas.

### Remover imagens
``` # docker rmi <imagem>```

Para remover uma imagem que está sendo utilizada, deve-se adicionar a *flag*  ``-f`` ao comando acima.

### Remover todas as imagens não utilizadas

Muito cuidado com este comando, use-o com muita sabedoria, pois você poderá excluir uma imagem simplesmente por ela não estar sendo utilizada no momento da execução do comando a seguir.

``` # docker system prune```

> Ao executá-lo, uma caixa de diálogo será apresentada pedindo a confirmação da ação.

## Mais ações com contêineres
### Modo interativo
Para executarmos um contêiner e deixá-lo disponível em um terminal interativo, podemos iniciá-lo passando uma *flag* ``-i``.
```# docker start -i <CONTAINER ID>```

#### Facilitando a vida
Muitas vezes temos que utilizar comando que levam como parâmetro o *CONTAINER ID*, que não é nada legível e amigável. Mas felizmente o *docker* já previu essa situação e disponibilizou um atalho pra facilitar a vida do *Devops*, basta digitar os primeiros caracteres do ID que o *docker* já irá entender qual é o contêiner a ser atingido no comando. 

### Copiar arquivos do contêiner para o computador hospedeiro
```# docker cp <CONTAINER ID>:<path de origem> <path de destino>``` 

### Informações de um contêiner
O comando top basicamente fornece as informações sobre o processo que está executando o referido contêiner.

``` docker top <CONTAINER ID>```

### Configurações do contêiner
O comando inspect fornece um objeto JSON com todas as informações a respeito de um contêiner, como redes, volumes, portas, variáveis de ambiente etc.
```# docker inspect <CONATINER ID>```

### Informações de processamento e memória ocupados por um contêiner

``` # docker  stats <CONTAINER ID>```

## Volumes

Volumes são os responsáveis por persistirem arquivos acessados por um contêiner e deixar esses arquivos disponíveis caso o contêiner seja parado ou até mesmo excluído, ou compartilhar estes mesmos arquivos com outros contêineres que tenham acesso aos respectivos volumes.
Basicamente temos 3 tipos de volumes no *docker*:

### Volume anônimo
O volume anônimo é gerado sem uma identificação muito legível. Seu nome é gerado de forma aleatória e o seu ciclo de vida só dura enquanto o contêiner existir, ou seja, se apagar o contêiner os dados serão perdidos. Para criar um contêiner que com um volume anônimo, utiliza-se o seguinte comando:
``` # docker run -d -p 80:80 --name my_container -v /data --rm my_image```
Agora existe um contêiner chamado *my_container* que possui um volume anônimo. Isso pode ser verificado com o comando ```# docker inspect my_container``` que irá retornar um array de objetos *JSON* com diversas informações sobre o contêiner. Ex:
```mermaid
"Volumes": {
	           "/data": {}
           },
```
	
### Volume nomeado
Este tipo de volume tem um nome amigável que pode ser facilmente referenciado. O comando para montar tal volume é muito semelhante ao anterior, porém, nesse tipo de volume deve ser especificado tanto o nome do volume quanto o *path* de onde ele deverá ser montado, que comumente é o caminho *WORKDIR* definido no *Dockerfile*. Ex:

``` # docker run -d -p 80:80 --name my_container -v my_volume:/var/www/html/messages --rm my_image```

Agora faça o seguinte teste: Crie um contêiner com o mesmo volume do contêiner criando anteriormente, mudando apenas o nome do contêiner e a porta de comunicação.

``` # docker run -d -p 81:80 --name my_container_2 -v my_volume:/var/www/html/messages --rm my_image```

Agora temos 2 contêineres que acessam o mesmo volume. O conteúdo armazenado por um contêiner pode ser acessado pelo outro contêiner.

### Volume criado manualmente
O Docker nos oferece uma possibilidade de criar um volume de forma manual, ou seja, criá-lo sem a necessidade de vinculá-lo a um contêiner no momento da sua criação. Para isso, temos o comando:
 ``` # docker volume create <nome_do_volume> ```
Agora, de posso de um volume, podemos referenciá-lo ao contêiner que desejarmos.

``` # docker run -d -p 82:80 --name my_container_3 -v <nome_do_volume>:/var/www/html/messages --rm my_image```

Agora podemos referenciá-lo ao contêiner que desejarmos, e todos os contêineres terão acesso ao mesmo volume. 


### Bind Mounts
O *bind mount* é um tipo de volume onde o local de gravação e leitura dos arquivos é na maquina hospedeira, no caso, o seu computador ou o servidor de hospedagem da aplicação.

``` # docker run -d -p 81:80 --name my_container_2 -v /home/usuario/curso_docker/volumes/messages:/var/www/html/messages --rm my_image```

Dessa forma, os arquivos agora aparecerão diretamente no seu projeto, na sua máquina e serão persistidos fora do contêiner.
Por ser fora do contêiner, o *Docker* não lista *bind mounts*, pois não é papel dele gerenciar este tipo de volume e sim do *Devops*.

#### Detalhe importante
Ao trabalhar em um projeto utilizando *docker*, nos necessitamos sempre estar atualizando o código fonte de nossas aplicações, e seria muito inconveniente se a cada vez que precisássemos atualizar algo tivéssemos que refazer o *build* da imagem e reexecutar a montagem do contêiner. Para resolver esse inconveniente, podemos utilizar o próprio *bind mount*, indicando o diretório de montagem para o *path* do projeto na maquina local e o destino para o *path* do diretório de publicação do projeto no contêiner, assim, toda vez que fizermos uma alteração no projeto, tudo já estará acessível pelo contêiner.
``` # docker run -d -p 81:80 --name my_container_2 -v /home/usuario/curso_docker/volumes:/var/www/html --rm my_image```

Dessa forma, podemos editar os códigos fonte a vontade sem nos preocuparmos com o *rebuild* da imagem e regerar o contêiner.


### Criando volumes somente leitura
Em alguns casos, o volume utilizado deve ter acesso restrito apenas para leitura do conteúdo armazenado. Dessa forma, o Docker dispõe 
de um parâmetro para especificar esse tipo de comportamento ao gerar o volume. Veja no exemplo a seguir:

``` # docker run -d -p 83:80 --name my_container_3 -v /home/usuario/curso_docker/volumes:/var/www/html:ro --rm my_image```

Observe que a única coisa a mudar na forma vista até agora é a adição de um parâmetro ':ro' ao final do PATH, o mesmo do WORKDIR no contêiner.

### Removendo volumes

Você já deve estar familiarizado com a estrutura dos comandos do Docker, afinal tudo sempre começa com ``` docker ``` seguido do nome da estrutura com a qual deseja trabalhar, seguida de uma ação e por fim o alvo dessa ação.  Tudo o que vem imediatamente após o comando docker diz muito sobre o que o usuário deseja trabalhar. Ex:
``` docker volume rm  <nome_do_volume>```

No exemplo do comando usado para remover um volume, podemos perceber a estrutura limpa e coerente que o Docker utiliza, isso torna as coisas mais intuitivas e ajuda o usuário Docker a dominar de forma mais fácil a ferramenta.

#### Removendo volumes em massa

Se precisarmos remover vários volumes de uma só vez, basta utilizar o comando seguinte:
``` # docker volume prune ```
Lembrando que os volumes que serão apagados são somente os que não estão sendo utilizados por algum contêiner.



## Networks

### Tipos de conexão

#### Externa
Aqui estamos num tópico muito interessante e útil, a comunicação dos nossos contêineres.
Um container pode se comunicar com a rede externa, intranet, internet etc. Essa comunicação é a mais comum dentre os contêineres, pois a maioria dos serviços executados por eles depende de algo externo a eles mesmo, por exemplo a comunicação com uma API ou serviços de terceiros.


#### Host

Essa modalidade consite na comunicação com o host local, ou seja, a máquina que estava executando o Docker (seu computador).

#### Entre contêiners

Esta é uma modalidade bastante usada para serviços distribuidos, onde cada serviço tem o seu próprio contêiner exclusivo. Cominicação entre contêineres ocorre muito em serviços web, onde o contêiner do servidor web se comunica com o contêiner do serviço de banco de dados, com o contêiner do serviço de *cache* e muitos outros.


### Tipos de redes (drivers)

- Bridge: o mais comum, utilizado na grande maioria dos casos de quando um contêiner preocisa se comunicar;
- Host: Pertmite a conexão entre contêiner e a máquina que está fazendo host do Docker.
- macvlan: Permite a conexão a um contêiner por meio de um endereço MAC;
- none: remove todas as redes de um contêiner;
- plugins: Permite plugins de terceiros para criar outros tipos de rede.


### Listando redes

Por padrão, o docker já define três redes. Confira executando o comando a seguir:

```# docker network ls```

Este comando lista todas as redes existentes no docker.

### Criando Redes

#### Rede bridge
Para criar uma rede, utiliza-se o comando ```# docker network create minharede ```. Observe que o tipo de rede foi omitido no comando de criação, porém o docker por padrão define o tipo de rede como bridge.

Uma rede do tipo bridge por padrão já se comunica com o mundo externo, sendo necessário apenas o programador da aplicação definir o endereço de comunicação no código fonte, por exemplo, um request à uma API remota.

#### Rede host
Para definir a comunicação com a máquina host, o endereço de request deverá ser ```'host.docker.internal'```, dessa forma o docker já saberá que a comunicação é com a maquina host.

#### Rede macvlan

Para definir um tipo de rede, o docker exige uma flag -d seguida do tipo de rede desejado. Veja no exemplo:

``` # docker network create -d macvlan ```

### Remover Rede

``` docker network rm minharede ```

#### Remover redes não utilizadas

``` docker network prune ```

Muito cuidado ao utilizar este comando, pois ele exlcuirá toadas as redes que não estiverem sendo utilizadas no momento, preservando sempre as redes padrão do Docker.

### Conectar dois contêineres

Primeiro vamos criar uma rede do tipo bridge para conectar os contêineres.

``` # docker network create minharede ```

De posse da rede, agora podemos executar o contêiner setando a rede que ele deverá utilizar. Para isso, o docker disponibiliza uma flag ``` --network ``` seguida do nome da rede que acabamos de criar no comando anterior.

```# docker run -d -p 3306:3306 --name mysql_container --rm --network minharede -e MYSQL_EMPTY_PASSWORD=True imagem_mysql```


Nesse comando podemos notar uma novidade, a flag ```-e``` seguida de uma definição de constante. Essa flag indica a criação de uma variável de ambinete no sistema do contêiner, essa variável de ambiente indicará ao myslq que ele poderá aceitar conexões com o password vazio. 

Pronto, acabamos de executar um container conectada à uma rede bridge personalizada, por onde a aplicação de outro contêiner também conctado à mesma rede irá acessar o banco mysql de forma satisfatória.

Na aplicação que se conectará ao serviço MySQL no contêiner que acabamos de gerar, temos em algum lugar o endereço indicando onde está rodando o serviço, supomos que seja uma plicação PHP Laravel. Neste caso, agora esse endereço passará a ser o nome do contêiner que forneçe o serviço de banco, no caso deste exemplo, será "mysql_container". No Laravel, essa configuração fica no arquivo .env na raiz do projeto. Ex:

```mermaind
APP_ENV=local
APP_DEBUG=true
APP_KEY=YOUR_API_KEY

DB_HOST=mysql_container
DB_DATABASE=YOUR_DATABASE
DB_USERNAME=YOUR_USERNAME
DB_PASSWORD=YOUR_PASSWORD

CACHE_DRIVER=file
SESSION_DRIVER=file
QUEUE_DRIVER=sync

MAIL_DRIVER=smtp
MAIL_HOST=mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
```

Agora vamos conectar mais um contêiner à rede 'minharede'

```# docker run -d -p 8080:80 --name laravel_container --rm --network minharede imagem_laravel```

O contêiner que acabou de ser criado possui o serviço de hospedagem de uma aplicação Laravel e o seu arquivo .env está configurado com a variável ```'DB_HOST=mysql_container'``` onde o valor de 'DB_HOST' é o nome do container que fornece o serviço do MySQL.

Pronto, agora temos a comunicação via rede entre dois contêineres.

### Conectar/Desconectar um contêiner a uma rede em tempo de execução

Temos também a possibilidade de conectar um contêiner já em execução a uma determinada rede. Primeiro temos que saber o CONTAINER ID respectivo ao contêiner que desejamos conectar à rede. Para isso, basta executar o ```# docker ps``` e identificar o contêiner desejado. Agora é só conecta-lo à rede desejada usando o comando a seguir:

```# docker connect <nome_da_rede> <CONTAINER_ID>```

Dessa forma, o contêiner será conectado à rede informada.


Para desconectar, vamos utilizar um comando bastante parecido
```# docker disconnect <nome_da_rede> <CONTAINER_ID>```

### Inspecionar um Rede

Para verificarmos as caracteristicas de uma rede, como data de criação, ip, tipo de conexão etc, podemos utilizar um comando para inspecionar a rede e obtermos essas e outras informações. Para inspecionar uma rede, podemos utilizar o comando *inspect* e indicar a rede alvo da inspeção. Ex:
``` docker network inspect <nome_da_rede>```

Pode-se utilizar o ID da REDE ao invés do nome para indocar qual rede inspecionar, e o resultado será o mesmo.


> Written with [StackEdit](https://stackedit.io/).
