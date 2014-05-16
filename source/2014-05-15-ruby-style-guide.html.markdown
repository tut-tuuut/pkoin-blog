---
title: ruby style guide
date: 2014-05-15 20:07 UTC
tags: ruby
---

Voici maintenant quelques mois que je code en Ruby et je n'ai jamais pris le temps de centraliser les bonnes pratiques. Il est donc temps de m'y mettre !  
Je me suis grandement inspiré de l'excellent [The Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide).  
Surtout, ne lisez pas la traduction française, elle n'est pas à jour, incomplète et parfois fausse !

![Ruby](images/ruby-style-guide/ruby.png)

Il existe plusieurs outils en ligne de commande pour vérifier le bon respect de ce guide mais, personnellement, j'utilise [Rubocop](https://github.com/bbatsov/rubocop).  
Sachez aussi que vous pouvez écrire du code ruby plus "safe" grace à `ruby -w`.

## Encodage des fichiers

UTF-8 par défaut.

<i class="fa fa-info-circle"></i> Les différentes versions de Ruby n'ont pas toujours utilisé le même encodage par défaut :

* ruby <= 1.8 n'avait pas de notion d'encodage. Les chaînes étaient plus ou moins des tableaux d'octets
* ruby 1.9 utilisait l'US_ASCII comme encodage par défaut. Il était possible de définir l'encodage en UTF-8 en ajoutant en haut de fichier : `# encoding: utf-8`
* ruby >= 2.0 utilise l'UTF-8 par défaut

## Espacement & Saut de ligne

Utilisez des espaces autour des opérateurs, après les `:`, `;` et `,`, autour de `{` et devant `}`.

```ruby
sum = 1 + 2
a, b = 1, 2
1 > 2 ? true : false; puts 'Hi'
[1, 2, 3].each { |e| puts e }
some(arg).other
```

L'opérateur d'exposant est la seule exception.

```ruby
e = M * c**2
```

Pas d'espace après `(`, `[` ou devant `]`, `)`.  
Pas d'espace entre le nom d'une méthode et la parenthèse d'ouverture.  
Pas d'espace après `!`.

```ruby
some(arg).other
[1, 2, 3].length
f(3 + 2) + 1
!something
```

Utilisez des espaces autour de `=` lors de l'assignement des valeurs par défaut aux paramètres de méthode.

```ruby
def some_method(arg1 = :default, arg2 = nil, arg3 = [])
  # do something...
end
```

Sautez des lignes entre les `def`.

```ruby
def some_method
  result
end

def some_method
  result
end
```

Pensez à entourer le code interpolé d'un espace. Cela permet de mieux distinguer le code de la chaîne de caractères.

```ruby
"#{ user.last_name }, #{ user.first_name }"
```

## Identation & Alignement

Utilisez deux espaces par niveau d'indentation.  Pas de tabulation.

Indentez `when` au même niveau que `case`.

```ruby
case
when song.name == 'Misty'
  puts 'Not again!'
when song.duration > 120
  puts 'Too long!'
when Time.now.hour > 21
  puts "It's too late"
else
  song.play
end

kind = case year
       when 1850..1889 then 'Blues'
       when 1890..1909 then 'Ragtime'
       when 1910..1929 then 'New Orleans Jazz'
       when 1930..1939 then 'Swing'
       when 1940..1950 then 'Bebop'
       else 'Jazz'
       end
```

Alignez les paramètres de l'appel de méthode s'ils s'étalent sur plus d'une ligne.

```ruby
def send_mail(source)
  Mailer.deliver(to: 'bob@example.com',
                 from: 'us@example.com',
                 subject: 'Important message',
                 body: source.text)
end
```

## Lisibilité

Utilisez RDoc et ses conventions pour la documentation d'API.  
Limiter les lignes à 80 caractères.  
Utiliser des fins de ligne de type Unix.  
Ne laisser pas des espaces en fin de ligne.

Ajoutez des tirets bas aux grands expressions numériques.

```ruby
num = 1_000_000
```

Évitez les prolongements de ligne `\`. Sauf dans le cas de prolongement de chaine.

```ruby
long_string = 'First part of the long string' \
              ' and second part of the long string'
```

N'utilisez pas de parenthèses autour des conditions d'un `if`, `unless`, `while`, `until`.

Utilisez `||=` pour initialiser des variables uniquement si elles ne le sont pas déjà.  
Ne l'utilisez pas pour des variables booléennes.

```ruby
name ||= 'Bozhidar'
enabled = true if enabled.nil?
```

Utilisez `&&=` pour changer la valeur d'une variable qui existe déjà. Cela supprime l'intérêt de tester son existance avec `if`.

```ruby
something &&= something.downcase
```

Préfixez par `_` les paramètres de bloc et les variables locales inutilisés.

```ruby
result = hash.map { |_, v| v + 1 }

def something(x)
  _, used_var = something_else(x)
  # ...
end
```

Utilisez `[*var]` ou `Array()`, au lieu de d'une vérification explicite, lorsque vous voulez traiter une variable comme un `Array` mais vous n'êtes pas sûr que c'est un `Array`.

```ruby
[*paths].each { |path| do_something(path) }
Array(paths).each { |path| do_something(path) }
```

Utilisez les plages ou `Comparable#between?` au lieu d'une comparaison logique complexe.

```ruby
do_something if (1000..2000).include?(x)
do_something if x.between?(1000, 2000)
```

Préférez les méthodes dédiées aux comparaisons explicites avec `==`. Sauf pour les numériques.

```ruby
if x.even?
end

if x.odd?
end

if x.nil?
end

if x.zero?
end

if x == 0
end
```

Ne faites pas de vérification non-`nil` sauf sur les valeurs booléennes.

```ruby
do_something if something

def value_set?
  !@some_boolean.nil?
end
```

Préférez :

* `map` à `collect`
* `find` à `detect`
* `select` à `find_all`
* `reduce` à `inject`
* `size` à `count` ou `lenght`

Privilégiez la notation littérale pour créer des tableaux ou des hashs (sauf si vous devez passer des paramètres à leurs constructeurs).

```ruby
arr = []
hash = {}
```

Privilégiez `%w` à la syntaxe littérale de création de tableau quand vous avez besoin d'un tableau de chaines de caractères.

```ruby
STATES = %w(draft open closed)
```

Privilégiez `%i` à la syntaxe littérale de création de tableau quand vous avez besoin d'un tableau de symboles.

```ruby
STATES = %i(draft open closed)
```

## Bonnes pratiques

Quand vous concevez des hiérarchies de classes, assurez-vous qu'elles sont conformes au [Principe de substitution de Liskov](http://fr.wikipedia.org/wiki/Principe_de_substitution_de_Liskov) et essayez de les rendre aussi [SOLIDES](http://en.wikipedia.org/wiki/SOLID_\(object-oriented_design\)) que possible.

Utilisez `::`seulement pour référencer des constantes (de classes ou de modules) et des constructeurs (comme `Array()` ou `Nokogiri::HTML()`). Jamais pour invoquer des méthodes.

```ruby
SomeClass.some_method
some_object.some_method
SomeModule::SomeClass::SOME_CONST
SomeModule::SomeClass()
```

Tirez parti du fait que `if` et `case` sont des expressions qui renvoient un résultat.

```ruby
result =
  if condition
    x
  else
    y
  end
```

N'utilisez jamais `and` et `or`. Ils sont bannis.  
Utilisez toujours `&&` et `||`.

N'utilisez jamais `for`. Utiliser `each`.

Evitez `self` quand il n'est pas nécessaire. Il est nécessaire uniquement pour appeller un accesseur d'écriture local.

```ruby
def ready?
  if last_reviewed_at > last_updated_at
    worker.update(content, options)
    self.status = :in_progress
  end
  status == :verified
end
```

Utilisez `$stdout/$stderr/$stdin` au lieu de `STDOUT/STDERR/STDIN`.  
Ce sont des constantes, vous pouvez les ré-assigner et vous aurez un avertissement de la part de l'interpréteur.

Utilisez `warn` au lieu de `$stderr.puts`.  
Plus clair, plus concis. `warn` vous permet également de supprimer les avertissements au besoin (en définissant le niveau d'avertissement à 0 via `-W0`).

Préférez les modules aux classes qui comportent uniquement des méthodes. Les classes doivent être utilisé uniquement si il y a un intérêt de créer une instance de ces-dernières.

```ruby
# bad
class SomeClass
  def self.some_method
    # body omitted
  end

  def self.some_other_method
  end
end

# good
module SomeClass
  module_function

  def some_method
    # body omitted
  end

  def some_other_method
  end
end
```

Fournissez toujours une méthode `to_s` qui représente vos objets.

```ruby
class Person
  attr_reader :first_name, :last_name

  def initialize(first_name, last_name)
    @first_name = first_name
    @last_name = last_name
  end

  def to_s
    "#{@first_name} #{@last_name}"
  end
end
```

Privilégiez le [duck-typing](http://fr.wikipedia.org/wiki/Duck_typing) plutôt que l'héritage.

```ruby
# bad
class Animal
  # abstract method
  def speak
  end
end

# extend superclass
class Duck < Animal
  def speak
    puts 'Quack! Quack'
  end
end

# extend superclass
class Dog < Animal
  def speak
    puts 'Bau! Bau!'
  end
end

# good
class Duck
  def speak
    puts 'Quack! Quack'
  end
end

class Dog
  def speak
    puts 'Bau! Bau!'
  end
end
```

Utilisez `def self.method` pour définir les méthodes statiques.

```ruby
class TestClass
  def self.some_other_method
    # body omitted
  end

  # Also possible and convenient when you
  # have to define many singleton methods.
  class << self
    def first_method
      # body omitted
    end

    def second_method_etc
      # body omitted
    end
  end
end
```

Utilisez `Set` au lieu de `Array` quand vous travaillez sur des éléments uniques.  
`Set` implémente une collection de valeurs uniques désordonnées. C'est un mélange entre les fonctionnalités pratiques et intuitives d'un `Array` et l'accès rapide dans un `Hash`.

Préférez les symboles aux chaines pour les clés dans un `Hash`.

```ruby
hash = { one: 1, two: 2, three: 3 }
```

Utilisez `Hash#fetch` quand vous souhaitez récupérer une clé présente dans un `Hash`.

```ruby
heroes = { batman: 'Bruce Wayne', superman: 'Clark Kent' }
# bad - if we make a mistake we might not spot it right away
heroes[:batman] # => "Bruce Wayne"
heroes[:supermann] # => nil

# good - fetch raises a KeyError making the problem obvious
heroes.fetch(:supermann)
```

Jouez avec `Hash#fetch` pour définir une valeur par défaut lorsque la clé n'existe pas dans un `Hash`.

```ruby
batman = { name: 'Bruce Wayne', is_evil: false }

# bad - if we just use || operator with falsy value we won't get the expected result
batman[:is_evil] || true # => true

# good - fetch work correctly with falsy values
batman.fetch(:is_evil, true) # => false
```

Si vous avez besoin de transiter par une méthode pour récupérer la valeur par défaut, utilisez un block. La méthode sera appelée uniquement si besoin.

```ruby
batman.fetch(:powers) { get_batman_powers }
```

Evitez d'utiliser `String#+` pour assembler de longues chaînes de caractères.  
Utilisez plutôt `String#<<` qui transforme la chaîne sur place et s'avère toujours plus rapide que `String#+`, qui crée beaucoup de nouveaux objets.

```ruby
html = ''
html << '<h1>Page title</h1>'

paragraphs.each do |paragraph|
  html << "<p>#{paragraph}</p>"
end
```

## Nommage

Utilisez le `snake_case` pour :

* les symboles, les méthodes et les variables
* les noms de fichier (ex: `hello_world.rb`)
* les noms de répertoire (ex: `lib/hello_world/hello_world.rb`)

```ruby
:some_symbol

some_var = 5

def some_method
  ...
end
```

Utilisez le `CamelCase` pour les classes et les modules.  
Conservez les acronymes comme HTTP, RFC, XML en `uppercase`.

```ruby
class SomeClass
  ...
end

class SomeXML
  ...
end
```

Essayez d'avoir une classe ou un module par fichier. Nommez ce fichier avec le nom de la classe ou du module, en remplacant le `CamelCase` par du `snake_case`.

Utilisez le `SCREAMING_SNAKE_CASE` pour les constantes.

```ruby
SOME_CONST = 5
```

Seulement les méthodes qui retournent une valeur booléenne doivent se finir par `?`.

```ruby
["foo", "bar"].empty?
```

Les méthodes dangereuses (qui modifient `self` ou les arguments, ...) doivent se finir par `!`. Une version plus sûr de la méthode dangereuse doit toujours exister.

```ruby
class Person
  def update!
  end

  def update
  end
end
```

## Commentaires

> Un code de qualité constitue sa propre documentation. Quand vous êtes sur le point d'ajouter un commentaire, demandez-vous, "Comment puis-je améliorer le code pour que ce commentaire ne soit pas nécessaire ?" Améliorez le code et documentez le ensuite pour le rendre encore plus limpide. *Steve McConnell*

Utilisez : 

* `TODO` pour marquer les fonctionnalités manquantes ou qui devraient être ajoutées ultérieurement
* `FIXME` pour signaler un code erroné qui doit être corrigé
* `OPTIMIZE` pour signaler un code lent ou inefficace pouvant poser des problèmes de performance
* `HACK` pour signaler un code qui semble être issu de pratiques d'écriture douteuses qui devrait être refactorisé
* `REVIEW` pour signaler toute chose devant être vérifiée pour confirmer qu'elle fonctionne comme prévu. Par exemple: `REVIEW: Are we sure this is how the client does X currently?`
* d'autres mots clés d'annotation personnalisés si vous considérez que c'est approprié, mais assurez-vous de les documenter dans le README de votre projet.

## Exceptions

Signalez les exceptions en utilisant la méthode `fail`.  
Utilisez `raise` uniquement lorsque vous capturez une exception et la levez à nouveau (parcequ'il ne s'agit pas d'un échec mais d'une levée d'exception explicite et intentionnelle).

```ruby
begin
  fail 'Oops'
rescue => error
  raise if error.message != 'Oops'
end
```

N'utilisez jamais `return` dans un bloc `ensure`. Cela donnerait la priorité au retour sur une éventuelle levée d'exception, et la méthode retournerait un résultat comme si aucune exception n'avait été levée. En fait, l'exception serait ignorée silencieusement.

```ruby
def foo
  begin
    fail
  ensure
    return 'very bad idea'
  end
end
```

Utilisez des blocs `begin` implicites autant que possible.

```ruby
def foo
  # main logic goes here
rescue
  # failure handling goes here
end
```

Limitez la prolifération des blocs `begin` en utilisant les méthodes de contingence (un terme inventé par Avdi Grimm.)

```ruby
def with_io_error_handling
  yield
rescue IOError
  # handle IOError
end

with_io_error_handling { something_that_might_fail }
with_io_error_handling { something_else_that_might_fail }
```

Evitez `rescue Exception`.  
Cela aurait pour effet d'intercepter les signaux et appels à `exit`, et nécesiterait de `kill -9` le processus.

```ruby
# bad
begin
  # calls to exit and kill signals will be caught (except kill -9)
  exit
rescue Exception
  puts "you didn't really want to exit, right?"
  # exception handling
end

# good
begin
  # a blind rescue rescues from StandardError, not Exception as many
  # programmers assume.
rescue => e
  # exception handling
end

# also good
begin
  # an exception occurs here

rescue StandardError => e
  # exception handling
end
```

Libérez les ressources externes ouvertes par votre programme dans un bloc `ensure`.

```ruby
f = File.open('testfile')
begin
  # .. process
rescue
  # .. handle error
ensure
  f.close unless f.nil?
end
```

Privilégiez l'utilisation d'exceptions standards plutôt que d'introduire de nouvelles classes d'exception.

Enjoy! \o/
