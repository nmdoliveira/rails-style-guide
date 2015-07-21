# Prelúdio

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

Esse guia de estido do Rails recomenda boas práticas para que programadores Rails reais escrevam código que possa ser mantido por outros programadores Rails reais. Um guia de estilo que reflete o uso real é usado, enquanto um guia de estido que se apega a um ideal que foi rejeitado pelas pessoas que ele deveria ajudar está sob risco de cair em desuso &ndash; não importa o quão bom ele seja.

Esse guia está dividido em várias seções de regras relacionadas. Eu tentei incluir o motivo por trás das regras (se ele foi omitido, é porque eu assumi que é óbvio).

Eu não inventei essas regras do nada - elas são baseadas principalmente na minha extensa carreira como engenheiro de software profissional, feedback e sugestões de membros da comunidade Rails e vários outros recursos conceituados de programação Rails. 

## Sumário

* [Configuração](#configuração)
* [Roteamento](#roteamento)
* [Controllers](#controllers)
* [Models](#models)
  * [ActiveRecord](#activerecord)
  * [Consultas com ActiveRecord](#consultas-com-activerecord)
* [Migrações](#migrações)
* [Views](#views)
* [Internacionalização](#internacionalização)
* [Assets](#assets)
* [Mailers](#mailers)
* [Tempo](#tempo)
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
  Crie um ambiente adicional chamado `staging` que se assemelhe ao ambiente `production`.
<sup>[[link](#staging-like-prod)]</sup>

* <a name="yaml-config"></a>
  Deixe quaisquer configurações adicionais em arquivos YAML na pasta `config/`.
<sup>[[link](#yaml-config)]</sup>

  Desde o Rails 4.2, arquivos de configuração YAML podem ser carregados facilmente com o novo método `config_for`:

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

  * Override the `to_param` method of the model. This method is used by Rails
    for constructing a URL to the object.  The default implementation returns
    the `id` of the record as a String.  It could be overridden to include another
    human-readable attribute.

      ```Ruby
      class Person
        def to_param
          "#{id} #{name}".parameterize
        end
      end
      ```

  In order to convert this to a URL-friendly value, `parameterize` should be
  called on the string. The `id` of the object needs to be at the beginning so
  that it can be found by the `find` method of ActiveRecord.

  * Use the `friendly_id` gem. It allows creation of human-readable URLs by
    using some descriptive attribute of the model instead of its `id`.

      ```Ruby
      class Person
        extend FriendlyId
        friendly_id :name, use: :slugged
      end
      ```

  Check the [gem documentation](https://github.com/norman/friendly_id) for more
  information about its usage.

* <a name="find-each"></a>
  Use `find_each` to iterate over a collection of AR objects. Looping through a
  collection of records from the database (using the `all` method, for example)
  is very inefficient since it will try to instantiate all the objects at once.
  In that case, batch processing methods allow you to work with the records in
  batches, thereby greatly reducing memory consumption.
<sup>[[link](#find-each)]</sup>


  ```Ruby
  # bad
  Person.all.each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').each do |person|
    person.party_all_night!
  end

  # good
  Person.find_each do |person|
    person.do_awesome_stuff
  end

  Person.where('age > 21').find_each do |person|
    person.party_all_night!
  end
  ```

* <a name="before_destroy"></a>
  Since [Rails creates callbacks for dependent
  associations](https://github.com/rails/rails/issues/3458), always call
  `before_destroy` callbacks that perform validation with `prepend: true`.
<sup>[[link](#before_destroy)]</sup>

  ```Ruby
  # bad (roles will be deleted automatically even if super_admin? is true)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # good
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```

### ActiveRecord Queries

* <a name="avoid-interpolation"></a>
  Avoid string interpolation in
  queries, as it will make your code susceptible to SQL injection
  attacks.
<sup>[[link](#avoid-interpolation)]</sup>

  ```Ruby
  # bad - param will be interpolated unescaped
  Client.where("orders_count = #{params[:orders]}")

  # good - param will be properly escaped
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Consider using named placeholders instead of positional placeholders
  when you have more than 1 placeholder in your query.
<sup>[[link](#named-placeholder)]</sup>

  ```Ruby
  # okish
  Client.where(
    'created_at >= ? AND created_at <= ?',
    params[:start_date], params[:end_date]
  )

  # good
  Client.where(
    'created_at >= :start_date AND created_at <= :end_date',
    start_date: params[:start_date], end_date: params[:end_date]
  )
  ```

* <a name="find"></a>
  Favor the use of `find` over `where`
when you need to retrieve a single record by id.
<sup>[[link](#find)]</sup>

  ```Ruby
  # bad
  User.where(id: id).take

  # good
  User.find(id)
  ```

* <a name="find_by"></a>
  Favor the use of `find_by` over `where`
when you need to retrieve a single record by some attributes.
<sup>[[link](#find_by)]</sup>

  ```Ruby
  # bad
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # good
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```

* <a name="find_each"></a>
  Use `find_each` when you need to process a lot of records.
<sup>[[link](#find_each)]</sup>

  ```Ruby
  # bad - loads all the records at once
  # This is very inefficient when the users table has thousands of rows.
  User.all.each do |user|
    NewsMailer.weekly(user).deliver_now
  end

  # good - records are retrieved in batches
  User.find_each do |user|
    NewsMailer.weekly(user).deliver_now
  end
  ```

* <a name="where-not"></a>
  Favor the use of `where.not` over SQL.
<sup>[[link](#where-not)]</sup>

  ```Ruby
  # bad
  User.where("id != ?", id)

  # good
  User.where.not(id: id)
  ```
* <a name="squished-heredocs"></a>
  When specifying an explicit query in a method such as `find_by_sql`, use
  heredocs with `squish`. This allows you to legibly format the SQL with
  line breaks and indentations, while supporting syntax highlighting in many
  tools (including GitHub, Atom, and RubyMine).
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
    # further complexities...
  SQL
  ```

  [`String#squish`](http://apidock.com/rails/String/squish) removes the indentation and newline characters so that your server
  log shows a fluid string of SQL rather than something like this:

  ```
  SELECT\n    users.id, accounts.plan\n  FROM\n    users\n  INNER JOIN\n    acounts\n  ON\n    accounts.user_id = users.id
  ```

## Migrations

* <a name="schema-version"></a>
  Keep the `schema.rb` (or `structure.sql`) under version control.
<sup>[[link](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Use `rake db:schema:load` instead of `rake db:migrate` to initialize an empty
  database.
<sup>[[link](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Enforce default values in the migrations themselves instead of in the
  application layer.
<sup>[[link](#default-migration-values)]</sup>

  ```Ruby
  # bad - application enforced default value
  def amount
    self[:amount] or 0
  end
  ```

  While enforcing table defaults only in Rails is suggested by many
  Rails developers, it's an extremely brittle approach that
  leaves your data vulnerable to many application bugs.  And you'll
  have to consider the fact that most non-trivial apps share a
  database with other applications, so imposing data integrity from
  the Rails app is impossible.

* <a name="foreign-key-constraints"></a>
  Enforce foreign-key constraints. As of Rails 4.2, ActiveRecord
  supports foreign key constraints natively.
  <sup>[[link](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  When writing constructive migrations (adding tables or columns),
  use the `change` method instead of `up` and `down` methods.
  <sup>[[link](#change-vs-up-down)]</sup>

  ```Ruby
  # the old way
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # the new prefered way
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
  Don't use model classes in migrations. The model classes are constantly
  evolving and at some point in the future migrations that used to work might
  stop, because of changes in the models used.
<sup>[[link](#no-model-class-migrations)]</sup>

## Views

* <a name="no-direct-model-view"></a>
  Never call the model layer directly from a view.
<sup>[[link](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Never make complex formatting in the views, export the formatting to a method
  in the view helper or the model.
<sup>[[link](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Mitigate code duplication by using partial templates and layouts.
<sup>[[link](#partials)]</sup>

## Internationalization

* <a name="locale-texts"></a>
  No strings or other locale specific settings should be used in the views,
  models and controllers. These texts should be moved to the locale files in the
  `config/locales` directory.
<sup>[[link](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  When the labels of an ActiveRecord model need to be translated, use the
  `activerecord` scope:
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

  Then `User.model_name.human` will return "Member" and
  `User.human_attribute_name("name")` will return "Full name". These
  translations of the attributes will be used as labels in the views.


* <a name="organize-locale-files"></a>
  Separate the texts used in the views from translations of ActiveRecord
  attributes. Place the locale files for the models in a folder `models` and the
  texts used in the views in folder `views`.
<sup>[[link](#organize-locale-files)]</sup>

  * When organization of the locale files is done with additional directories,
    these directories must be described in the `application.rb` file in order
    to be loaded.

      ```Ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  Place the shared localization options, such as date or currency formats, in
  files under the root of the `locales` directory.
<sup>[[link](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  Use the short form of the I18n methods: `I18n.t` instead of `I18n.translate`
  and `I18n.l` instead of `I18n.localize`.
<sup>[[link](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  Use "lazy" lookup for the texts used in views. Let's say we have the following
  structure:
<sup>[[link](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  The value for `users.show.title` can be looked up in the template
  `app/views/users/show.html.haml` like this:

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Use the dot-separated keys in the controllers and models instead of specifying
  the `:scope` option. The dot-separated call is easier to read and trace the
  hierarchy.
<sup>[[link](#dot-separated-keys)]</sup>

  ```Ruby
  # bad
  I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]

  # good
  I18n.t 'activerecord.errors.messages.record_invalid'
  ```

* <a name="i18n-guides"></a>
  More detailed information about the Rails I18n can be found in the [Rails
  Guides](http://guides.rubyonrails.org/i18n.html)
<sup>[[link](#i18n-guides)]</sup>

## Assets

Use the [assets pipeline](http://guides.rubyonrails.org/asset_pipeline.html) to leverage organization within
your application.

* <a name="reserve-app-assets"></a>
  Reserve `app/assets` for custom stylesheets, javascripts, or images.
<sup>[[link](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Use `lib/assets` for your own libraries that don’t really fit into the
  scope of the application.
<sup>[[link](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Third party code such as [jQuery](http://jquery.com/) or
  [bootstrap](http://twitter.github.com/bootstrap/) should be placed in
  `vendor/assets`.
<sup>[[link](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  When possible, use gemified versions of assets (e.g.
  [jquery-rails](https://github.com/rails/jquery-rails),
  [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails),
  [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass),
  [zurb-foundation](https://github.com/zurb/foundation)).
<sup>[[link](#gem-assets)]</sup>

## Mailers

* <a name="mailer-name"></a>
  Name the mailers `SomethingMailer`. Without the Mailer suffix it isn't
  immediately apparent what's a mailer and which views are related to the
  mailer.
<sup>[[link](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Provide both HTML and plain-text view templates.
<sup>[[link](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Enable errors raised on failed mail delivery in your development environment.
  The errors are disabled by default.
<sup>[[link](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Use a local SMTP server like
  [Mailcatcher](https://github.com/sj26/mailcatcher) in the development
  environment.
<sup>[[link](#local-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.smtp_settings = {
    address: 'localhost',
    port: 1025,
    # more settings
  }
  ```

* <a name="default-hostname"></a>
  Provide default settings for the host name.
<sup>[[link](#default-hostname)]</sup>

  ```Ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'your_site.com' }

  # in your mailer class
  default_url_options[:host] = 'your_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  If you need to use a link to your site in an email, always use the `_url`, not
  `_path` methods. The `_url` methods include the host name and the `_path`
  methods don't.
<sup>[[link](#url-not-path-in-email)]</sup>

  ```Ruby
  # bad
  You can always find more info about this course
  <%= link_to 'here', course_path(@course) %>

  # good
  You can always find more info about this course
  <%= link_to 'here', course_url(@course) %>
  ```

* <a name="email-addresses"></a>
  Format the from and to addresses properly. Use the following format:
<sup>[[link](#email-addresses)]</sup>

  ```Ruby
  # in your mailer class
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Make sure that the e-mail delivery method for your test environment is set to
  `test`:
<sup>[[link](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  The delivery method for development and production should be `smtp`:
<sup>[[link](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  When sending html emails all styles should be inline, as some mail clients
  have problems with external styles. This however makes them harder to maintain
  and leads to code duplication. There are two similar gems that transform the
  styles and put them in the corresponding html tags:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) and
  [roadie](https://github.com/Mange/roadie).
<sup>[[link](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Sending emails while generating page response should be avoided. It causes
  delays in loading of the page and request can timeout if multiple email are
  sent. To overcome this emails can be sent in background process with the help
  of [sidekiq](https://github.com/mperham/sidekiq) gem.
<sup>[[link](#background-email)]</sup>

## Time

* <a name="tz-config"></a>
  Config your timezone accordingly in `application.rb`.
<sup>[[link](#tz-config)]</sup>

  ```Ruby
  config.time_zone = 'Eastern European Time'
  # optional - note it can be only :utc or :local (default is :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Don't use `Time.parse`.
<sup>[[link](#time-parse)]</sup>

  ```Ruby
  # bad
  Time.parse('2015-03-02 19:05:37') # => Will assume time string given is in the system's time zone.

  # good
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Don't use `Time.now`.
<sup>[[link](#time-now)]</sup>

  ```Ruby
  # bad
  Time.now # => Returns system time and ignores your configured time zone.

  # good
  Time.zone.now # => Fri, 12 Mar 2014 22:04:47 EET +02:00
  Time.current # Same thing but shorter.
  ```

## Bundler

* <a name="dev-test-gems"></a>
  Put gems used only for development or testing in the appropriate group in the
  Gemfile.
<sup>[[link](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  Use only established gems in your projects. If you're contemplating on
  including some little-known gem you should do a careful review of its source
  code first.
<sup>[[link](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  OS-specific gems will by default result in a constantly changing
  `Gemfile.lock` for projects with multiple developers using different operating
  systems.  Add all OS X specific gems to a `darwin` group in the Gemfile, and
  all Linux specific gems to a `linux` group:
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

  To require the appropriate gems in the right environment, add the
  following to `config/application.rb`:

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Do not remove the `Gemfile.lock` from version control. This is not some
  randomly generated file - it makes sure that all of your team members get the
  same gem versions when they do a `bundle install`.
<sup>[[link](#gemfile-lock)]</sup>

## Flawed Gems

This is a list of gems that are either problematic or superseded by
other gems. You should avoid using them in your projects.

* [rmagick](http://rmagick.rubyforge.org/) - this gem is notorious for its
  memory consumption. Use
  [minimagick](https://github.com/minimagick/minimagick) instead.

* [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/) - old solution for
  running tests automatically. Far inferior to
  [guard](https://github.com/guard/guard) and
  [watchr](https://github.com/mynyml/watchr).

* [rcov](https://github.com/relevance/rcov) - code coverage tool, not compatible
  with Ruby 1.9. Use [SimpleCov](https://github.com/colszowka/simplecov)
  instead.

* [therubyracer](https://github.com/cowboyd/therubyracer) - the use of this gem
  in production is strongly discouraged as it uses a very large amount of
  memory. I'd suggest using `node.js` instead.

This list is also a work in progress. Please, let me know if you know other
popular, but flawed gems.

## Managing processes

* <a name="foreman"></a>
  If your projects depends on various external processes use
  [foreman](https://github.com/ddollar/foreman) to manage them.
<sup>[[link](#foreman)]</sup>

# Further Reading

There are a few excellent resources on Rails style, that you should consider if
you have time to spare:

* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Better Specs for RSpec](http://betterspecs.org)

# Contributing

Nothing written in this guide is set in stone. It's my desire to work together
with everyone interested in Rails coding style, so that we could ultimately
create a resource that will be beneficial to the entire Ruby community.

Feel free to open tickets or send pull requests with improvements. Thanks in
advance for your help!

You can also support the project (and RuboCop) with financial contributions via
[gittip](https://www.gittip.com/bbatsov).

[![Support via Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)

## How to Contribute?

It's easy, just follow the [contribution guidelines](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md).

# License

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
This work is licensed under a [Creative Commons Attribution 3.0 Unported
License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

# Spread the Word

A community-driven style guide is of little use to a community that doesn't know
about its existence. Tweet about the guide, share it with your friends and
colleagues. Every comment, suggestion or opinion we get makes the guide just a
little bit better. And we want to have the best possible guide, don't we?

Cheers,<br/>
[Bozhidar](https://twitter.com/bbatsov)
