---
title: Exemples de MetaProgramació dins de Ruby on Rails
date: 2024-10-31 00:00:00 +0000
categories: []
tags: []
lang: cat
---

![Desenmascarat](assets/meta-programming.webp){: style="display: block; margin-left: auto; margin-right: auto; width: 50%; height: auto;" }

## Que es la metaprogramació?
La metaprogramació és una tècnica de programació en la qual un programa té la capacitat de tractar altres programes com a dades. Això vol dir que pot llegir, generar, analitzar o transformar codi mentre s’està executant, i fins i tot modificar-se a si mateix en temps real.

En altres paraules, és programar un programa perquè escrigui o alteri altres programes o a ell mateix. Això és molt potent perquè permet, per exemple, automatitzar tasques repetitives, crear components reutilitzables i escriure codi més flexible i adaptable.

En llenguatges com Ruby, la metaprogramació és força comuna i s'utilitza per crear funcionalitats avançades de manera dinàmica, com definir mètodes durant l'execució o crear DSLs (Domain-Specific Languages).

## Mètodes bàsics
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

