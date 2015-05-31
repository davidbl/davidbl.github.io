---
layout: post
title: 5 problems in a hour in Ruby
---

Just so that I can record my solutions

Number 1

```ruby
#for loop  (tbh, I had to look up the syntax for a 'for' loop in ruby)

def sum_with_for_loop(array)
  acc = 0
  for i in array
    acc += i
  end
  acc
end

def sum_with_while_loop(array)
  acc = i = 0
  while i < array.length
    acc += array[i]
    i += 1
  end
  acc
end

def sum_with_recursion(array)
  return 0 if array.length == 0
  array[0] + sum_with_recursion(array[1..-1])
end

# how i would probably do it in Ruby
def sum_idiomatic(array)
  array.reduce(&:+)
end
```

Number 2

```ruby
def alternate_array_elements(arr1,arr2)
  arr1.zip(arr2).flatten
end
```

Number 3

```ruby
def n_fibs(count)
  (count-2).times.reduce([0,1]){ |acc,_| acc << acc[-2] + acc[-1]}
end
```

Number 4

```ruby
#brute force
def largest_int_brute(array)
  array.map(&:to_s).permutation.map{ |e| e.join.to_i }.max
end

# a bit smarter
def largest_int(array)
  array.map(&:to_s).sort{ |a,b|  b+a <=> a+b}.join.to_i }
end
```

Number 5
```ruby
#brute force
def arrangments(sum=100)
  # 8 is the number of operators needed to fill the gaps in the 1..9 array
  numbers = (1..9)
  ['-','+',''].repeated_permutation(8).map{ |e| numbers.zip(e).join }.select{ |s| eval(s)==sum }
end
```

Full disclosure,  number 5 pushed my total time to more than an hour

