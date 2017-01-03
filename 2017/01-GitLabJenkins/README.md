GitLab + Jenkins: Uma integração poderosa
=========================================


![Integracao_Docker](imagens/integ.png)

Fala pessoal!




O início
-------------
Para iniciarmos nosso projeto, precisamos ter o [Docker](https://www.docker.com/) instalado, pois, com ele, subiremos os serviços do GitLab e do Jenkins através do [docker-compose](https://docs.docker.com/compose/) *(que falaremos mais abaixo* :sweat_smile:*).*
Para efetuar a instalação, verifique qual SO você está utilizando e *Go!Go!Go!* :feelsgood:. É só seguir esse [tutorial](https://docs.docker.com/engine/installation/).

Para a instalação do docker-compose é só seguir a [documentação oficial](https://docs.docker.com/compose/install/) que vai dar certo, pode confiar. :metal:

Docker Compose
--------------
Com o docker-compose, podemos criar e configurar e subir todos os serviços que vamos necessitar de uma só vez, com um único comando,  e para isso, vamos criar um arquivo ``` docker-compose.yml ``` em uma pasta de sua preferencia, e copiar todo o conteúdo abaixo para dentro dele. Não se preocupe, vou explicar cada item. :relieved:
Para mais informações, é só acessar [aqui](https://docs.docker.com/compose/overview/).




```yml

version: '2'

services:
    ambassador:
      image: cpuguy83/docker-grand-ambassador
      container_name: ambassador
      hostname: ambassador01
      network_mode: "bridge"
      volumes:
        - "/var/run/docker.sock:/var/run/docker.sock"
      command: "-name jenkins -name gitlab "

    jenkins:
      image: jenkins:latest
      container_name: jenkins
      hostname: jenkins01
      network_mode: "bridge"
      links:
        - "ambassador:gitlab"
      ports:
        - "8080:8080"
        - "50000:50000"
      volumes:
        - ~/Projects/jenkins_home:/var/jenkins_home

    gitlab:
        image: gitlab/gitlab-ce
        container_name: gitlab
        hostname: gitlab.teste.com
        restart: always
        network_mode: bridge
        links:
          - "ambassador:jenkins"
        ports:
           - "443:443"
           - "8050:80"
           - "22:22"
        volumes:
          - ~/Projects/gitlab/config:/etc/gitlab
          - ~/Projects/gitlab/logs:/var/log/gitlab
          - ~/Projects/gitlab/data:/var/opt/gitlab


```


* **version: '2'**: Utilizando a versão 2 da sintaxe do docker-compose, conseguiremos utilizar todas as novas funcionalidades disponíveis do Compose e do Docker Engine.

* **services**: Todos os serviços que serão utilizados em nosso projeto devem estar listados abaixo dele. *(A estrutura de services só funciona com a versão 2 da sintaxe).*

* **nome-do-serviço**: Nome para identificar o serviço no docker-compose.
  * **image:** Imagem docker que será utilizada para o serviço.
  * **container_name:** Atribui um nome ao container, tornando mais fácil a administração.
  * **hostname:** Adiciona um nome ao host. Utilizaremos os hostnames nas próximas etapas.
  * **network_mode:** Configurações de rede para o container. O modo bridge sobe todos os containers na mesma rede.
  * **links:** Faz o link entre os containers, para que a comunicação por nome seja possível. (Falei que o hostname seria importante :relieved:)
  * **ports:** Você pode definir quais portas serão abertas interna e externamente no container. Todos os padrões para definição de portas podem ser encontrados [aqui](https://docs.docker.com/compose/compose-file/#/ports).
  * **volumes:** Monta *paths* ou volumes nomeados, opcionalmente especificando um caminho no host. Todos os tipos de montagem de volume [aqui](https://docs.docker.com/compose/compose-file/#volumes-volumedriver).


Essa é só uma explicação básica da estrutura do YAML para o *docker-compose file* que iremos utilizar. Caso você queira se aprofundar ainda mais no assunto, a documentação oficial pode ajudar nisso :smiley:. Voce pode acessá-la  *[aqui](https://docs.docker.com/compose/compose-file/).*

:warning: A identação é essencial para o funcionamento do docker-compose.

Agora que o *docker-compose.yml* foi configurado e entendido, vamos começar com oque realmente interessa. *Let's go! :goberserk:*


Serviços
--------

Antes de prosseguirmos com o projeto, uma breve explicação sobre cada serviço que estamos subindo.
Voltaremos com a programação normal em seguida, nesse mesmo canal, e nesse mesmo horário.


#### Docker Grand Ambassador

Esse serviço permite a comunicação bi-direcional entre containers. Isso significa que ele criará automaticamente um proxy em todas as portas expostas, e também detectará automaticamente as alterações feitas em um container e ajustará o servidor proxy de acordo com a necessidade.(Como por exemplo, um container é reiniciado e seu IP muda). Ou seja, ele vai nos poupar o trabalho de fazer um servidor DNS. :trollface:

Voce pode ler mais sobre o Docker Grand Ambassador [aqui](https://github.com/cpuguy83/docker-grand-ambassador).

### Jenkins

O [Jenkins](https://jenkins.io/) vai permitir a automatização dos processos no nosso projeto. Podemos automatizar testes, builds e etc., escolhendo quais serão as ações que vão acionar nosso *Job*.

Veremos mais sobre essa integração mais abaixo.


### GitLab

O Gitlab vai ser o nosso gerenciador de repositório nesse projeto, ele é muito parecido com o [GitHub](https://github.com/), mas com a vantagem de que podemos subir o serviço localmente, e a integração dele com o Jenkins é muito grande (Que por coincidência, é o propósito desse post :suspect:). Através de um WebHook vamos conseguir a comunicação direta com o Jenkins, eliminando a necessidade do Jenkins ficar checando o reposítório constantemente. O GitLab vai informar o Jenkins quando um evento ocorre(um *Merge Request* de uma *feature branch* na *branch Develop* por exemplo) e com isso um Job ligado ao Jenkins pelo WebHook é iniciado.

Agora depois de tanta <s>enrolação</s> explicação, é hora da mágica acontecer.

![Magic](imagens/magic.gif)
