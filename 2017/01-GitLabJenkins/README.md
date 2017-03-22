GitLab + Jenkins: Uma integração poderosa
=========================================


![Integracao_Docker](imagens/integ.png)

Fala pessoal! Hoje vamos falar um pouco aqui no Blog sobre a integração entre o GitLab e o Jenkins, que pode adiantar muito a sua vida. Vamos lá?

O início
-------------
Para começar nosso projeto, a primeira coisa que precisamos é ter o [Docker](https://www.docker.com/) instalado, pois é com ele que vamos subir os serviços do GitLab e do Jenkins por meio do [docker-compose](https://docs.docker.com/compose/) *(vamos falar mais pra frente* :sweat_smile:*).*
Para instalar, verifique qual SO você está usando e *Go!Go!Go!* :feelsgood:. É só seguir esse [tutorial](https://docs.docker.com/engine/installation/).

Para a instalação do docker-compose é só seguir a [documentação oficial](https://docs.docker.com/compose/install/) que vai dar certo, pode confiar. :metal:

Docker Compose
--------------
Com o docker-compose podemos criar, configurar e subir todos os serviços que vamos precisar de uma só vez, com um único comando. Para isso, vamos criar um arquivo ``` docker-compose.yml ``` na pasta de sua preferência e copiar todo o conteúdo abaixo para dentro dele. Não se preocupe, vou explicar cada item. :relieved:
Se você quiser saber mais, é só acessar [aqui](https://docs.docker.com/compose/overview/).




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
        hostname: gitlab
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


* **version: '2'**: Com a versão 2 da sintaxe do docker-compose conseguimos usar todas as novas funcionalidades disponíveis do Compose e do Docker Engine.

* **services**: Todos os serviços que serão utilizados no nosso projeto devem estar listados abaixo dele. *(A estrutura de services só funciona com a versão 2 da sintaxe).*

* **nome-do-serviço**: Nome para identificar o serviço no docker-compose.
  * **image:** Imagem docker que será utilizada para o serviço.
  * **container_name:** Atribui um nome ao container, tornando mais fácil a administração.
  * **hostname:** Adiciona um nome ao host. Vamos usar os hostnames nas próximas etapas.
  * **network_mode:** Configurações de rede para o container. O modo bridge sobe todos os containers na mesma rede.
  * **links:** Faz o link entre os containers, para que a comunicação por nome seja possível. (Falei que o hostname era importante :relieved:)
  * **ports:** Você pode definir quais portas serão abertas interna e externamente no container. Todos os padrões para definição de portas podem ser encontrados [aqui](https://docs.docker.com/compose/compose-file/#/ports).
  * **volumes:** Monta *paths* ou volumes nomeados, especificando um caminho no host ou não. Todos os tipos de montagem de volume [aqui](https://docs.docker.com/compose/compose-file/#volumes-volumedriver).


Essa é só uma explicação básica da estrutura do YAML para o *docker-compose file* que vamos utilizar. Caso você queira se aprofundar ainda mais no assunto, a documentação oficial pode ajudar nisso :smiley:. Você pode acessá-la  *[aqui](https://docs.docker.com/compose/compose-file/).*

:warning: A identação é essencial para o funcionamento do docker-compose.

Agora que o *docker-compose.yml* foi configurado e entendido, vamos começar com oque realmente interessa. *Let's go! :goberserk:*


Serviços
--------

Antes de prosseguir com o projeto, uma breve explicação sobre cada serviço que estamos subindo.
Voltaremos com a programação normal em seguida, nesse mesmo canal e nesse mesmo horário.

#### Docker Grand Ambassador

Esse serviço permite a comunicação bidirecional entre containers. Isso significa que ele criará automaticamente um proxy em todas as portas expostas, e também detectará automaticamente as alterações feitas em um container ajustando o servidor proxy de acordo com a necessidade.(Por exemplo, um container é reiniciado e seu IP muda). Ou seja, ele vai nos poupar o trabalho de fazer um servidor DNS. :trollface:

Você pode ler mais sobre o Docker Grand Ambassador [aqui](https://github.com/cpuguy83/docker-grand-ambassador).

### Jenkins

O [Jenkins](https://jenkins.io/) permite a automatização dos processos no nosso projeto. Podemos automatizar testes, builds e etc., escolhendo quais serão as ações que vão acionar nosso *Job*.

Vamos ver mais sobre essa integração ainda neste post.


### GitLab

O Gitlab vai ser o nosso gerenciador de repositório. Ele é muito parecido com o [GitHub](https://github.com/), mas com a vantagem de podermos subir o serviço localmente. Além disso, a integração dele com o Jenkins é muito grande (aliás esse é o propósito deste post :suspect:). Por meio de um WebHook vamos conseguir a comunicação direta com o Jenkins, eliminando a necessidade de ele ficar checando o repositório constantemente. O GitLab vai informar o Jenkins quando um evento ocorre (um *Merge Request* de uma *feature branch* na *branch Develop*, por exemplo) e com isso um Job ligado ao Jenkins pelo WebHook é iniciado.

Agora, depois de tanta <s>enrolação</s> explicação, é hora de a mágica acontecer.

![Magic](imagens/magic.gif )

Subindo o ambiente
==================

### Iniciando os serviços

No diretório em que você criou o arquivo ```docker-compose.yml``` execute o comando:

```bash
  docker-compose up
```
A saída deverá ser parecida com essa. (Pelo menos eu espero :no_mouth:):

![ComposeUp](imagens/composeup.png)

Caso você não tenha as imagens docker dos serviços que estamos subindo, o docker-compose vai fazer o download automaticamente.

### Jenkins

Após a inicialização dos serviços com o docker-compose, vamos fazer a configuração inicial do Jenkins. Acesse a url [http://localhost:8080/](http://localhost:8080/) e você será direcionado para a página inicial do Jenkins.

![JenkinsIni](imagens/jenkins.png)

Para configurá-lo, vamos inserir a chave que ele gerou no momento da instalação. Existem duas maneiras de encontrar a chave, a mais simples é digitar no terminal ```docker logs -f jenkins```. Ele vai exibir assim:

![JenkinsPass](imagens/jenkinspass.png)  

Há outro modo que é acessando a pasta ```/var/jenkins_home/secrets/initialAdminPassword```, mas dá muito trabalho, melhor ficarmos com o primeiro mesmo. :sleeping:

Após colocar a senha inicial, o Jenkins vai exibir a página de customização dos plugins iniciais. Selecione **Install sugested plugins** e aguarde o donwload e instalação.

![CustomizeJenkins](imagens/customizejenkins.png)

Pronto, o Jenkins já está pronto para ser utilizado!


### GitLab

Com o GitLab o processo é bem mais simples. Só acessar a página inicial dele [http://localhost:8050/](http://localhost:8050/), e colocar uma senha com no mínimo 8 caracteres.

![GitlabSenha](imagens/gitlab_senha.png)

Agora estamos com o ambiente pronto e podemos começar a configuração da integração. Segura que o filho é seu!

<img src="imagens/son.gif" height="470" width="680">



### Criação do repositório

Vamos criar um repositório no Gitlab para armazenar o nosso projeto.(São só dois arquivos, mas faz de conta que é um projeto :trollface:)

Na tela inicial, clique em _New Project_, preencha o nome para seu projeto e clique em _Create Project_.

![GitlabCriar](imagens/gitlab_criar.png)

Após a criação do repositório, vamos adicionar os dois arquivos para execução de um teste unitário em Python.

O primeiro faz validação muito simples de email, apenas para esse exemplo. Ele recebe um email e retorna se é verdadeiro ou falso seu houver um @.

*mail.py*

```
def is_valid(email):
  return email and '@' in email
```

O segundo arquivo, vamos fazer para automatizar o teste e facilitar nossa vida.(é para isso que estamos aqui :sunglasses:)

*mail_test.py*


```
import unittest
import mail
class MailTest(unittest.TestCase):
  def test_should_return_true_when_email_is_valid(self):
      self.assertTrue(mail.is_valid('iam@gustavohenrique.net'))
  def test_should_return_false_when_email_is_invalid(self):
      self.assertFalse(mail.is_valid('xxxxx'))
if __name__ == '__main__':
  unittest.main()

```
[Fonte dos códigos em Python](http://www.concretesolutions.com.br/2016/11/23/devops-ferramentas-pos/)

### Configurando o Jenkins

Vamos precisar do [GitLab Plugin](https://wiki.jenkins-ci.org/display/JENKINS/GitLab+Plugin) para conectarmos no repositório do nosso projeto.(Lembre-se, são só dois arquivos, mas vamos manter a positividade).
Para instala-lo, vá até [Gerenciar Jenkins]->[Gerenciar Plugins], clique na aba _Disponíveis_.

Com o plugin instalado, vamos adicionar uma conexão com o GitLab, vamos seguir alguns passos simples e <s>talvez</s> vamos chegar lá! :astonished:

##### Criando um access token

Clique no círculo com sua imagem de perfil, no canto superior direito e vá em *Profile Settings* e clique na aba *Access Tokens*.
Digite um nome fácil de identificar  para seu token e clique em *Create Personal Access Token*.

![CriarToken](imagens/gerando_token.png)

Após a criação, copie o hash exibido na tela, utilizaremos ele no próximo passo.

![CriarToken](imagens/token_gerado.png)

:warning: O token desaparece após atualizar a página, então guarde em lugar seguro.


#### Adicionando a credencial no Jenkins

Na tela inicial do Jenkins, vá até Credentials. Clique na seta do lado de global e selecione *Add Credentials*.

![CriarToken](imagens/clic_add.png)

Na janela a seguir, selecione *GitLab API Token* e coloque o hash gerado no GitLab no campo API token.

![AddCred](imagens/add_cred.png)

Após clicar em OK, você verá sua credencial criada.

![OK](imagens/entrada_ok.png)

Vamos ao próximo passo!


#### Configurando a conexão Gitlab x Jenkins

Com a nossa credencial criada vamos em [Gerenciar Jenkins]->[Configurar o sistema]. Desça até a Gitlab e preencha as informações conforme a imagem abaixo, ao terminar, clique em *Test Connection*. Se tudo estiver certo clique em *Salvar*.

![GitLabConnection](imagens/gitlab_conn.png)

Depois de tantas configurações, vamos em frente.:goberserk:


#### Criação do Job

No canto esquerdo, clique em _Novo Job_, escolha _Projeto Freestyle_, digite um nome para o job e clique em _OK_.

![JenkinsJob](imagens/jenkins_criar.png)

Com o Job criado, vamos as configurações.

![Thinking](imagens/zak.gif)


###### General
Adicione a conexão que criamos no campo *GitLab Connection*.

![GitLabConnection](imagens/conn_added.png)

###### Gerenciamento de código fonte

Selecione a opção Git, e adicione as informações do repositório criado no GitLab.

![GitLabRepoConnection](imagens/git_repo_job.png)

Para adicionar as credencias válidas para o repositório, clique em [Add]->[Jenkins] e adicione um usuário e senha válidos para conexão com o repositório.

![GitLabRepoConnection](imagens/add_user.png)


###### Trigger de builds

Esse é o primeiro passo para nossa integração entre os dois serviços, estamos quase lá!

![applause](imagens/applause.gif)


Marque a opção *Build when a change is pushed to GitLab* e selecione quais ações executadas no GitLab vão acionar o Job no Jenkins. Copie a URL logo após a frase *GitLab CI Service URL*, pois vamos usa-la mais pra frente.

![trigger](imagens/trigger.png)

###### Build

Adicione um passo no build, com a opção *Executar shell* e adicione o código ```python mail_test.py```.
Com isso os testes que foram adicionados no repositórios irão ser executados pelo Jenkins.

![trigger](imagens/acao_build.png)

Clique em Salvar e o Job está finalizado!

###### Ações de Pós Build

Selecione a opção *Publish build status to GitLab commit*. Agora o Jenkins colocará o feedback de cada build nos commits/merges que acionaram o Job.

![trigger](imagens/pos.png)


### Webhook

Como último passo, vamos adicionar o Webhook em nosso repositório no GitLab. Ele vai fornecer as informações para o Jenkins quando houver alguma alteração ou acão no repositório no qual ele foi configurado.

Na página inicial do repositório, clique na engrenagem no canto superior direito e selecione *Webhooks*

Cole a URL que o Jenkins forneceu no campo URL, e substitua localhost por Jenkins, selecione as ações que vão acionar o webhook e clique em *Add Webhook*

![Hook](imagens/webhook.png)



### O Grand Finale

Após tantos passos e configurações, vamos ver nosso projeto rodando. Abra um merge request, ou faça um push para o repositório, e aguarde o Job ser iniciado, ao finalizar, ele adicionará no GitLab o feedback do build realizado com as alterações que foram feitas no código. Com isso seu projeto guanha muito mais agilidade e confiabilidade, com teste e feedbacks rápidos e automatizados.


![Merge](imagens/merge_ok.png)

Muito obrigado por lerem esse artigo, e espero que eu tenha conseguido ajudar de alguma forma no entendimento sobre esse tema.
Ficou alguma dúvida ou tem alguma sugestão? Utilize os comentários abaixo!


Até a próxima!
