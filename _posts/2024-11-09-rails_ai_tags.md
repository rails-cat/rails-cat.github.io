---
title: Creació de tags html d'imatge sense src.
date: 2024-11-09 00:00:00 +0000
categories: []
tags: []
lang: ca
---


# Crear i Integrar Imatges Generades per IA a Rails amb la Gem `rails_ai_tag`

Amb l'augment de les capacitats de la intel·ligència artificial, cada vegada més desenvolupadors busquen integrar aquestes tecnologies als seus projectes. Una aplicació pràctica molt interessant és la generació d'imatges a partir de descripcions de text, especialment útil per enriquir experiències visuals de forma dinàmica. Aquí és on entra en joc `rails_ai_tag`, una gem que facilita aquesta funcionalitat dins d'aplicacions Rails.

## Què és `rails_ai_tag`?

La gem `rails_ai_tag` és una eina per a desenvolupadors Rails que volen generar imatges amb IA de manera senzilla. Aquesta gem s'integra amb serveis d'intel·ligència artificial com el de OpenAI per generar imatges a partir de descripcions textuals i inserir-les directament en les vistes Rails, com si fos una etiqueta HTML `<img>`.

Amb `rails_ai_tag`, només cal proporcionar una descripció de la imatge que vols crear, i la gem s'encarrega de comunicar-se amb l'API de generació d’imatges, obtenir l'URL de la imatge generada i inserir-la a la pàgina.

## Com Funciona `rails_ai_tag`?

`rails_ai_tag` utilitza el client d’OpenAI per enviar descripcions a l'API i rebre URL d'imatges generades. Aquesta gem inclou:

- **Un servei** per gestionar les peticions a OpenAI.
- **Un helper** que permet inserir la imatge fàcilment en les vistes Rails.

### Exemple d’ús

Un cop instal·lada i configurada la gem, pots generar una imatge amb només una línia en la teva vista:

```erb
<%= ai_image_tag("Un bosc serè amb la llum del sol filtrant-se entre els arbres") %>
```
Aquest codi crida l’API de generació d’imatges amb el text “Un bosc serè amb la llum del sol filtrant-se entre els arbres” i insereix la imatge resultant directament a la pàgina web.
![Bosc](assets/ai_tag_bosc.png){: style="display: block; margin-left: auto; margin-right: auto; width: 50%; height: auto;" }

## Instal·lació i Configuració de `rails_ai_tag`

### Pas 1: Instal·la la Gem

Afegir `rails_ai_tag` al teu `Gemfile`:

```ruby
gem 'rails_ai_tag'
```

Després executa `bundle install` per instal·lar la gem.

### Pas 2: Configura l'API d'OpenAI

Per utilitzar `rails_ai_tag`, necessites una clau d'API d'OpenAI. Pots obtenir-la des de [la web d’OpenAI](https://platform.openai.com/signup).

Un cop tinguis la clau, crea un fitxer `.env` per a emmagatzemar-la de forma segura (pots utilitzar la gem `dotenv-rails` per carregar variables d'entorn en el desenvolupament local):

```bash
echo "OPENAI_API_KEY=la_teva_clau_api" > .env
```

### Pas 3: Utilitza el Helper en les Vistes Rails

Ara pots començar a utilitzar `ai_image_tag` en qualsevol vista de Rails:

```erb
<%= ai_image_tag("Una posta de sol a les muntanyes") %>
```
![Bosc](assets/ai_tag_montanyes.png){: style="display: block; margin-left: auto; margin-right: auto; width: 50%; height: auto;" }
## Casos d'Ús de `rails_ai_tag`

- **Personalització Dinàmica**: Imagina una aplicació on els usuaris poden descriure la seva imatge ideal, i aquesta es genera automàticament.
- **Prototipat Ràpid**: Quan desenvolupes una aplicació i vols imatges personalitzades sense haver de cercar-les manualment.
- **E-commerce**: Generar imatges basades en la descripció dels productes que es venen, ideal per prototipar nous dissenys o per a productes que encara no s'han fotografiat.

## Consideracions a Tenir en Compte

Tot i que `rails_ai_tag` és una eina potent, té algunes limitacions:

- **Cost d’ús de l’API**: La generació d'imatges amb OpenAI o altres serveis similars pot tenir un cost per ús, especialment si la teva aplicació fa moltes crides a l'API.
- **Limitacions del Model**: Les imatges generades poden no ser sempre exactes al 100% segons la descripció, ja que depèn de les capacitats i limitacions del model.
- **Privadesa i Seguretat**: Quan utilitzes serveis externs, assegura't de complir amb les polítiques de privadesa i de tractament de dades sensibles.

## Conclusió

La gem `rails_ai_tag` aporta una funcionalitat innovadora i potent per als desenvolupadors de Rails que volen aprofitar la intel·ligència artificial per crear imatges a partir de text. Amb aquesta gem, el procés d'integració d'imatges generades automàticament es simplifica, permetent-te afegir creativitat i interactivitat a la teva aplicació Rails.

Si vols afegir un toc visual únic al teu projecte, prova `rails_ai_tag` i experimenta amb la generació d'imatges IA al teu abast. Aquesta gem podria ser un primer pas cap a una nova era d’interacció entre text i imatge en el món de les aplicacions web.
