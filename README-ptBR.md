# Introdução

> Exemplos são importantes. <br/>
> -- Oficial Alex J. Murphy / RoboCop

O objetivo desse guia é apresentar um conjunto de boas práticas e orientações de estilo para o desenvolvimento em Ruby on Rails 4. Esse guia é complementar ao já existente [guia de estilo de programação em Ruby](https://github.com/bbatsov/ruby-style-guide), criado pela comunidade.

Alguns dos conselhos apresentados aqui só se aplicam ao Rails 4.0+.

Você pode gerar uma cópia desse guia em PDF ou HTML usando o [Transmuter](https://github.com/TechnoGate/transmuter).

Traduções desse guia estão disponíveis nos seguintes idiomas:

* [Chinês simplificado](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [Chinês tradicional](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [Alemão](https://github.com/arbox/de-rails-style-guide/blob/master/README-deDE.md)
* [Japonês](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)
* [Russo](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)
* [Turco](https://github.com/tolgaavci/rails-style-guide/blob/master/README-trTR.md)
* [Coreano](https://github.com/pureugong/rails-style-guide/blob/master/README-koKR.md)
* Português brasileiro

# O Guia de Estilo do Rails

Esse guia de estilo do Rails recomenda boas práticas para que programadores Rails reais escrevam código que possa ser mantido por outros programadores Rails reais. Se um guia de estilo reflete o uso real, ele é usado; porém, um guia de estilo que se apega a um ideal que foi rejeitado pelas pessoas que ele deveria ajudar fica sob risco de cair em desuso &ndash; não importa o quão bom ele seja.

Esse guia está dividido em várias seções de regras relacionadas. Eu tentei incluir o motivo por trás das regras (se ele foi omitido, é porque eu assumi que é bastante óbvio).

Eu não inventei essas regras do nada - elas são baseadas principalmente na minha extensa carreira como engenheiro de software profissional, feedback e sugestões de membros da comunidade Rails e vários outros recursos conceituados de programação Rails. 

## Sumário

* [Configuração](#configuração)
* [Roteamento](#roteamento)
* [Controllers](#controllers)
* [Models](#models)
  * [ActiveRecord](#activerecord)
  * [Consultas com o ActiveRecord](#consultas-com-o-activerecord)
* [Migrações](#migrações)
* [Views](#views)
* [Internacionalização](#internacionalização)
* [Assets](#assets)
* [Mailers](#mailers)
* [Hora](#hora)
* [Bundler](#bundler)
* [Gems problemáticas](#gems-problemáticas)
* [Gerenciando processos](#gerenciando-processos)

## Configuração

* <a name="config-initializers"></a>
  Coloque suas rotinas de inicialização em `config/initializers`. Os arquivos dessa pasta são executados quando a aplicação é iniciada.
<sup>[[link](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
  Deixe o código de inicialização de cada gem num arquivo separado com o mesmo nome da gem, por exemplo, `carrierwave.rb`, `active_admin.rb`, etc.
<sup>[[link](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>
  Ajuste adequadamente as configurações para os ambientes de desenvolvimento, teste e produção (nos respectivos arquivos em `config/environments/`).
<sup>[[link](#dev-test-prod-configs)]</sup>

  * Marque assets adicionais para serem pré-compilados (se houver):

    ```Ruby
    # config/environments/production.rb
    # Precompile additional assets (application.js, application.css,
    #and all non-JS/CSS are already added)
    config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
    ```

* <a name="app-config"></a>
  Deixe as configurações que se aplicam a todos os ambientes no arquivo `config/application.rb`.
<sup>[[link](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  Crie um ambiente adicional chamado `staging` que seja muito parecido com o ambiente `production`.
<sup>[[link](#staging-like-prod)]</sup>

* <a name="yaml-config"></a>
  Deixe quaisquer configurações adicionais em arquivos YAML na pasta `config/`.
<sup>[[link](#yaml-config)]</sup>

  Desde o Rails 4.2, é muito simples carregar arquivos de configuração YAML com o novo método `config_for`:

  ```Ruby
  Rails::Application.config_for(:yaml_file)
  ```

## Roteamento

* <a name="member-collection-routes"></a>
  Quando você precisar adicionar mais ações a um recurso RESTful (você precisa mesmo delas?) use as rotas `member` e `collection`.
<sup>[[link](#member-collection-routes)]</sup>

  ```Ruby
  # ruim
  get 'subscriptions/:id/unsubscribe'
  resources :subscriptions

  # bom
  resources :subscriptions do
    get 'unsubscribe', on: :member
  end

  # ruim
  get 'photos/search'
  resources :photos

  # bom
  resources :photos do
    get 'search', on: :collection
  end
  ```

* <a name="many-member-collection-routes"></a>
  Se você quiser definir múltiplas rotas `member/collection`, use a sintaxe alternativa que recebe um bloco.
<sup>[[link](#many-member-collection-routes)]</sup>

  ```Ruby
  resources :subscriptions do
    member do
      get 'unsubscribe'
      # mais rotas
    end
  end

  resources :photos do
    collection do
      get 'search'
      # mais rotas
    end
  end
  ```

* <a name="nested-routes"></a>
  Use rotas aninhadas para expressar melhor o relacionamento entre models do ActiveRecord.
<sup>[[link](#nested-routes)]</sup>

  ```Ruby
  class Post < ActiveRecord::Base
    has_many :comments
  end

  class Comments < ActiveRecord::Base
    belongs_to :post
  end

  # routes.rb
  resources :posts do
    resources :comments
  end
  ```
  
* <a name="namespaced-routes"></a>
  Se você precisar aninhar rotas com mais de 1 nível de profundidade, use a opção `shallow: true`. Isso vai poupar seu usuário de urls longas `posts/1/comments/5/versions/7/edit` e também vai te poupar de helpers longos `edit_post_comment_version`.
  
  ```Ruby
  resources :posts, shallow: true do
    resources :comments do
      resources :versions
    end
  end
  ```

* <a name="namespaced-routes"></a>
  Defina rotas dentro de um `namespace` para agrupar ações relacionadas.
<sup>[[link](#namespaced-routes)]</sup>

  ```Ruby
  namespace :admin do
    # Direciona /admin/products/* para Admin::ProductsController
    # (app/controllers/admin/products_controller.rb)
    resources :products
  end
  ```

* <a name="no-wild-routes"></a>
  Nunca use a antiga rota 'wild'. Essa rota vai deixar todas as ações em todos os controllers acessíveis via requisições GET.
<sup>[[link](#no-wild-routes)]</sup>

  ```Ruby
  # muito ruim
  match ':controller(/:action(/:id(.:format)))'
  ```

* <a name="no-match-routes"></a>
  Não use `match` para definir nenhuma rota, a não ser que você precise mapear múltiplos métodos de requisição dentre `[:get, :post, :patch, :put, :delete]` para uma mesma ação através da opção `:via`.
<sup>[[link](#no-match-routes)]</sup>

## Controllers

* <a name="skinny-controllers"></a>
  Mantenha os controllers enxutos - eles só devem obter dados para a camada de view e não devem conter nenhuma lógica de negócio (toda a lógica de negócio deve ficar naturalmente no model).
<sup>[[link](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  Cada ação de um controller deve (idealmente) invocar apenas um método além de um `find` ou `new` inicial.
<sup>[[link](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  Não compartilhe mais que duas variáveis de instância entre um controller e uma view.
<sup>[[link](#shared-instance-variables)]</sup>

## Models

* <a name="model-classes"></a>
  Introduza classes não-ActiveRecord livremente.
<sup>[[link](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
  Dê nomes significativos (porém curtos) para os models, sem abreviações.
<sup>[[link](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
  Se você precisar de objetos do model que suportem comportamento do ActiveRecord, mas sem a funcionalidade de banco de dados, use a gem [ActiveAttr(https://github.com/cgriego/active_attr)].
<sup>[[link](#activeattr-gem)]</sup>

  ```Ruby
  class Message
    include ActiveAttr::Model

    attribute :name
    attribute :email
    attribute :content
    attribute :priority

    attr_accessible :name, :email, :content

    validates :name, presence: true
    validates :email, format: { with: /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i }
    validates :content, length: { maximum: 500 }
  end
  ```

  Para um exemplo mais completo, veja o
  [RailsCast sobre isso](http://railscasts.com/episodes/326-activeattr).

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  Evite alterar os padrões do ActiveRecord (nomes de tabelas, chave primária, etc) a não ser que você tenha uma razão muito boa (como um banco de dados que você não controla).
<sup>[[link](#keep-ar-defaults)]</sup>

  ```Ruby
  # ruim - não faça isso se você puder modificar o schema
  class Transaction < ActiveRecord::Base
    self.table_name = 'order'
    ...
  end
  ```

* <a name="macro-style-methods"></a>
  Agrupe métodos no estilo macro (`has_many`, `validates`, etc) no topo da definição da classe.
<sup>[[link](#macro-style-methods)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    # deixe a default_scope primeiro (se houver)
    default_scope { where(active: true) }

    # depois vêm as constantes
    COLORS = %w(red green blue)

    # depois nós colocamos macros relacionadas a attr
    attr_accessor :formatted_date_of_birth

    attr_accessible :login, :first_name, :last_name, :email, :password

    # seguidas de macros de associações
    belongs_to :country

    has_many :authentications, dependent: :destroy

    # e macros de validação
    validates :email, presence: true
    validates :username, presence: true
    validates :username, uniqueness: { case_sensitive: false }
    validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
    validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true}

    # depois nós colocamos os callbacks
    before_save :cook
    before_save :update_username_lower

    # outras macros (como as do devise) devem ser colocadas depois dos callbacks

    ...
  end
  ```

* <a name="has-many-through"></a>
  Prefira `has_many :through` a `has_and_belongs_to_many`. Usando `has_many :through` você pode adicionar atributos e validações adicionais no modelo de junção.
<sup>[[link](#has-many-through)]</sup>

  ```Ruby
  # não muito bom - usando has_and_belongs_to_many
  class User < ActiveRecord::Base
    has_and_belongs_to_many :groups
  end

  class Group < ActiveRecord::Base
    has_and_belongs_to_many :users
  end

  # jeito melhor - usando has_many :through
  class User < ActiveRecord::Base
    has_many :memberships
    has_many :groups, through: :memberships
  end

  class Membership < ActiveRecord::Base
    belongs_to :user
    belongs_to :group
  end

  class Group < ActiveRecord::Base
    has_many :memberships
    has_many :users, through: :memberships
  end
  ```

* <a name="read-attribute"></a>
  Prefira `self[:attribute]` a `read_attribute(:attribute)`.
<sup>[[link](#read-attribute)]</sup>

  ```Ruby
  # ruim
  def amount
    read_attribute(:amount) * 100
  end

  # bom
  def amount
    self[:amount] * 100
  end
  ```

* <a name="write-attribute"></a>
  Prefira `self[:attribute] = value` a `write_attribute(:attribute, value)`.
<sup>[[link](#write-attribute)]</sup>

  ```Ruby
  # ruim
  def amount
    write_attribute(:amount, 100)
  end

  # bom
  def amount
    self[:amount] = 100
  end
  ```

* <a name="sexy-validations"></a>
  Sempre use as novas [validações "sexy"](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
<sup>[[link](#sexy-validations)]</sup>

  ```Ruby
  # ruim
  validates_presence_of :email
  validates_length_of :email, maximum: 100

  # bom
  validates :email, presence: true, length: { maximum: 100 }
  ```

* <a name="custom-validator-file"></a>
  Quando uma validação personalizada é usada mais que uma vez, ou a validação é um mapeamento de expressões regulares, crie uma nova classe de validação.
<sup>[[link](#custom-validator-file)]</sup>

  ```Ruby
  # ruim
  class Person
    validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
  end

  # bom
  class EmailValidator < ActiveModel::EachValidator
    def validate_each(record, attribute, value)
      record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
    end
  end

  class Person
    validates :email, email: true
  end
  ```

* <a name="app-validators"></a>
  Deixe os seus validadores em `app/validators/`.
<sup>[[link](#app-validators)]</sup>

* <a name="custom-validators-gem"></a>
  Considere extrair seus validadores próprios para uma gem compartilhada se você estiver mantendo vários apps relacionados ou se os validadores forem genéricos o suficiente.
<sup>[[link](#custom-validators-gem)]</sup>

* <a name="named-scopes"></a>
  Use escopos nomeados livremente.
<sup>[[link](#named-scopes)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    scope :active, -> { where(active: true) }
    scope :inactive, -> { where(active: false) }

    scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
  end
  ```

* <a name="named-scope-class"></a>
  Quando um escopo nomeado definido com um lambda e parâmetros ficar muito complicado, é preferível fazer ao invés dele um método de classe que sirva para o mesmo propósito e retorne uma instância de `ActiveRecord::Relation`. Você pode até definir escopos mais simples desse jeito.

<sup>[[link](#named-scope-class)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    def self.with_orders
      joins(:orders).select('distinct(users.id)')
    end
  end
  ```

  Perceba que esse estilo de escopo não pode ser colocado em chain do mesmo jeito que escopos nomeados. Por exemplo:

  ```Ruby
  # não dá pra fazer chain
  class User < ActiveRecord::Base
    def User.old
      where('age > ?', 80)
    end

    def User.heavy
      where('weight > ?', 200)
    end
  end
  ```

  Nesse estilo, tanto `old` como `heavy` funcionam separadamente, mas você não pode chamar `User.old.heavy`; para fazer chain nesses escopos, use:

  ```Ruby
  # dá pra fazer chain
  class User < ActiveRecord::Base
    scope :old, -> { where('age > 60') }
    scope :heavy, -> { where('weight > 200') }
  end
  ```


* <a name="beware-update-attribute"></a>
  Tome cuidado com o comportamento do método [`update_attribute`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute). Ele não executa as validações do model (diferente de `update_attributes`) e pode facilmente corromper o estado do model.
<sup>[[link](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
  Use URLs amigáveis. Mostre algum atributo descritivo do model na URL ao invés do `id` dele. Há várias formas de conseguir isso:
<sup>[[link](#user-friendly-urls)]</sup>
  
  * Sobrescrever o método `to_param` do model. Esse método é usado pelo Rails para construir a URL do objeto. A implementação padrão retorna o `id` do registro como String. Ela poderia ser sobrescrita para incluir outro atributo mais legível. 

      ```Ruby
      class Person
        def to_param
          "#{id} #{name}".parameterize
        end
      end
      ```

  Para deixar esse valor num formato compatível com URLs, você deve chamar `parameterize` na string. O `id` do objeto precisa estar no começo para que ele possa ser encontrado pelo método `find` do ActiveRecord.

  * Usar a gem `friendly_id`. Ela permite a criação de URLs legíveis através do uso de algum atributo descritivo do model ao invés do seu `id`.

      ```Ruby
      class Person
        extend FriendlyId
        friendly_id :name, use: :slugged
      end
      ```

  Veja a [documentação da gem](https://github.com/norman/friendly_id) para mais informações sobre como usá-la.

* <a name="find-each"></a>
  Use `find_each` para iterar sobre uma coleção de objetos AR. Iterar sobre uma coleção de registros do banco de dados (usando o método `all`, por exemplo) é muito ineficiente, porque o AR tentará instanciar todos os objetos de uma vez. Nesse caso, métodos de processamento em batch permitem que você trabalhe com os registros em batches, dessa forma reduzindo muito o consumo de memória.
<sup>[[link](#find-each)]</sup>


  ```Ruby
  # ruim
  Person.all.each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').each do |person|
    person.party_all_night!
  end

  # bom
  Person.find_each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').find_each do |person|
    person.party_all_night!
  end
  ```

* <a name="before_destroy"></a>
  Como o [Rails cria callbacks para associações dependentes](https://github.com/rails/rails/issues/3458), sempre chame callbacks `before_destroy` que realizem validações com `prepend: true`.
<sup>[[link](#before_destroy)]</sup>

  ```Ruby
  # ruim (roles vão ser deletados automaticamente mesmo que super_admin? seja true)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # bom
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```

### Queries do ActiveRecord

* <a name="avoid-interpolation"></a>
  Evite usar interpolação de strings em queries, porque isso deixa seu código suscetível a SQL injection.
<sup>[[link](#avoid-interpolation)]</sup>

  ```Ruby
  # ruim - param será interpolado sem ser escapado
  Client.where("orders_count = #{params[:orders]}")

  # bom - param será escapado adequadamente
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Considere usar placeholders nomeados ao invés de placeholders posicionais quando você tiver mais de 1 placeholder na sua query.
<sup>[[link](#named-placeholder)]</sup>

  ```Ruby
  # mais ou menos
  Client.where(
    'created_at >= ? AND created_at <= ?',
    params[:start_date], params[:end_date]
  )

  # bom
  Client.where(
    'created_at >= :start_date AND created_at <= :end_date',
    start_date: params[:start_date], end_date: params[:end_date]
  )
  ```

* <a name="find"></a>
  Prefira usar `find` ao invés de `where` quando você precisar obter um único registro pelo seu id.
<sup>[[link](#find)]</sup>

  ```Ruby
  # ruim
  User.where(id: id).take

  # bom
  User.find(id)
  ```

* <a name="find_by"></a>
  Prefira usar `find_by` ao invés de `where` quando você precisar obter um único registro por alguns atributos.
<sup>[[link](#find_by)]</sup>

  ```Ruby
  # ruim
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # bom
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```

* <a name="find_each"></a>
  Use `find_each` quando você precisar processar muitos registros.
<sup>[[link](#find_each)]</sup>

  ```Ruby
  # ruim - carrega todos os registros de uma vez
  # Isso é muito ineficiente caso a tabela de users tenha milhares de linhas.
  User.all.each do |user|
    NewsMailer.weekly(user).deliver_now
  end

  # bom - registros são obtidos em batches
  User.find_each do |user|
    NewsMailer.weekly(user).deliver_now
  end
  ```

* <a name="where-not"></a>
  Prefira usar `where.not` ao invés de SQL.
<sup>[[link](#where-not)]</sup>

  ```Ruby
  # ruim
  User.where("id != ?", id)

  # bom
  User.where.not(id: id)
  ```
* <a name="squished-heredocs"></a>
  Quando especificar uma query explícita num método como `find_by_sql`, use heredocs com `squish`. Isso permite que você formate o SQL de forma legível com quebras de linha e identações, ao mesmo tempo em que suporta o realce de sintaxe em diversas ferramentas (inclusive o GitHub, Atom e RubyMine).
<sup>[[link](#squished-heredocs)]</sup>

  ```Ruby
  User.find_by_sql(<<SQL.squish)
    SELECT
      users.id, accounts.plan
    FROM
      users
    INNER JOIN
      accounts
    ON
      accounts.user_id = users.id
    # outras complexidades...
  SQL
  ```

  O método [`String#squish`](http://apidock.com/rails/String/squish) remove a identação e quebras de linha; dessa forma o log do seu servidor vai mostrar uma string fluida de SQL ao invés de algo assim:

  ```
  SELECT\n    users.id, accounts.plan\n  FROM\n    users\n  INNER JOIN\n    acounts\n  ON\n    accounts.user_id = users.id
  ```

## Migrações

* <a name="schema-version"></a>
  Mantenha o arquivo `schema.rb` (ou `structure.sql`) sob controle de versão.
<sup>[[link](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Use `rake db:schema:load` ao invés de `rake db:migrate` para inicializar um banco de dados vazio.
<sup>[[link](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Imponha valores default nas próprias migrações ao invés de fazê-lo na camada de aplicação.
<sup>[[link](#default-migration-values)]</sup>

  ```Ruby
  # ruim - valor default definido na aplicação
  def amount
    self[:amount] or 0
  end
  ```

  Apesar de muitos desenvolvedores sugerirem a definição de padrões de tabelas apenas no Rails, isso é uma abordagem extremamente frágil, que deixa seus dados vulneráveis a diversos bugs da aplicação. E há de se considerar o fato de que a maioria dos apps não-triviais compartilha um banco de dados com outras aplicações, então impor a integridade dos dados a partir do app Rails é impossível.

* <a name="foreign-key-constraints"></a>
  Imponha restrições de chave estrangeira. A partir da versão 4.2, o ActiveRecord suporta restrições de chave estrangeira nativamente.
  <sup>[[link](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  When writing constructive migrations (adding tables or columns),
  use the `change` method instead of `up` and `down` methods.
  Quando estiver escrevendo migrações construtivas (acrescentando tabelas ou colunas), use o método `change` ao invés dos métodos `up` e `down`.
  <sup>[[link](#change-vs-up-down)]</sup>

  ```Ruby
  # o jeito antigo
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # o novo jeito preferencial
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
  Não use classes do model em migrações. As classes do model estão constantemente evoluindo e em algum ponto do futuro as migrações que funcionavam podem parar de funcionar, por causa de mudanças nos models usados.
<sup>[[link](#no-model-class-migrations)]</sup>

## Views

* <a name="no-direct-model-view"></a>
  Nunca invoque a camada do model diretamente de uma view.
<sup>[[link](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Nunca faça formatações complexas nas views; exporte a formatação para um método no helper da view ou no model.
<sup>[[link](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Reduza a duplicação de código através do uso de templates e layouts parciais.
<sup>[[link](#partials)]</sup>

## Internacionalização

* <a name="locale-texts"></a>
  Nenhuma string ou outras configurações específicas da região devem ser usadas nas views, models e controllers. Esses textos devem ser movidos para os arquivos regionais na pasta `config/locales`.
<sup>[[link](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  Quando os labels de um model do ActiveRecord precisarem ser traduzidos, use o escopo `activerecord`:
<sup>[[link](#translated-labels)]</sup>

  ```
  en:
    activerecord:
      models:
        user: Member
      attributes:
        user:
          name: 'Full name'
  ```

  Dessa forma, `User.model_name.human` vai retornar "Member" e `User.human_attribute_name("name")` vai retornar "Full name". Essas traduções dos atributos serão utilizadas como labels nas views.

* <a name="organize-locale-files"></a>
  Separe os textos usados nas views das traduções de atributos do ActiveRecord. Coloque os arquilos regionais dos models numa pasta `models` e os textos usados nas views numa pasta `views`.
<sup>[[link](#organize-locale-files)]</sup>

  * Quando a organização dos arquivos regionais é feita com diretórios adicionais, esses diretórios devem ser descritos no arquivo `application.rb` para poderem ser carregados. 

      ```Ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  Coloque as opções compartilhadas de localização, como data ou formatos de moeda, em arquivos no root da pasta `locales`.
<sup>[[link](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  Use a forma abreviada dos métodos de I18n: `I18n.t` ao invés de `I18n.translate` e `I18n.l` ao invés de `I18n.localize`.
<sup>[[link](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  Use a busca "preguiçosa" por textos usados nas views. Vamos supor que nós temos a seguinte estrutura:
<sup>[[link](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  O valor de `users.show.title` pode ser buscado no template `app/views/users/show.html.haml` assim:

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Use as chaves separadas por pontos nos controllers e models ao invés de especificar a opção `:scope`. A chamada separada por pontos é mais fácil de ler e de traçar a hierarquia.
<sup>[[link](#dot-separated-keys)]</sup>

  ```Ruby
  # ruim
  I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]

  # bom
  I18n.t 'activerecord.errors.messages.record_invalid'
  ```

* <a name="i18n-guides"></a>
  Mais informações sobre o I18n do Rails pode ser encontrada nos [Guias do Rails](http://guides.rubyonrails.org/i18n.html).
<sup>[[link](#i18n-guides)]</sup>

## Assets

Use o [pipeline de assets](http://guides.rubyonrails.org/asset_pipeline.html) para maximizar a organização dentro da sua aplicação.

* <a name="reserve-app-assets"></a>
  Reserve a pasta `app/assets` para stylesheets, javascripts ou imagens customizadas.
<sup>[[link](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Use a pasta `lib/assets` para suas próprias bibliotecas que não se encaixam muito bem no escopo da aplicação.
<sup>[[link](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Códigos de terceiros, como o [jQuery](http://jquery.com/) ou [bootstrap](http://twitter.github.com/bootstrap/), devem ser colocados na pasta `vendor/assets`.
<sup>[[link](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  Quando possível, use versões "gemificadas" dos assets (i.e.,
  [jquery-rails](https://github.com/rails/jquery-rails),
  [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails),
  [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass),
  [zurb-foundation](https://github.com/zurb/foundation)).
<sup>[[link](#gem-assets)]</sup>

## Mailers

* <a name="mailer-name"></a>
  Nomeie os mailers como `SomethingMailer`. Sem o sufixo "Mailer", não fica imediatamente aparente o que é um mailer e quais views são relacionadas ao mailer.
<sup>[[link](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Forneça templates tanto em HTML quando em texto puro.
<sup>[[link](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Habilite os erros lançados quando há falhas no envio de emails no seu ambiente de desenvolvimento. Esses erros são desabilitados por padrão.
<sup>[[link](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Use um servidor SMTP local como o [Mailcatcher](https://github.com/sj26/mailcatcher) no ambiente de desenvolvimento.
<sup>[[link](#local-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.smtp_settings = {
    address: 'localhost',
    port: 1025,
    # mais configurações
  }
  ```

* <a name="default-hostname"></a>
  Forneça configurações padrão para o nome do host.
<sup>[[link](#default-hostname)]</sup>

  ```Ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'your_site.com' }

  # na sua classe mailer
  default_url_options[:host] = 'your_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  Se você precisar usar um link para o seu site num e-mail, sempre use os métodos `_url`, não os métodos `_path`. Os métodos `_url` incluem o nome do host e os métodos `_path` não.
<sup>[[link](#url-not-path-in-email)]</sup>

  ```Ruby
  # ruim
  You can always find more info about this course
  <%= link_to 'here', course_path(@course) %>

  # bom
  You can always find more info about this course
  <%= link_to 'here', course_url(@course) %>
  ```

* <a name="email-addresses"></a>
  Formate os endereços `de` e `para` adequadamente. Use o seguinte formato:
<sup>[[link](#email-addresses)]</sup>

  ```Ruby
  # na sua classe mailer
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Certifique-se de que o método de envio de e-mails para o seu ambiente de desenvolvimento está definido como `test`:
<sup>[[link](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  O método de envio para desenvolvimento e produção deve ser `smtp`:
<sup>[[link](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  Quando enviar e-mails HTML, todos os estilos devem estar inline, porque alguns clientes de e-mail têm problemas com estilos externos. No entanto, isso torna os templates mais difíceis de manter e leva a duplicação de código. Existem duas gems similares que transformam os estilos e os colocam nas tags HTML correspondentes: [premailer-rails](https://github.com/fphilipe/premailer-rails) e [roadie](https://github.com/Mange/roadie).
<sup>[[link](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Enviar e-mails enquanto gera páginas de resposta deve ser evitado. Isso causa atrados no carregamento da página e a requisição pode exceder o tempo limite se múltiplos e-mails estão sendo enviados. Para superar isso, os e-mails podem ser enviados em processos de segundo placo com a ajuda da gem [sidekiq](https://github.com/mperham/sidekiq).
<sup>[[link](#background-email)]</sup>

## Hora

* <a name="tz-config"></a>
  Configure seu fuso horário adequadamente em `application.rb`.
<sup>[[link](#tz-config)]</sup>

  ```Ruby
  config.time_zone = 'Eastern European Time'
  # opcional - note que isso pode ser apenas :utc ou :local (o padrão é :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Não use `Time.parse`.
<sup>[[link](#time-parse)]</sup>

  ```Ruby
  # ruim
  Time.parse('2015-03-02 19:05:37') # => Vai assumir que a string dada está no fuso horário do sistema.

  # bom
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Não use `Time.now`.
<sup>[[link](#time-now)]</sup>

  ```Ruby
  # ruim
  Time.now # => Retorna o horário do sistema e ignora o fuso horário configurado.

  # bom
  Time.zone.now # => Fri, 12 Mar 2014 22:04:47 EET +02:00
  Time.current # Mesma coisa, mas mais curto.
  ```

## Bundler

* <a name="dev-test-gems"></a>
  Coloque as gems usadas só para desenvolvimento ou teste nos grupos apropriados no Gemfile.
<sup>[[link](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  Use apenas gems estabelecidas nos seus projetos. Se você estiver considerando incluir alguma gem pouco conhecida, você deve fazer uma revisão cuidadosa do código fonte dela primeiro.
<sup>[[link](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  Gems específicas de um sistema operacional vão por padrão resultar num `Gemfile.lock` constantemente mudando para múltiplos desenvolvedores usando diferentes sistemas operacionais. Coloque todas as gems específicas do OS X num grupo chamado `darwin` no Gemfile, e todas as gems específicas do Linux num grupo chamado `linux`:
<sup>[[link](#os-specific-gemfile-locks)]</sup>

  ```Ruby
  # Gemfile
  group :darwin do
    gem 'rb-fsevent'
    gem 'growl'
  end

  group :linux do
    gem 'rb-inotify'
  end
  ```

  Para carregar as gems apropriadas no ambiente certo, coloque o sequinte em `config/application.rb`:

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Não tire o `Gemfile.lock` do controle de versão. Esse arquivo não é um arquivo qualquer gerado aleatoriamente - ele garante que todos os membros da sua equipe tenham as mesmas versões das gem quando eles fizerem `bundle install`.
<sup>[[link](#gemfile-lock)]</sup>

## Gems problemáticas

Essa é uma lista de gem que, ou são problemáticas, ou foram suplantadas por outras gems. Você deve evitar usá-las nos seus projetos.

* [rmagick](http://rmagick.rubyforge.org/) - essa gem é famosa por seu consumo de memória. Use [minimagick](https://github.com/minimagick/minimagick) ao invés dela.

* [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/) - antiga solução para rodar testes automaticamente. Muito inferior a [guard](https://github.com/guard/guard) ou [watchr](https://github.com/mynyml/watchr).

* [rcov](https://github.com/relevance/rcov) - ferramenta de cobertura de código, não compatível com Ruby 1.9. Use [SimpleCov](https://github.com/colszowka/simplecov) no lugar dela.

* [therubyracer](https://github.com/cowboyd/therubyracer) - o uso dessa gem em produção é fortemente desencorajado porque ela usa uma grante quantidade de memória. Eu sugeriria usar `node.js` ao invés dela.

Essa lista também é um trabalho em andamento. Por favor, me fale caso você saiba de outras gem populares, mas problemáticas.

## Gerenciando processos

* <a name="foreman"></a>
  Se seus projetos dependem de vários processos externos, use [foreman](https://github.com/ddollar/foreman) para gerenciá-los.
<sup>[[link](#foreman)]</sup>

# Leituras adicionais

Existem alguns recursos excelentes sobre o estilo em Rails, que você deveria considerar olhar caso tenha tempo livre:

* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Better Specs for RSpec](http://betterspecs.org)

# Contribuindo

Nada que está escrito nesse guia é imutável. É meu desejo trabalhar junto com todos que estejam interessados no estilo de codificação em Rails, para que nós possamos ultimamente criar um recurso que será benéfico para a comunidade Ruby inteira.

Sinta-se livre para abrir tickets ou pull requests com melhorias. Obrigado desde já pela sua ajuda!

Você também pode apoiar o projeto (e o RuboCop) com contribuições financeiras através do [gittip](https://www.gittip.com/bbatsov).

[![Contribuir via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

## Como contribuir?

É simples, basta seguir as [diretrizes de contribuição](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md).

# Licença

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 3.0 Unported
License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

# Espalhe

Um guia de estilo criado pela comunidade não serve pra muita coisa numa comunidade que não sabe que ele existe. Tuite sobre o guia, compartilhe com seus amigos e colegas. Cada comentário, sugestão ou opinião que nós recebemos torna o guia um pouquinho melhor. E nós queremos ter o melhor guia possível, não é?

Cheers,<br/>
[Bozhidar](https://twitter.com/bbatsov)
