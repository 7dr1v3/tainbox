# Tainbox

Tainbox is a utility gem that can be used to inject attributes into Ruby objects. It is similar to <a href="https://github.com/solnic/virtus">Virtus</a>, but works a bit more sensibly (hopefully) and throws in some additional features.

## But we already have attr_accessor!

Consider this code:

``` ruby
class Person
  include Tainbox
  attribute :name, default: -> { "person_#{age}" }
  attribute :age, Integer, default: 20
end
```

Here are the basic features you get:

``` ruby
person = Person.new(name: 'John', age: '30')
person.attributes # => { :name => "John", :age => 30 }
person.age = 50
person.age # => 50
person.attributes = {}
person.attributes # => { :name => "person_20", :age => 20 }
```

## But what's wrong with Virtus?

Observe:

``` ruby
class Person
  include Virtus::Model
  attribute :age, Integer
end

Person.new(age: 'invalid_integer').age # => "invalid_integer"
```

## Additional features

### Method overrides

Tainbox attributes can be freely overriden:

``` ruby
def name
  super.strip
end
```

``` ruby
def name=(value)
  super('Stupid example')
end
```

### #attribute_provided?

Attribute is considered provided if its setter was explicitly invoked via a setter, `.new`, or `#attributes=` or it has a default value.

``` ruby
class Person
  include Tainbox
  attribute :name, default: 'John'
  attribute :age
end

person = Person.new
person.attribute_provided?(:age) # => false
person.attribute_provided?(:name) # => true
```

### Readonly and writeonly attributes

Speaks for itself:

``` ruby
class Person
  include Tainbox
  attribute :name, writeonly: true
  attribute :age, readonly: true
end
```

### Built-in type converters

All converters return nil if conversion could not be made.

- Integer
- Float
- String
- Symbol
- Time
- Boolean

### Custom type converters

You can define a custom type converter like so:

``` ruby
# value and options are automatically available in scope
Tainbox.define_converter(MyType) do
  # Do whatever you want with value
  options[:strict] ? MyType.convert!(value) : MyType.convert(value)
end
```

### Suppressing tainbox initializer

If you don't want to pollute your class with tainbox initializer you can do:

``` ruby
class Person
  include Tainbox
  suppress_tainbox_initializer!
end
```
