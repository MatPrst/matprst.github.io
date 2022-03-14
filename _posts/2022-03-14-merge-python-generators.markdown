---
layout: post
title:  "merge python generators"
date:   2022-03-14 17:40:00 +0100
categories: python iterator generator itertools
---

Recently, I had to interleave two large files into one. The obvious solution is to read both files, then iterate over them and alternatively write one line from each onto the new file. My issue was that none of them would fit into memory. The next solution is to use [python generators][python-generator] which prevent storing all the file contents into memory. I found that itertools has a cool function [itertools.chain()][python-chain] which creates an iterator from the iterable arguments. The issue now is that I would not interleave the files together.

I came up with this solution:

{% highlight python %}
def merge_iterators(iterators: List[Iterator]) -> Iterator:
  """
  Returns a generator composed of an interleaving of the input generators.
  Args:
    - generators: list of generators to merge
  Return:
    - new generator
  """
  finished = [False for it in iterators]
  while not all(finished):
      for i, it in enumerate(iterators):
          try:
              yield next(it)
          except StopIteration:
              finished[i] = True

# Examples
gen1 = (_ for _ in "abc"]) # dummy generator yielding a, b, c
gen2 = range(4) # dummy generator yielding 0, 1, 2, 3
gen3 = (_ for _ in "xy"]) # dummy generator yielding x, y

merge_generators([gen1, gen2, gen3]) # yields a, 0, x, b, 1, y, c, 2, 3
{% endhighlight %}

The function takes a list of iterators and interleaves them into one generator. The input iterators do not have to yield the same number of objects. When one of the iterators is finished (when it raises a `StopIteration`{:.python} exception), the function continues by interleaving the remaining iterators.

[python-generator]: https://docs.python.org/3/glossary.html#term-generator
[python-chain]: https://docs.python.org/3/library/itertools.html#itertools.chain

