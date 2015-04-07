---
layout: post
title: "Do zero ao Deploy"
description: Preparando um ambiente com integração continua
headline:
category: DevOps
tags: [TravisCI, Heroku, Rails, Deploy, Github, Code Climate, CI]
imagefeature: from-zero-to-deploy/main.jpg
comments: true
mathjax:
---
Atualmente o mercado exige que sejamos o mais produtivos possível, independente da area de atuação, e quando falamos de tecnologia essa exigência aumenta ainda mais.
 Portanto, precisamos automatizar tarefas repititivas sempre que possível e neste post trago uma de muitas possíveis soluções para você manter um projeto com um
 pipeline que automatiza o seu deploy e mede a qualidade do seu código.

Nesse artigo iniciaremos um projeto do zero e o deixaremos configurado para o deploy. Utilizaremos o [Github](http://github.com) como nosso gerenciador de repositório,
 o [Ruby on Rails](http://rubyonrails.org) como web framework para o nosso projeto, o [Code Climate](https://codeclimate.com) para analise de qualidade e cobertura de testes,
 o [TravisCI](http://travis-ci.org) como nosso servidor de Integração Continua e o [Heroku](https://www.heroku.com) como provedor da nossa aplicação.

Você deve estar se perguntando, porque ele fez essas escolhas? Quando concluir explicarei ;)

> *Cubrir todo conteúdo git, rails, etc, foge do escopo deste post. Portanto, tenho em mente que você já tenha algum conhecimento ou busque-os com as referências apontadas na primeira vez em que cito os mesmos;*<br>

<section>
  <header>
    <h2>Os passos abaixo serão demonstrados usando um sistema unix.</h2>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section>

## Criando a aplicação Rails ##

O [Rails](http://rubyonrails.org) é um framework de código aberto para desenvolvimento de aplicações web feito sobre a linguagem Ruby, otimizado para a felicidade do programador \o/. Portanto, caso você não conheça, não perca mais tempo ;)

Execute o comando a seguir para instalar a gem do Rails

{% highlight bash %}
~ $ gem install rails
{% endhighlight %}

Esse comando instalará a versão atual do Rails. Em seguida, crie a aplicação de examplo **blog** e navegue para ela.

{% highlight bash %}
~ $ rails new blog
$ cd hello-rails
{% endhighlight %}

Pronto, já criamos nossa aplicação. Simples, não foi?
Para verificar se tudo ocorreu corretamente, vamos iniciar a nossa aplicação

{% highlight bash %}
$ rails server
{% endhighlight %}

Se tudo ocorreu bem, nossa aplicação poderá ser acessada através da url [http://localhost:3000](http://localhost:3000) e você verá uma página semelhante a essa imagem:

<img src="http://guides.rubyonrails.org/images/getting_started/rails_welcome.png" >

## Criando um repositório no GitHub ##
 O [GitHub](https://github.com/) é um provedor de repositórios [Git](http://git-scm.com) que permite a você gerenciar de maneira muit fácil seu projeto, oferecendo Wiki Pages, controle de issues, gráficos para analisar melhor as contribuições, etc. 

 O Git é um sistema de controle de versão distribuído de código aberto e gratuíto.
 Você pode instalá-lo, caso ainda não o tenha feito, manualmente baixando o [instalador](http://git-scm.com/downloads) ou atráves do [Homebrew](http://brew.sh/). Para este exemplo usaremos o Homebrew, portanto execute o comando a seguir:

{% highlight bash %}
~ $ brew update
~ $ brew install git
{% endhighlight %}

Se tudo ocorreu bem, esse comando mostrará a versão do git instalado:

{% highlight bash %}
~ $ git version
{% endhighlight %}

Com o Git instalado, vamos adicionar o controle de versão sobre o nosso projeto com o comando:

{% highlight bash %}
/blog $ git init
{% endhighlight %}

Pronto! Com esse único comando, nosso projeto está sobre controle de versão =). Para você conferir o estado do projeto, execute o comando:

{% highlight bash %}
/blog $ git status
{% endhighlight %}

Você verá uma lista com todas as alterações no seu projeto. Como acabamos de criá-lo, apenas faremos um commit da versão inicial do projeto.

{% highlight bash %}
/blog $ git add .
/blog $ git commit -m 'initial version'
{% endhighlight %}

Ok. Agora estamos pronto para criar nosso repositório no GitHub. Vá até o [GitHub](https://github.com/) e crie uma conta, caso não possua. Após isso crie um novo repositório ***Public*** com o nome ***blog***. Conforme a imagem abaixo:

![Criando Repositório Git]({{ site.url }}/images/from-zero-to-deploy/create_git_repository.gif)

Após isso o Github te dá uma serie de opções. Nós escolheremos a de usar um repositório existente, pois já temos um local. Então execute os comandos sugeridos por ele na pasta do projeto, substituindo **your_account** pela sua conta. Veja

{% highlight bash %}
# Adiciona uma referência chamada origin, de um repositório remoto para o nosso repositorio local
~ $ git remote add origin https://github.com/your_account/blog.git

# Envia o conteúdo do nosso branch local **master** para um novo branch **master** no repositório remoto linkando-os
~ $ git push -u origin master
{% endhighlight %}

Logo em seguida o git pedirá pra você se autenticar. Informe sua conta e senha. Após isso, visite o seu repositório novamente, você verá que o código que criamos se encontra lá em um branch **master**.

Com o repositório criado no Github, você pode adicionar novos colaboradores da sua equipe e até mesmo receber contribuições atráves de Pull Requests.

## Adicionando Analise de qualidade e cobertura de testes ##
É essencial à todo projeto que começa a evoluir e receber contribuições, poder analisar a qualidade do seu código e qual o percentual de cobertura dos seus testes.
 Para isso utilizaremos o [Code Climate](https://codeclimate.com) que oferece esses serviços gratuitamente para projetos open source nas linguagens Ruby, Javascript e PHP, além de integrações com diversos sistemas que ajudam no gerenciamento do seu projeto.

Adicionar o Code Climate é simples, vá até o site, faça o login com sua conta do Github e clique em **Add Open Source Repo**.
 Então informe o path ***your_account/blog*** e clique em **Import Repo from GitHub**. Após isso o Code Climate fará a analise de qualidade do repositório informado.
 Caso esteja demorando para exibir a resposta, atualize a página.

Ok.
 Após isso você pode ver que ele gerou uma métrica que vai de 0 a 4 para o seu projeto, onde 4 é o máximo, o que não quer dizer que o projeto é perfeito, mas pelo menos você está seguindo as práticas recomendadas.

Agora podemos configurar a nossa cobertura de teste. Ná página do nosso projeto **https://codeclimate.com/github/your_account/blog** e clique em ***Set Up Test Coverage***. Seguindo as instruções informadas precisamos:

* Adicionar a gem **codeclimate-test-reporter** ao nosso arquivo **Gemfile**:

{% highlight ruby %}
gem "codeclimate-test-reporter", group: :test, require: nil
{% endhighlight %}

* Adicionar na primeira linha do nosso *test_helper.rb* as linhas a seguir:

{% highlight ruby %}
require "codeclimate-test-reporter"
CodeClimate::TestReporter.start
{% endhighlight %}

* Execute o bundle install, pois adicionamos uma nova *gem* no nosso *Gemfile*

{% highlight bash %}
$ bundle install
{% endhighlight %}

* E quando formos executar nossos testes, devemos informar o token do Code Climate para o nosso projeto, para que ele atualize o status da cobertura de testes:

{% highlight bash %}
$ CODECLIMATE_REPO_TOKEN=f8c67efe3adebe899e094743fb370e1ca2d71e31db12773e3396a608f5b1ebbf bundle exec rake
{% endhighlight %}

Quando executarmos o comando acima, nenhum relatório de cobertura será gerado ainda, pois não temos nenhum teste no nosso
projeto. Mas fique tranquilo, faremos isso posteriormente.

Adicione ao arquivo **.gitignore** a seguinte linha:

{% highlight ruby %}
coverage/
{% endhighlight %}

Pois quando rodarmos os testes, ele gera um relatório nesse diretório e envia ao Code Climate, portanto vamos ignorá-lo.

Um recurso bacana, são os **badges** que o Code Climate fornece para o status do seu projeto. Ele torna fácil para qualquer pessoa visualizar quais as métricas
 do projeto, com link para uma melhor análise. Então vamos adicioná-lo.

 Acesse a url referente as métricas do seu projeto e adicione **/badges** ao final da url. Ex.: [https://codeclimate.com/github/your_account/blog/badges](https://codeclimate.com/github/your_account/blog/badges).
 Copie os badges para rdoc e cole-os no arquivo **README.rdoc** do projeto. Ex.

 {% highlight ruby %}
 # BLOG

 {<img src="https://codeclimate.com/github/ddomingues/blog/badges/gpa.svg" />}[https://codeclimate.com/github/ddomingues/blog]

 {<img src="https://codeclimate.com/github/ddomingues/blog/badges/coverage.svg" />}[https://codeclimate.com/github/ddomingues/blog]
 {% endhighlight %}

O Code Climate te dá a flexibilidade de escolher quais arquivos você quer analisar, para isso vá em ***Settings*** e
clique na aba **Analysis**, lá você pode informar um pattern para ignorar pastas ou arquivos ;)

Com isso configuramos a análise do nosso código. Faça um commit e push das alterações e bora pra próxima etapa.

## Colocando nossa aplicação no Heroku ##
Se você ainda não conhece o [Heroku](https://www.heroku.com/), ele é um [PaaS](http://en.wikipedia.org/wiki/Platform_as_a_service) que fornece
 uma plataforma completa para que você possa executar e gerenciar suas aplicações sem se preocupar com toda complexidade de infraestrutura.

O Heroku fornece muitas integrações para suas aplicações, nomeados como [Add-ons](https://addons.heroku.com/), que você pode adicionar à sua aplicação quase que transparentemente.

Mas chega de conversa, para mais detalhes, dê uma espiada na [documentação](https://devcenter.heroku.com/) do site, que por sinal é excelente ;)

Antes de criar nossa aplicação no Heroku, precisamos fazer alguns ajustes na configuração do nosso projeto. Por padrão, o Rails cria o projeto usando como modulo de acesso a dados o **SQLite3**,
 porém ele não é recomendado para produção, então precisamos de um banco de dados robusto. Como recomendado pelo Heroku, usaremos o **PostgreSQL**. 

Altere a linha a seguir do nosso **Gemfile**

{% highlight ruby %}
gem 'sqlite3'
{% endhighlight %}

 por

{% highlight ruby %}
gem 'sqlite3', group: [:development, :test]
gem 'pg', group: :production
{% endhighlight %}

Ok. Agora precisamos adicionar a gem que habilita o heroku para servir nossos assets estaticamente e integrar os logs.

{% highlight ruby %}
gem 'rails_12factor', group: :production
{% endhighlight %}

Vamos também apontar qual a versão do Ruby que queremos que o Heroku suba nossa arquitetura. Adicione no **Gemfile** após o `source 'https://rubygems.org'`, da seguinte maneira:

{% highlight ruby %}
source 'https://rubygems.org'
ruby '2.1.5'
{% endhighlight %}

Como alteramos nosso **Gemfile** é necessário que a gente execute o comando abaixo, para atualizar nossas dependências:

{% highlight bash %}
/blog $ bundle install
{% endhighlight %}

E comitar nossas alterações:

{% highlight bash %}
/blog $ git add .
/blog $ git commit -m 'Adding Heroku configuration'
/blog $ git push
{% endhighlight %}

Pronto. Agora vamos criar nossa aplicação no Heroku, para isso execute os seguintes passos:

* Crie uma conta.
* Baixe o [Heroku Toolbelt](https://toolbelt.heroku.com/) que permite que você tenha acesso ao Heroku via linha de comando.
* Autentique-se no heroku:
{% highlight bash %}
/blog $ heroku login
{% endhighlight %}
* Crie a aplicação
{% highlight bash %}
/blog $ heroku create
{% endhighlight %}
Ao rodar o comando acima, uma referência remota para um repositório no heroku é adicionado ao nosso projeto com o alias **heroku**.
 O Heroku também gera um nome aleatório para sua aplicação, veja no log algo parecido com isso: `Creating cryptic-eyrie-5775... done`, onde **cryptic-eyrie-5775** é o nome da aplicação que o heroku gerou.
 No entanto, você pode personalizá-lo, renomeando-a caso o nome desejado esteja disponível, com o comando a seguir.

{% highlight bash %}
/blog $ heroku apps:rename newname
{% endhighlight %}

* Suba nosso projeto
{% highlight bash %}
/blog $ git push heroku master
{% endhighlight %}
Você verá todo o log do processo de build da aplicação no heroku.

Caso finalize com sucesso, ao rodar o comando abaixo você verá a aplicação rodando no Heroku.

{% highlight bash %}
/blog $ heroku open
{% endhighlight %}

Como não implementamos nada na aplicação, ele não encontrará a página index. Faremos isso em uma próxima etapa.

Já viu algo mais simples? =)

Ainda não acabou.

## Adicionando o Servidor de Integração Continua Travis CI ##
O [Travis CI](https://travis-ci.com) é um serviço que possibilita a build do seu projeto, execução de testes e facilita o deploy de uma maneira
 muito simples e flexível, permitindo que você foque no que é mais importante, seu código.

Configurar o travis no seu projeto é muito simples. Vá ao [site](https://travis-ci.org) e crie uma conta usando o seu usuário do GitHub. Ele listará todos os repositórios públicos da sua conta, então selecione o repositório do nosso projeto. Adicione um arquivo **.travis.yml** na raiz do projeto. Após isso o Travis estará observando seu repositório, e no próximo push que o repositório receber, ele fará o build do projeto.

Ok. Agora vamos customizar o nosso build. O Travis é bem flexível e nos permite personalizar praticamente todas as etapas do Build, por exemplo: *antes* ou *após* o build, faça algo. Portanto vamos customizar nossa configuração.

A idéia é simples, após o Travis executar nossos testes, caso todos os testes passem, eu quero que ele faça *deploy* no Heroku \o/

Para isso, vamos a configuração. Primeiro vamos informar qual a linguagem no projeto e qual a versão do Ruby queremos que o travis utilize. Adicione ao arquivo **.travis.yml** as seguintes intruções:

{% highlight ruby %}
language: ruby
rvm:
- 2.1.5
{% endhighlight %}

Após isso vamos adicionar um e-mail para que o Travis nos notifique ao final do build qual o status, sucesso ou falha.

{% highlight ruby %}
notifications:
  email:
  - diego.domingues16@gmail.com
{% endhighlight %}

O Travis já tem integração com vários serviços, e um deles é Code Climate. Como já fizemos a configuração do nosso projeto, basta adicionarmos as instruções abaixo, substituindo pelo token do nosso projeto:

{% highlight ruby %}
addons:
  code_climate:
    repo_token: adf08323....
{% endhighlight %}

Ok. Agora o mais legal, após o nosso build ser feito com sucesso queremos que ele faça o deploy. Para essa integração basta informarmos o token do Heroku para nossa aplicação. Porém, por motivo de segurança não devemos expor o token da nossa aplicação. Para solucionar isso, o Travis fornece um client que encripta nosso token de maneira muito simples. Para instalá-lo rode o seguinte comando:

{% highlight bash %}
~ $ gem install travis
{% endhighlight %}

Com o travis instalado vamos encriptar nosso token. Na raiz do nosso projeto execute:

{% highlight bash %}
~ $ travis encrypt $(heroku auth:token) --add deploy.api_key
{% endhighlight %}

Esse comando encriptará nosso token e já adicionará as intruções a nossa configuração. Algo similar a isso:

{% highlight ruby %}
deploy:
  provider: heroku
  api_key: ...
{% endhighlight %}

O Travis, por padrão, tentará fazer deploy em uma aplicação com o mesmo nome do repositório, para personalizar você pode informar o nome que você deu para sua aplicação, por exemplo **personal-blog**. A configuração ficaria assim:

{% highlight ruby %}
deploy:
  provider: heroku
  api_key: ...
  app: personal-blog
  run: "rake db:migrate"
{% endhighlight %}

> Um ponto importante, é que nossa base de dados evolui constantemente então, precisamos adicionar o comando para executar nossas migrações.

Por fim, vamos adicionar a badge do status do nosso build ao **README.rdoc**.

{% highlight ruby %}
{<img src="https://travis-ci.org/ddomingues/blog.svg?branch=master" alt="Build Status" />}[https://travis-ci.org/your_account/blog]
{% endhighlight %}

É isso. Faça um commit com nossas alterações e quando o repositório receber o push, o Travis fará o build, e caso tenha sucesso, enviará nosso build para o heroku.

Sensacional, não é? =)

## Evoluindo nossa aplicação ##
Agora para simular um cenário real de desenvolvimento, vamos gerar um [CRUD](http://en.wikipedia.org/wiki/Create,_read,_update_and_delete) de exemplo. Execute o seguinte comando:

{% highlight ruby %}
rails generate scaffold article title description:text
{% endhighlight %}

O comando [scaffold](http://guides.rubyonrails.org/command_line.html#rails-generate) criará um recurso do modelo **article**, incluindo todas ações CRUD com testes, views, helpers, migration, etc.

Como geramos um novo modelo, precisamos executar a migration criada para evoluir nosso base de dados, portanto execute:

{% highlight bash %}
/blog $ bundle exec rake db:migrate
{% endhighlight %}

Ok. Para ver em ação, acesse a url [http://localhost:3000/articles](http://localhost:3000/articles).

Você pode ver os testes gerados na pasta ***test***. Por padrão o Rails usa o [minitest](https://github.com/seattlerb/minitest), para mais detalhes veja sua ótima [documentação](http://guides.rubyonrails.org/testing.html).

Para rodar os testes, execute o comando:

{% highlight bash %}
/blog $ bundle exec rake test
{% endhighlight %}

Se você seguiu todos os passos corretamente, os testes estarão passando. E verá algo parecido com isso:

{% highlight bash %}
# Running:

.......

Finished in 1.157917s, 6.0453 runs/s, 11.2271 assertions/s.

7 runs, 13 assertions, 0 failures, 0 errors, 0 skips
{% endhighlight %}

Agora vamos apontar o root da nossa aplicação para a ***index*** dos nossos artigos. Altere o arquivo **routes.rb** para:

{% highlight ruby %}
Rails.application.routes.draw do
  resources :articles

  root 'articles#index'
end
{% endhighlight %}

Agora ao acessarmos a url root da nossa aplicação, veremos a página **index** do recurso **Article**. Faça um commit e push das alterações.
Você verá que o Travis fará um novo build e ao final, fará um deploy da aplicação com nossas alterações.

Vá a url da sua aplicação no Heroku, e você verá que sua aplicação está sincronizada com seu branch master. Legal neh =)

Faça algumas alterações, quebre o build, corrija. Explore recursos não citados aqui!

Esse é um fluxo básico, poderiamos trabalhar também com um ambiente de [staging](https://devcenter.heroku.com/articles/deploying-to-a-custom-rails-environment#setting-staging-behavior), utilizar a técnica de [Feature Branch](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow) com [pull-requests](https://help.github.com/articles/using-pull-requests/), [code-review](http://en.wikipedia.org/wiki/Code_review),
mas isso fica como lição de casa, seja curioso! ;)

Caso você queira ver o código final dessa aplicação de exemplo, veja o meu [repositório](https://github.com/ddomingues/blog)

# Conclusão

Quase esqueci de explicar o porque escolhi essas tecnologias. Escolhi, porque eu quero, o blog é meu =P ... Zueira...

Bom acho que se você chegou até já aqui deve ter observado o porquê. Escolhi pela simplicidade, facilidade, qualidade e flexibilidade que todas essas tecnologias oferecem.

O Rails é um web framework fantástico e completo, além de ser feito em Ruby =). Quando você descobre se apaixona.

O Git dispensa comentários, usado com o GitHub, fica melhor ainda. Não usei quase nada dele no post, mas recomendo muito que você se aprofunde em conhecê-lo melhor, é fantástico.

O Code Climate é essencial, auxiliando a manter a qualidade do seu projeto, facilitando o refactoring, apontanto código duplicado, entre outros. Caso, você ache o
 serviço caro, você pode utilizar alternativas que, inclusive, o próprio Code Climate utiliza, como [SimpleCov](https://www.ruby-toolbox.com/projects/simplecov), [Rubocop](https://www.ruby-toolbox.com/projects/rubocop), [etc](https://www.ruby-toolbox.com/categories/code_metrics).

Heroku é o PaaS mais fácil que conheço para gerenciar e fazer o deploy de uma aplicação, permitindo à você focar no mais importante, seu negócio/código. Há tantas alternativas que nem vou citar todas, mas entre elas uma opção bacana também é a [Digital Ocean](https://www.digitalocean.com/).

Quanto a Servidor de Integração Continua, optei pelo Travis pela flexibilidade, simplicidade e qualidade, mas vale a pena dar uma olhada no [Snap CI](https://www.snap-ci.com/) e no [Wercker](http://wercker.com/)

O mais importante desse post, é notar que ao eliminar tarefas repititivas do nosso fluxo de trabalho, aumentamos nossa produtividade, e evitamos possíveis erros que poderiam vir a ocorrer. Afinal, somos humanos, e humanos falham ;)

Apenas apresentei aqui o **básico** dessas tecnologias bacanas, é essencial que você se aprofunde para um melhor aproveitamento delas.

É isso aí galera. Espero que tenham gostado, desse meu primeiro post. Deixe ai seu comentário, e me de seu feedback! =)