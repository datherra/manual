# Rails

## Nova Aplicação

Usaremos como **aplicação exemplo** um cadastro de SRs atendidas de cada sistema que também relacione o analista que a atendeu.

Crie a nova aplicação:

```bash
$ rails new srmanager -T -d mysql
```

Ajuste o seu ***Gemfile***:

```ruby
source "https://rubygems.org"

gem "rails", "3.2.6"
gem "mysql2"

group :assets do
  gem "sass-rails",   "~> 3.2.3"
  gem "therubyracer", :platforms => :ruby
  gem "uglifier", ">= 1.0.3"
end

gem "jquery-rails"

group :development, :test do
  gem "rspec-rails"
  gem "capybara"
  gem "thin"
  gem "rspec-rails"
end
```

SEMPRE que atualizar seu ***Gemfile***, execute:

```bash
$ bundle install
```

## Primeiros Passos com RSpec

Integre o rspec com o Rails:

```bash
$ rails generate rspec:install
```

Crie esqueleto do teste de integração:

```bash
$ rails generate integration_test cadastra_sr
```
 
Edite o arquivo criado pelo comando anterior, deixando-o com o seguinte conteúdo:

```ruby
# encoding: utf-8

require "spec_helper"

describe "Cadastra SRs" do
  context "quando enviando dados válidos" do
    
    before do
      visit "/"
      click_link "Cadastrar SR"
      
      fill_in "Número da SR", :with => "SR12001234"
      select "DSF", :from => "Sistema"
      select "Robert Plant", :from => "Analista"
      fill_in "Projeto", :with => "23o dígito"
      fill_in "Conclusão da SR", :with => "Tudo certo!"
      click_button "Adicionar SR"
    end
    
    it "redireciona para página de cadastro de SR"
    it "mostra mensagem de sucesso"
    it "mostra a SR cadastrada"
  end
  
  context "quando enviando dados inválidos" do
    
    before do
      visit "/"
      click_link "Cadastrar SR"
    end

    it "renderiza a página de cadastro de SR"
    it "mostra mensagem de erro"
  end    
end
```

Ainda não temos os testes implementados, mas neste arquivo já podemos verificar o quão simples é a DSL do RSpec. Ao ser executado, ele mostrará nossos testes com o status ***pending***:

```bash
$ rspec  spec/requests/cadastra_srs_spec.rb 

*****

Pending:
  Cadastra SRs quando enviando dados válidos redireciona para página de cadastro de SR
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:20
  Cadastra SRs quando enviando dados válidos mostra mensagem de sucesso
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:21
  Cadastra SRs quando enviando dados válidos mostra a SR cadastrada
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:22
  Cadastra SRs quando enviando dados inválidos renderiza a página de cadastro de SR
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:32
  Cadastra SRs quando enviando dados inválidos mostra mensagem de erro
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:33

Finished in 0.00056 seconds
5 examples, 0 failures, 5 pending
```

Agora, vamos começar a implementar as verificações, também conhecidas como *asserts*. Deixe o mesmo arquivo com o seguinte conteúdo:

```ruby
# encoding: utf-8

require "spec_helper"

describe "Cadastra SRs" do
  context "quando enviando dados válidos" do
    
    before do
      visit "/"
      click_link "Cadastrar SR"
      
      fill_in "Número da SR", :with => "SR12001234"
      select "DSF", :from => "Sistema"
      select "Robert Plant", :from => "Analista"
      fill_in "Projeto", :with => "23o dígito"
      fill_in "Conclusão da SR", :with => "Tudo certo!"
      click_button "Adicionar SR"
    end
    
    it "redireciona para página de cadastro de SR" do
      current_path.should be(sr_path)
    end
    it "mostra mensagem de sucesso" do
      page.should have_content("SR12001234 adicionada com sucesso!")
    end
    it "mostra a SR cadastrada" do
      current_path.should match(%r[/sr/\d+])
    end
  end
  
  context "quando enviando dados inválidos" do
    
    before do
      visit "/"
      click_link "Cadastrar SR"
    end

    it "renderiza a página de cadastro de SR"
    it "mostra mensagem de erro"
  end    
end
```

Execute o teste novamente e verifique a mensagem de erro:

```bash
$ rspec  spec/requests/cadastra_srs_spec.rb 
FFF**

Pending:
  Cadastra SRs quando enviando dados inválidos renderiza a página de cadastro de SR
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:38
  Cadastra SRs quando enviando dados inválidos mostra mensagem de erro
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:39

Failures:

  1) Cadastra SRs quando enviando dados válidos redireciona para página de cadastro de SR
     Failure/Error: Unable to find matching line from backtrace
     Mysql2::Error:
       Unknown database 'srmanager_test'
     # /Users/datherra/.rvm/gems/ruby-1.9.3-p194/gems/mysql2-0.3.11/lib/mysql2/client.rb:44:in `connect'
```

3 testes falharam. Como todos foram pelo mesmo motivo, removi algumas linhas do erro.
A linha `Mysql2::Error: Unknown database 'srmanager_test'` é bem autoexplicativa e esperada, já que ainda não criamos a base de dados desta aplicação.

Vamos criá-la?

## Criando a Base de Dados

Simples assim:

```bash
$ rake db:create
```
(aqui pode ser bom falar um pouco sobre o rake, pra que serve, etc)

Rode o teste novamente, desta vez, vamos rodar através do *rake*, que dá no mesmo:

```bash
$ rake spec:requests
/Users/datherra/Devel/Ruby/Rails/apps/srmanager/db/schema.rb doesn't exist yet. Run `rake db:migrate` to create it then try again. If you do not intend to use a database, you should instead alter /Users/datherra/Devel/Ruby/Rails/apps/srmanager/config/application.rb to limit the frameworks that will be loaded
```

Opa, novo erro =(  
Mas a solução já vem indicada no próprio erro. (Conseguiu achar?)  
Rode:

```bash
$ rake db:migrate
```

Rode o teste novamente:
```bash
$ rake spec:requests
/Users/datherra/.rvm/rubies/ruby-1.9.3-p194/bin/ruby -S rspec ./spec/requests/cadastra_srs_spec.rb
FFF**

Pending:
  Cadastra SRs quando enviando dados inválidos renderiza a página de cadastro de SR
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:38
  Cadastra SRs quando enviando dados inválidos mostra mensagem de erro
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:39

Failures:

  1) Cadastra SRs quando enviando dados válidos redireciona para página de cadastro de SR
     Failure/Error: click_link "Cadastrar SR"
     Capybara::ElementNotFound:
       no link with title, id or text 'Cadastrar SR' found
     # (eval):2:in `click_link'
     # ./spec/requests/cadastra_srs_spec.rb:10:in `block (3 levels) in <top (required)>'

  2) Cadastra SRs quando enviando dados válidos mostra mensagem de sucesso
     Failure/Error: click_link "Cadastrar SR"
     Capybara::ElementNotFound:
       no link with title, id or text 'Cadastrar SR' found
     # (eval):2:in `click_link'
     # ./spec/requests/cadastra_srs_spec.rb:10:in `block (3 levels) in <top (required)>'

  3) Cadastra SRs quando enviando dados válidos mostra a SR cadastrada
     Failure/Error: click_link "Cadastrar SR"
     Capybara::ElementNotFound:
       no link with title, id or text 'Cadastrar SR' found
     # (eval):2:in `click_link'
     # ./spec/requests/cadastra_srs_spec.rb:10:in `block (3 levels) in <top (required)>'

Finished in 0.3224 seconds
5 examples, 3 failures, 2 pending

Failed examples:

rspec ./spec/requests/cadastra_srs_spec.rb:20 # Cadastra SRs quando enviando dados válidos redireciona para página de cadastro de SR
rspec ./spec/requests/cadastra_srs_spec.rb:23 # Cadastra SRs quando enviando dados válidos mostra mensagem de sucesso
rspec ./spec/requests/cadastra_srs_spec.rb:26 # Cadastra SRs quando enviando dados válidos mostra a SR cadastrada
rake aborted!
/Users/datherra/.rvm/rubies/ruby-1.9.3-p194/bin/ruby -S rspec ./spec/requests/cadastra_srs_spec.rb failed

Tasks: TOP => spec:requests
(See full trace by running task with --trace)
```

Ok, agora o erro mudou.
Temos uma *exception* lançada pelo ***Capybara***, que é a lib usada pelo RSpec para simular a navegação do usuário em um site. A *exception* diz:

```ruby
Failure/Error: click_link "Cadastrar SR"
Capybara::ElementNotFound: no link with title, id or text 'Cadastrar SR' found
# (eval):2:in `click_link'
# ./spec/requests/cadastra_srs_spec.rb:10:in `block (3 levels) in <top (required)>'
```

Faz sentido, já que pedimos para o teste clicar no link "Cadastrar SR", mas ele ainda não existe.

Qual o próximo passo?  
Repare nestas linhas de nosso teste:

```ruby
  visit "/"
  click_link "Cadastrar SR"
```

Quando o usuário ***visita a raiz da aplicação***, o que é exibido no browser?  
Vamos verificar?  

## Server

Para subir a aplicação, basta executar o comando abaixo na raiz do projeto, neste caso, na pasta ***srmanager***:

```bash
$ rails server
```

Ou então, para economizar ponta de dedo:

```bash
$ rails s
=> Booting Thin
=> Rails 3.2.6 application starting in development on http://0.0.0.0:3000
=> Call with -d to detach
=> Ctrl-C to shutdown server
>> Thin web server (v1.3.1 codename Triple Espresso)
>> Maximum connections set to 1024
>> Listening on 0.0.0.0:3000, CTRL+C to stop
```

Detalhando o *output* do comando:

* `=> Booting Thin`  
***Thin*** é o *application server* que estamos usando, graças esta linha em nosso *Gemfile*:
```ruby
gem "thin"
```
O *application server* padrão do Rails é o ***WEBrick***, mas bisbilhotando e conversando com alguns desenvolvedores Rails mais experientes, fiquei sabendo que o Thin é mais rápido. Se no seu ambiente você usar o *WEBrick*, não tem problema, pois nada muda no código da aplicação. As diferenças poderiam começar a aparecer apenas no caso de você precisar ajustar alguma configuração muito específica deste *middleware*.

* `=> Rails 3.2.6 application starting in development on http://0.0.0.0:3000`  
Versão do Rails e a URL onde ele está ouvindo. Copie esta URL e cole no seu navegador.  

Esta página deverá aparecer:

![](./img02.png "index.html default")

Esta página é mostrada por padrão no Rails e ela fica em *srmanager/public/index.html*  
Bem, ela não tem o link "Cadastrar SR" como esperado pelo nosso teste, não é mesmo? Já que não vamos utilizá-la, apague o arquivo:

```bash
$ rm public/index.html
```

Sem mesmo reiniciar o servidor Rails, recarregue a página no seu browser. Isto é o que temos agora:

![](./img03.png "routing error")

Algumas dicas aparecem neste passo:

1. Repare que na janela que você deixou rodando seu *rails server* apareceram algumas mensagens. Lá é exibido o *output* do servidor, que muitas vezes possui mensagens úteis para um *troubleshooting*. Vale dizer que estas mesmas mensagens podem ser encontradas no arquivo `log/development.log`

2. O erro no browser acompanha uma sugestão:
Try running `rake routes` for more information on available routes.

Ok então. Execute:
```bash
$ rake routes
```
***Nada!?***  

Pois é, ainda não definimos nenhuma rota em nossa aplicação. Mas...  
...que diabos são ***ROTAS!?***


## Routes

Citando o [*Rails Routing from the Outside In*](http://guides.rubyonrails.org/routing.html) :  

> *The Rails router recognizes URLs and dispatches them to a controller’s action. It can also generate paths and URLs, avoiding the need to hardcode strings in your views.*  

Pois bem, se é ele quem decide para onde mandar o usuário quando os *requests* do browser chegam ao Rails, precisamos configurar o que fazer quando o usuário solicitar o "/" (barra).  

O arquivo *config/routes.rb* é criado junto com aplicação e possui vários exemplos comentados. Para simplificar, irei apagar todas as linhas comentadas e ir adicionando somente as úteis para a aplicação.  

Edite o arquivo *config/routes.rb* e deixe-o assim:
```ruby
Srmanager::Application.routes.draw do
  root :to => "pages#index"
end
```

Em aplicações web é comum possuirmos as páginas que lidam com conteúdo dinâmico, que são aquelas por onde se manipula informações que são extraídas e salvas de algum repositório de dados, normalmente um banco de dados.  

Mas há também as páginas estáticas, como uma *home page*, página institucional com informações da empresa, *about page* com informações básicas da própria aplicação, etc. Para estes casos, é muito comum na comunidade Rails o uso de um *controller* chamado *pages* ou *site* que servirá apenas para servir suas páginas estáticas.  

No trecho de código que colocamos no *routes.rb* estamos indicando que, toda vez que alguém visitar a raiz da aplicação, ela deverá ser redirecionada a *ACTION* ***index*** do *CONTROLLER* ***pages***.

Salve o *routes.rb* e verifique suas rotas novamente:

```bash
$ rake routes
root  / pages#index
```

Vimos que a rota relacionada ao "/" foi adicionada.  

Agora, rode o teste:

```bash
$ rspec spec/requests/cadastra_srs_spec.rb 
FFF**

Pending:
  Cadastra SRs quando enviando dados inválidos renderiza a página de cadastro de SR
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:38
  Cadastra SRs quando enviando dados inválidos mostra mensagem de erro
    # Not yet implemented
    # ./spec/requests/cadastra_srs_spec.rb:39

Failures:

  1) Cadastra SRs quando enviando dados válidos redireciona para página de cadastro de SR
     Failure/Error: visit "/"
     ActionController::RoutingError:
       uninitialized constant PagesController
     # ./spec/requests/cadastra_srs_spec.rb:9:in `block (3 levels) in <top (required)>'
```

Antes de analizarmos o erro, vamos mudar uma configuração do RSpec para que o *output* dele fique mais bacana. Edite o arquivo `.rspec` na raiz do projeto e atualize-o para este conteúdo:

```
--colour --format documentation
```

Rode o teste novamente e repare na diferença do *output*:

```bash
Cadastra SRs
  quando enviando dados válidos
    redireciona para página de cadastro de SR (FAILED - 1)
    mostra mensagem de sucesso (FAILED - 2)
    mostra a SR cadastrada (FAILED - 3)
  quando enviando dados inválidos
    renderiza a página de cadastro de SR (PENDING: Not yet implemented)
    mostra mensagem de erro (PENDING: Not yet implemented)
```

Este sumário do resultado dos testes é muito útil por já deixar claro quais funcionalidades falharam e também nos ajuda a construir descritivos coerentes para os testes, já que eles ao serem lidos desta forma, lhe direcionarão mais rapidamente ao problema (quando houver um).  

Voltando ao erro:

```ruby
Failure/Error: visit "/"
ActionController::RoutingError:
  uninitialized constant PagesController
# ./spec/requests/cadastra_srs_spec.rb:9:in `block (3 levels) in <top (required)>'
```

Bem, como instruímos o Rails a redirecionar os usuários que procuram a raiz da aplicação para *"pages#index"*, vamos ter que criar este *controller* citado no erro, o ***PagesController***.  

## Criando entidades da aplicação

Lembra deste desenho?

![mvc](./mvc.png "MVC")

A grande maioria das entidades que usarmos nas aplicações, passarão por estes 3 componentes: o ***Controller***, o ***Model*** e a ***View***.

Vamos começar criando o ***Controller*** que nosso teste está pedindo.  
Conforme a imagem a abaixo, crie o arquivo *pages_controller.rb* na pasta *app/controllers/* :

![](./img04.png)

Para criarmos um controller no Rails, apenas extenda a classe ***ApplicationController***:

```ruby
class PagesController < ApplicationController
  
end
```

Salve o arquivo e, adivinhe, rode o teste *again!*

```bash
$ rake spec:requests
```

Para economizar espaço, irei colar aqui apenas o trecho relevante do *output* do RSpec:

```ruby
Failure/Error: visit "/"
AbstractController::ActionNotFound:
  The action 'index' could not be found for PagesController
```

Bom, resolvemos o erro que reclamava do ***PagesController***.  
Agora precisamos resolver o próximo erro, o da *action* ***index***.  

Em Rails, *actions* são simples métodos dos *controllers*, portanto, no arquivo `pages_controller.rb`, dentro da classe ***PagesController*** defina o método ***index***:

```ruby
class PagesController < ApplicationController
  def index
    
  end
end
```

Teste.
```bash
$ rake spec:requests
```

Novo erro:

```ruby
Failure/Error: visit "/"
 ActionView::MissingTemplate:
   Missing template pages/index, application/index with {:locale=>[:en], :formats=>[:html], :handlers=>[:erb, :builder]}. Searched in:
     * "/Users/datherra/Devel/Ruby/Rails/apps/srmanager/app/views"
 # ./spec/requests/cadastra_srs_spec.rb:9:in `block (3 levels) in <top (required)>'
```

*ActionView::MissingTemplate*!  
Pois bem, já passamos do *controller* e chegamos agora no outro componente, a *view*.  

Vamos criá-la?  

## View

A *view* é o que utilizamos para solicitar e mostrar dados aos usuários. E ela que traz ou leva as informações devidamente tratadas em nossos controladores.  
Até aqui ainda não colocamos nenhuma inteligência no *controller*, mas isso irá mudar mais adiante.  

Vamos detalhar esta linha do erro:

```ruby
Missing template pages/index, application/index with {:locale=>[:en], :formats=>[:html], :handlers=>[:erb, :builder]}
```

* Missing template pages/index  
indica qual a página que está faltando, neste caso, http://servidor/pages/index
* :locale=>[:en]  
ajuda a definir qual template renderizar baseado no idioma detectado no navegador (fora do escopo deste tutorial)
* :formats=>[:html]  
qual formato está sendo solicitado e portanto o que deverá ser comtemplado pelo template. Neste caso mostra ":html" porque utilizamos o browser (o RSpec simula o *request* de um browser). Poderia ser XML ou JSON, por exemplo.
* :handlers=>[:erb, :builder]  
qual parser/handler/template engine (chame como preferir) que será usado para renderizar este template

Sendo mais prático, vamos corrigir o erro.  
Na pasta *app/view* crie a subpasta *pages* (repare no plural), e dentro dela o arquivo *index.html.erb*:

![](./img05.png "index.html view")

Deixe o arquivo *index.html* vazio mesmo.

***Teste*** (preciso colocar o comando aqui? Acho que não mais né?)  

Novo erro:

```ruby
Failure/Error: click_link "Cadastrar SR"
Capybara::ElementNotFound:
  no link with title, id or text 'Cadastrar SR' found
# (eval):2:in `click_link'
# ./spec/requests/cadastra_srs_spec.rb:10:in `block (3 levels) in <top (required)>'
```

Faz sentido não é?  
Recarregue a página no browser, você verá que ira aparecer apenas uma página vazia, sem erros, o que já é bom sinal.  

O teste está reclamando que não encontra o link "Cadastrar SR".  

Vamos criá-lo?

## ERB
