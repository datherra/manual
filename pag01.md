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

````bash
$ rake db:create
```
(aqui pode ser bom falar um pouco sobre o rake, pra que serve, etc)

Rode o teste novamente, desta vez, vamos rodar através do *rake*, que dá no mesmo:

````bash
$ rake spec:requests
/Users/datherra/Devel/Ruby/Rails/apps/srmanager/db/schema.rb doesn't exist yet. Run `rake db:migrate` to create it then try again. If you do not intend to use a database, you should instead alter /Users/datherra/Devel/Ruby/Rails/apps/srmanager/config/application.rb to limit the frameworks that will be loaded
```

Opa, novo erro =(  
Mas a solução já vem indicada no próprio erro. (Conseguiu achar?)
Rode:

````bash
$ rake db:migrate
```

Rode o teste novamente:
````bash
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
Bem, se vamos "cadastrar uma SR", parece que vamos ter que criar este elemento em nossa aplicação.
