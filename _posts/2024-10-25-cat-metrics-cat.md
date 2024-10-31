---
title: CatMetrics
date: 2024-10-25 00:00:00 +0000
categories: []
tags: []
lang: cat
---

# Creació gem de monitoratge i mètriques de peticions per una aplicació Ruby on Rails 8
 Crear una gem per a mètriques i monitoratge en aplicacions Rails pot començar amb un enfocament senzill que es pot ampliar gradualment amb funcionalitats més avançades. Aquí tens una guia per començar a construir una gem bàsica de mètriques i monitoratge:


## Definició del seguiment bàsic de mètriques
Primer, crearem una configuració bàsica per capturar mètriques comunes de l'aplicació, com el recompte de peticions, el temps de resposta i el número d'errors.

### Inicialització:
 Iniciarem el procés creant una gem amb bundle:
 ```bash
bundle gem cat_metrics
 ```

#### Inicialitza la configuració de la gem
A lib/cat_metrics.rb:
 ```ruby
module CatMetrics
  class << self
    attr_accessor :config

    def setup
      self.config ||= Configuration.new
      yield(config) if block_given?
    end
  end

  class Configuration
    attr_accessor :log_level

    def initialize
      @log_level = :info
    end
  end
end

 ```

#### Crea un inicialitzador per a aplicacions Rails per configurar la gem.

A l'aplicació Rails, els usuaris poden configurar la gem amb un inicialitzador així:

```ruby

# config/initializers/cat_metrics.rb
CatMetrics.setup do |config|
  config.log_level = :debug
end
```

### Emmagatzematge de mètriques:
 Comença emmagatzemant les mètriques en memòria (com un hash), i més endavant es pot ampliar per utilitzar una solució persistent (db o Redis).
 
 A lib/cat_metrics/store.rb:

```ruby
module CatMetrics
  class Store
    @metrics = {
      requests: [],
      errors: 0
    }

    class << self
      def add_request(duration, status)
        @metrics[:requests] << duration
        @metrics[:errors] += 1 if status >= 500
      end

      def average_response_time
        return 0 if @metrics[:requests].empty?
        @metrics[:requests].sum / @metrics[:requests].size
      end

      def error_rate
        return 0 if @metrics[:requests].empty?
        (@metrics[:errors].to_f / @metrics[:requests].size) * 100
      end

      def reset
        @metrics[:requests] = []
        @metrics[:errors] = 0
      end
    end
  end
end
```

### Middleware per a mètriques bàsiques

Afegeix un middleware de Rack per capturar detalls de les peticions i respostes:

A lib/cat_metrics/middleware/metrics_collector.rb:

```ruby
module CatMetrics
  module Middleware
    class MetricsCollector
      def initialize(app)
        @app = app
      end

      def call(env)
        start_time = Time.now
        status, headers, response = @app.call(env)
        duration = (Time.now - start_time) * 1000.0 # en ms

        # Emmagatzema les mètriques en memòria
        CatMetrics::Store.add_request(duration, status)

        [status, headers, response]
      end
    end
  end
end
```

Afegeix el middleware a l'aplicació Rails a lib/cat_metrics/railtie.rb:

```ruby
require 'cat_metrics/middleware/metrics_collector'

module CatMetrics
  class Railtie < Rails::Railtie
    initializer "cat_metrics.configure_rails_initialization" do |app|
      app.middleware.use CatMetrics::Middleware::MetricsCollector
    end
  end
end
```

## Mostra les mètriques bàsiques

Per fer-ho útil, crea una ruta per mostrar aquestes mètriques com una resposta JSON senzilla.

Afegeix un controlador a lib/cat_metrics/metrics_controller.rb:

```ruby
require 'json'

module CatMetrics
  class MetricsController < ActionController::Base
    def show
      metrics = {
        average_response_time: CatMetrics::Store.average_response_time,
        error_rate: CatMetrics::Store.error_rate
      }
      render json: metrics
    end
  end
end
```

A railtie.rb, munta aquesta ruta:

```ruby
module CatMetrics
  class Railtie < Rails::Railtie
    initializer "cat_metrics.configure_rails_initialization" do |app|
      app.middleware.use CatMetrics::Middleware::MetricsCollector

      app.routes.prepend do
        get '/metrics' => 'cat_metrics/metrics#show'
      end
    end
  end
end
```

## Tests

Afegirem tests per comprovar el correcte funcionament inicial i de les modificacions que es puguin realitzar.

## Construcció de la gem i publicació

### Modificació del gemspec


Publicarem la gem a rubygems.org per poder utilitzarla en tots els nostres projectes i compartir-la amb la comunitat.

### Creació d'un compte a rubygems.org

Si no tens un compte, es el moment de fer-lo.

## Proves i extensions

Comença provant que les mètriques es recopilen i es mostren correctament. 

A partir d'aquí, es poden afegir noves funcionalitats:

- Persistència: 
 Emmagatzema les mètriques a Redis per a un seguiment a llarg termini.
- Mètriques detallades:
 Realitza un seguiment de més mètriques, com els temps de resposta d'endpoints individuals.
- Alertes:
 Integra amb notificacions per correu electrònic o Slack per a índexs d'error elevats o temps de resposta llargs.
- Tauler:
 Crea una interfície d'usuari senzilla per visualitzar les mètriques al llarg del temps.

Aquesta configuració proporciona una base que es pot ampliar progressivament segons les necessitats dels usuaris.

> Compren els detalls per ajustar la funcionalitat a les teves necessitats
{: .prompt-tip }
