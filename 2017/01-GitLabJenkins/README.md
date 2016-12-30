GitLab + Jenkins: Uma integração poderosa
=========================================


![Integracao_Docker](imagens/integ.png)

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
  * **ports:** Você pode definir quais portas serão abertas interna e externamente no container. *Todos os padrões para definição de portas podem ser encontrados [aqui](https://docs.docker.com/compose/compose-file/#/ports)*.
  * **volumes:** Monta *paths* ou volumes nomeados, opcionalmente especificando um caminho no host. *Todos os tipos de montagem de volume [aqui](https://docs.docker.com/compose/compose-file/#volumes-volumedriver)*.


Essa é só uma explicação básica da estrutura do YAML para o docker-compose que iremos utilizar. Caso você queira se aprofundar ainda mais no assunto, a documentação oficial pode ajudar nisso :smiley:. Voce pode acessá-la  *[aqui](https://docs.docker.com/compose/compose-file/).*

:warning: A identação é essencial para o funcionamento do docker-compose.

Agora que o *docker-compose.yml* foi configurado e entendido, vamos começar com oque realmente interessa. *Let's go! :goberserk:*
