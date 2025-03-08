+++
authors = ["Matthew Macdonald"]
title = "First Post"
date = "2025-03-06"
description = "First blog post, mainly to feel it out"
tags = [
    "misc"
]
+++

Welcome to my new blog. I'll be using this as a place to write about the things I learn and any interesting or useful findings during my tech journey.

There will be posts that are more of a brain dump, and posts that are polished for the consumption of others.

The rest of this post will be random types of formatting, just to get a feel for the way it handles markdown - and some of the other functionality.

# Heading 1
Large heading test

## Heading 2
Heading 2 test...

### Code Block testing
#### Python Codeblock
```python
oodles = ['Poodle', 'Groodle', 'Cavoodle', 'Spoodle']
for oodle in oodles:
    print(f'Hello {oodle}')
```

#### Powershell code block
```powershell
$oodles = @('Poodle', 'Groodle', 'Cavoodle', 'Spoodle')
foreach ($oodle in $oodles){
    Write-Host "Hello $oodle"
}
```

#### Nested list

* Fruit
  * Apple
  * Orange
  * Banana
* Dairy
  * Milk
  * Cheese
    * Nesting
      * More nesting
        * And more

### Image (Figure)

{{< figure src="/images/first-post-image.png" title="Test Image" >}}
