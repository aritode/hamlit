# Hamlit

Hamlit is a high performance [Haml](https://github.com/haml/haml) implementation.

## Introduction

### What is Hamlit?
Hamlit is another implementation of [Haml](https://github.com/haml/haml).
With some [limitations](REFERENCE.md#limitations) by design for performance,
Hamlit is **7.16x times faster** than original haml gem in [this benchmark](benchmark/slim/run-benchmarks.rb),
which is an HTML-escaped version of [slim-template/slim's one](https://github.com/slim-template/slim/blob/v3.0.6/benchmarks/run-benchmarks.rb) for fairness.

![Hamlit Benchmark](http://i.gyazo.com/4fe00ff2ac2fa959dfcf86a5e27dc914.png)

```
    erubis:   114501.6 i/s
    hamlit:   112888.1 i/s - 1.01x slower
      slim:   103298.5 i/s - 1.11x slower
      faml:    88675.4 i/s - 1.29x slower
      haml:    15750.6 i/s - 7.27x slower
```

### Why is Hamlit faster?

#### Less string concatenation by design
As written in [limitations](REFERENCE.md#limitations), Hamlit drops some not-so-important features which require
works on runtime. With the optimized language design, we can reduce the string concatenation
to build attributes.

#### Temple optimizers
Hamlit is built with [Temple](https://github.com/judofyr/temple), which is a framework to build
template engines and also used in Slim. By using the framework and its optimizers, Hamlit can
reduce string allocation and concatenation easily.

#### Static analyzer
Hamlit analyzes Ruby expressions with Ripper and render it on compilation if the expression
is static. And Hamlit can also compile string literal with string interpolation to reduce
string allocation and concatenation on runtime.

#### C extension to build attributes
While Hamlit has static analyzer and static attributes are rendered on compilation,
dynamic attributes must be rendered on runtime. So Hamlit optimizes rendering on runtime
with C extension.

## Usage

See [REFERENCE.md](REFERENCE.md) for detail features of Hamlit.

### Rails

Add this line to your application's Gemfile or just replace `gem "haml"` with `gem "hamlit"`.
It enables rendering by Hamlit for \*.haml automatically.

```rb
gem 'hamlit'
```

If you want to use view generator, consider using [hamlit-rails](https://github.com/mfung/hamlit-rails).

### Sinatra

Replace `gem "haml"` with `gem "hamlit"` in Gemfile, and require "hamlit".
See [sample/sinatra](sample/sinatra) for working sample.

While Haml disables `escape_html` option by default, Hamlit enables it for security.
If you want to disable it, please write:

```rb
set :haml, { escape_html: false }
```


## Command line interface

You can see compiled code or rendering result with "hamlit" command.

```bash
$ gem install hamlit
$ hamlit --help
Commands:
  hamlit compile HAML    # Show compile result
  hamlit help [COMMAND]  # Describe available commands or one specific command
  hamlit parse HAML      # Show parse result
  hamlit render HAML     # Render haml template
  hamlit temple HAML     # Show temple intermediate expression

$ cat in.haml
- user_id = 123
%a{ href: "/users/#{user_id}" }

# Show compiled code
$ hamlit compile in.haml
_buf = [];  user_id = 123;
; _buf << ("<a href='/users/".freeze); _buf << (::Hamlit::Utils.escape_html((user_id))); _buf << ("'></a>\n".freeze); _buf = _buf.join

# Render html
$ hamlit render in.haml
<a href='/users/123'></a>
```

## Contributing

### Development

Contributions are welcomed. It'd be good to see
[Temple's EXPRESSIONS.md](https://github.com/judofyr/temple/blob/v0.7.6/EXPRESSIONS.md)
to learn Temple which is a template engine framework used in Hamlit.

```bash
$ git clone https://github.com/k0kubun/hamlit
$ cd hamlit
$ bundle install

# Run all tests
$ bundle exec rake test

# Run one test
$ bundle exec ruby -Ilib:test -rtest_helper test/hamlit/line_number_test.rb -l 12

# Show compiling/rendering result of some template
$ bundle exec exe/hamlit compile in.haml
$ bundle exec exe/hamlit render in.haml

# Use rails app to debug Hamlit
$ cd sample/rails
$ bundle install
$ bundle exec rails s
```

### Reporting an issue

Please report an issue with following information:

- Full error backtrace
- Haml template
- Ruby version
- Hamlit version
- Rails/Sinatra version

## License

Copyright (c) 2015 Takashi Kokubun
