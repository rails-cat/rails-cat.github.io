---
title: Exemples de MetaProgramació dins de Ruby on Rails
date: 2024-10-31 00:00:00 +0000
categories: []
tags: []
lang: ca
---

![Desenmascarat](assets/meta-programming.webp){: style="display: block; margin-left: auto; margin-right: auto; width: 50%; height: auto;" }

## Que es la metaprogramació?
La metaprogramació és una tècnica de programació en la qual un programa té la capacitat de tractar altres programes com a dades. Això vol dir que pot llegir, generar, analitzar o transformar codi mentre s’està executant, i fins i tot modificar-se a si mateix en temps real.

En altres paraules, és programar un programa perquè escrigui o alteri altres programes o a ell mateix. Això és molt potent perquè permet, per exemple, automatitzar tasques repetitives, crear components reutilitzables i escriure codi més flexible i adaptable.

En llenguatges com Ruby, la metaprogramació és força comuna i s'utilitza per crear funcionalitats avançades de manera dinàmica, com definir mètodes durant l'execució o crear DSLs (Domain-Specific Languages).

### Mètodes bàsics
- Amb el mètode `send`, es poden cridar mètodes de forma dinàmica, passant el nom del mètode com un símbol o una cadena. Això permet executar mètodes que potser no coneixem prèviament, o que han estat definits en temps d'execució. 
- La funció `define_method`, per la seva banda, permet crear mètodes dins d'una classe o mòdul en temps real, facilitant la creació de codi flexible o ajustat a les necessitats específiques d’una aplicació. 
- `method_missing` també és útil per capturar trucades a mètodes inexistents, oferint la possibilitat d'interpretar-les o derivar-les a altres mètodes
- `class_eval` i `instance_eval`: Permeten executar codi dins del context d’una classe o instància. Això possibilita afegir o modificar mètodes i variables d'instància des de fora de la definició original de la classe, donant molta flexibilitat per personalitzar el comportament de les classes en temps d'execució.
- `alias_method`: Crea un alias per a un mètode existent, cosa que permet redefinir-lo i mantenir una referència a l'original. Això és útil per afegir funcionalitat nova a un mètode sense perdre el comportament anterior, en un patró anomenat sovint method wrapping.
- `const_get` i `const_set`: Permeten accedir i definir constants dinàmicament dins d’una classe o mòdul.
- `respond_to?`: Verifica si un objecte respon a un mètode específic. Això és important en metaprogramació per assegurar-se que els mètodes dinàmics estan disponibles abans de cridar-los, especialment quan es treballa amb method_missing.
- `extend` i `include`: Amb extend, es poden afegir mòduls com a mètodes de classe a un objecte específic, mentre que include afegeix mòduls com a mètodes d'instància en una classe. Això permet crear comportament compartit que es pot injectar a objectes o classes segons sigui necessari.

Amb aquests mètodes, Ruby ofereix un conjunt potent de funcionalitats per construir codi dinàmic i flexible, sovint usat per crear DSLs o simplificar estructures de codi en aplicacions grans.

## ActiveRecord
Les associacions d'ActiveRecord semblen màgiques: afegeixes un `has_many` aquí, un `belongs_to` allà, i de sobte els teus models estan plens de comportament.

ActiveRecord i els seus mètodes d’associació són possibles gràcies a l'ús de les capacitats de la metaprogramació. Aquestes eines ofereixen millores significatives en el rendiment i l'eficiència, la llegibilitat del codi, i la simplificació de la resolució de problemes i la depuració, demostrant les possibilitats que aporta la metaprogramació.

### Configurant una Associació belongs_to

A Ruby on Rails, l’associació `belongs_to` forma part d’ActiveRecord i defineix una conexió de "un a un" entre dos models. És molt útil quan un model conté una clau forania que fa referència a la clau primària d’un altre model, establint una relació de pare-fill. Aquí tens una explicació detallada de com funciona `belongs_to` i com permet recuperar objectes relacionats.

Imaginem que tens dos models, *Post*  i *Author*. En aquest cas, cada *Post* està associat amb un únic *Author*, mentre que cada *Author* pot tenir múltiples *Posts*. Els models es definirien així:

```ruby
class Post < ApplicationRecord
  belongs_to :author
end

class Author < ApplicationRecord
  has_many :posts
end
```

En aquest exemple:
- El model *Post* té una associació `belongs_to` :author, cosa que vol dir que cada post està associat a un sol autor.
- El model *Author* té una associació has_many :posts, és a dir, un autor pot tenir múltiples posts.

Perquè això funcioni, la taula posts necessita una columna author_id, que emmagatzema l’id de l’autor associat.

### Recuperant Objectes Relacionats amb `belongs_to`

L’associació `belongs_to` afegeix automàticament mètodes per recuperar l’objecte associat. Així és com funciona:
- Enllaç de la Clau Forania: Active Record utilitza la clau forania author_id a la taula posts per localitzar el registre d’Author associat a la taula authors.
- Creació Dinàmica de Mètodes: La declaració `belongs_to` :author crea mètodes en Post per recuperar i establir l’autor associat.

Per exemple:
```ruby
post = Post.find(1)
author = post.author # recupera l’objecte Author associat
```
En aquest codi:
- post.author utilitza el camp author_id de l'objecte post per trobar el registre Author amb un id coincident.
- Aquesta cerca es realitza amb una sola consulta SQL, típicament similar a `SELECT * FROM authors WHERE id = post.author_id LIMIT 1`.

### Amb molt més detall
Seguint amb l' exemple la associació `belongs_to` del fitxer [`activerecord/lib/active_record/associations.rb`](https://github.com/rails/rails/blob/96ea77265b8295e9559bb4aeeeddc88463538872/activerecord/lib/active_record/associations.rb#L1689):

```ruby
def belongs_to(name, scope = nil, **options)
  reflection = Builder::BelongsTo.build(self, name, scope, options)
  Reflection.add_reflection self, name, reflection
end
```
D'aqui anirem a [`Builder::BelongsTo`](https://github.com/rails/rails/blob/96ea77265b8295e9559bb4aeeeddc88463538872/activerecord/lib/active_record/associations/builder/belongs_to.rb#L4) 
```ruby
class BelongsTo < SingularAssociation
```
pots comprovar tu mateix que SingularAssociation < Association si vols, i que a [`ActiveRecord::Associations::Builder`](https://github.com/rails/rails/blob/96ea77265b8295e9559bb4aeeeddc88463538872/activerecord/lib/active_record/associations/builder/belongs_to.rb#L4), dins la classe `Association`, mètode [`build`](https://github.com/rails/rails/blob/a8a72f531a33eed636a72188c8d6e64c931e4849/activerecord/lib/active_record/associations/builder/association.rb#L32) acabem cridant al mètode `create_reflection` i el metode `define_accessors`.
```ruby
  class Association # :nodoc:
    class << self
      attr_accessor :extensions
    end
    self.extensions = []

    VALID_OPTIONS = [
      :class_name, :anonymous_class, :primary_key, :foreign_key, :dependent, :validate, :inverse_of, :strict_loading, :query_constraints
    ].freeze # :nodoc:

    def self.build(model, name, scope, options, &block)
      if model.dangerous_attribute_method?(name)
        raise ArgumentError, "You tried to define an association named #{name} on the model #{model.name}, but " \
                             "this will conflict with a method #{name} already defined by Active Record. " \
                             "Please choose a different association name."
      end

      reflection = create_reflection(model, name, scope, options, &block)
      define_accessors(model, reflection)
      define_callbacks(model, reflection)
      define_validations(model, reflection)
      define_change_tracking_methods(model, reflection)
      reflection
    end

    def self.create_reflection(model, name, scope, options, &block)
      raise ArgumentError, "association names must be a Symbol" unless name.kind_of?(Symbol)

      validate_options(options)

      extension = define_extensions(model, name, &block)
      options[:extend] = [*options[:extend], extension] if extension

      scope = build_scope(scope)

      ActiveRecord::Reflection.create(macro, name, scope, options, model)
    end
```
Que ens porta al seguent codi rellevant: [`ActiveRecord::Reflection.create`](https://github.com/rails/rails/blob/bfd5b59a82c4fae7cc8d564e5c5f276f65e8cb92/activerecord/lib/active_record/reflection.rb#L18) i un cop tenim creada la reflection podem cridar [`define_accessors`](https://github.com/rails/rails/blob/462d7c7cff2a0a375449a52f6edac34068733464/activerecord/lib/active_record/associations/builder/association.rb#L95) que es on podem veure clarament la utilització de la funció de metaprogramació `class_eval`:

```ruby
    # Defines the setter and getter methods for the association
    # class Post < ActiveRecord::Base
    #   has_many :comments
    # end
    #
    # Post.first.comments and Post.first.comments= methods are defined by this method...
    def self.define_accessors(model, reflection)
      mixin = model.generated_association_methods
      name = reflection.name
      define_readers(mixin, name)
      define_writers(mixin, name)
    end

    def self.define_readers(mixin, name)
      mixin.class_eval <<-CODE, __FILE__, __LINE__ + 1
        def #{name}
          association(:#{name}).reader
        end
      CODE
    end

    def self.define_writers(mixin, name)
      mixin.class_eval <<-CODE, __FILE__, __LINE__ + 1
        def #{name}=(value)
          association(:#{name}).writer(value)
        end
      CODE
    end
```

[![Pot veure un video molr relacionat aquí](assets/ar-reflections.png)](https://www.youtube.com/watch?v=6qHKtAkqguc){: style="display: block; margin-left: auto; margin-right: auto; width: 50%; height: auto;" }



## ActionPack DSL de Rutes

Rails fa servir metaprogramació per construir rutes amb el seu propi DSL (Domain-Specific Language), creant automàticament les rutes i helpers associats a cada ruta.

### Configurant les rutes de la nostra aplicació

Exemple:
```ruby
Rails.application.routes.draw do
  resources :posts do
    resources :comments
  end
end
```

Això defineix automàticament rutes com `posts_path`, `new_post_path`, `post_comments_path`, etc., utilitzant metaprogramació per convertir aquestes declaracions en un conjunt de mètodes i rutes URL.

### Amb molt més detall(per un exemple més senzill)

```ruby
Rails.application.routes.draw do
  get "up" => "rails/health#show", as: :rails_health_check
end
```

Podem llegir al fitxer `config/routes.rb` de la nostra aplicació la crida a `Rails.application.routes.draw` passant com a parametre un bloc.

Busca on és definit aquest mètode. Exacte, a la gem ActionPack, fitxer [`actionpack/lib/action_dispatch/routing/route_set.rb`](https://github.com/rails/rails/blob/a7c8a20825be5ac8d96566ec5dbe9daa85568bc8/actionpack/lib/action_dispatch/routing/route_set.rb#L459)

```ruby
def draw(&block)
  puts "Block class: #{block.class}"      # Outputs the class (Proc)
  puts "Block inspection: #{block.inspect}" # Inspects the Proc object
  puts block.source
  clear! unless @disable_clear_and_finalize
  eval_block(block)
  finalize! unless @disable_clear_and_finalize
  nil
end
```
per poder imprimir el contingut de la variable `block` podem servir la gem `method_source`, així el que veurem serà:
```shell
Block class: Proc
Block inspection: #<Proc:0x00007f705dd10e78 /path/to/myapp/config/routes.rb:1>
Rails.application.routes.draw do
  get "up" => "rails/health#show", as: :rails_health_check
end
```
comencem a tenir evidencies del ús de metaprogramació, estem pasan codi com a paràmetre a la funció `draw`, i que fem amb aquest codi? 
Exacte:
```ruby
  eval_block(block)
```
el passem a eval_bloc:
```ruby
def eval_block(block)
  mapper = Mapper.new(self)
  if default_scope
    mapper.with_default_scope(default_scope, &block)
  else
    mapper.instance_exec(&block)
  end
end
```
on trobem el que buscavem, la evidència a la instrucció `mapper.instance_exec(&block)`.

Consultem la api de [ruby](https://rubyapi.org/3.3/o/basicobject#method-i-instance_exec) per els detalls
```
instance_exec(*args) public

Executes the given block within the context of the receiver (obj). In order to set the context, the variable self is set to obj while the code is executing, giving the code access to obj\’s instance variables. Arguments are passed as block parameters.
```

