---
layout: post
title: "Ruby Notes"
date: 2014-11-06 21:50:50 -0600
comments: true
categories: ruby
---

## Lambdas vs Procs

> Both are proc objects

```ruby
l = lambda {|name| puts "hello #{name}"}
p = proc   {|name| puts "hello #{name}"}

l # => #<Proc:0x007ff16389da58@-:1 (lambda)>
p # => #<Proc:0x007ff16389da30@-:2>
```
> Return behaves differently

**lambda returns from the context of lambda only.**

<!-- more -->

```ruby

def do_it
  l = lambda do |name|
    puts "Hello #{name}"
    return
  end
  l.call("World!")
  puts "Bye World"
end

do_it
# >> Hello World!
# >> Bye World
```

**Proc returns from the calling context too.**
- Note that the bye world is not printed.

```ruby
def do_it
  l = proc do |name|
    puts "Hello #{name}"
    return
  end
  l.call("World!")
  puts "Bye World"
end

do_it
# >> Hello World!
```
