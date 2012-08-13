# Typecaster

This gem was built for create text files based in fixed columns.

## Instalation

  gem install typecaster

## Usage

The Typecaster can be used for two things. To create text files based on positions and read text files based on some descriptor.

### Creating

Given you need generate a text with the following format:
```
  column name | size  | starting | ending | type   |
  name        | 10    | 0        | 9      | string |
  price       | 8     | 10       | 17     | number |
  code        | 6     | 18       | 23     | string |
```
Here's how to use it:
```
  module StringTypecaster
    def self.call(value, options)
      value.to_s.ljust(options[:size], " ")
    end
  end

  module NumerTypecaster
    def self.call(value, options)
      value.to_s.rjust(options[:size], "0")
    end
  end

  class ProductFormatter
    include Typecaster

    attribute :name, :size => 10, :class => StringTypecaster
    attribute :price, :size => 8, :class => NumberTypecaster
    attribute :code, :size => 6, :class => StringTypecaster
  end

  product = ProductFormatter.new(:name => 'Coca', :price => '25.0', :code => '6312')
  puts product.to_s # => 'Coca      000025.06312  '
```

### Reading

Given a file like that:

```
0SOME IMPORTANT TEXT 000001
119999FOO BAR        000002
110000XPTO BAR       000003
109901JOAO BAR       000004
939900               000005
```

Each row in that file was identified by a number 0 for header, 1 for the records inside and 9 for footer, each row has 27 chars, to read then you must implement the parsers for each row type.

```
module StringTypecaster
  def self.parse(text)
    text.strip
  end
end

module IntegerTypecaster
  def self.parse(text)
    text.to_i
  end
end

class MyFileHeader
  include Typecaster

  attribute :identifier, :size => 1, :class => StringTypecaster
  attribute :text, :size => 20, :class => StringTypecaster
  attribute :sequential, :size => 5, :class => IntegerTypecaster
end

class MyFileRow
  include Typecaster

  attribute :identifier, :size => 1, :class => StringTypecaster
  attribute :amount, :size => 5, :class => IntegerTypecaster
  attribute :name, :size => 15, :class => StringTypecaster
  attribute :sequential, :size => 5, :class => IntegerTypecaster
end

class MyFileFooter
  include Typecaster

  attribute :identifier, :size => 1, :class => StringTypecaster
  attribute :total, :size => 5, :class => IntegerTypecaster
  attribute :blanks, :size => 15, :class => StringTypecaster
  attribute :sequential, :size => 6, :class => IntegerTypecaster
end

class MyFileParser
  include Typecaster::Parse

  parser :header, :with => MyFileHeader, :identifier => '0'
  parser :rows, :with => MyFileRow, :identifier => '1', :array => true
  parser :footer, :with => MyFileFooter, :identifier => '9'
end

MyFileParser.parse(File.new('my_file.txt')
```

its results in an hash, with keys :header, :rows and :footer, :rows will be an array, you can also use the same object to generate the file. Try it by yourself :)

## Contributing

* Fork the project.
* Make your feature addition or bug fix.
* Add tests for it.
* Commit, do not mess with rakefile, version, or history.
* Send me a pull request.
