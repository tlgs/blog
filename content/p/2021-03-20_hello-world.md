+++
title = "Hello, World!"
customcss = "hello-world.css"
+++

Whenever I encounter a programming language that I haven't worked with,
I like to do what I call the **Hello, World! test** —
I take a peek at what the classic Hello, World! program looks like
and then judge the language entirely on _how it makes me feel_.

It's of course very naive to reduce a language to the look of a simple program,
but I think there's something powerful about this test:
you get a glimpse of the explicit complexity that comes with such a basic task.
I like to think of it this way: the old adage "Don't judge a book by its cover"
is good to keep in mind but you should still be able to say "That's one ugly ass cover".

It's easy to come up with arguments about the importance of a language's
first impression on an user (or about ease-of-use, or learning curves) that would elevate
the discussion about the importance of Hello, World!, but that's not fun;
Assigning arbitrary values to the subjective feel of a language is — so that's
what I'll do.

## Making a tier list

I decided to restrict the candidate list to languages in the 'Most Popular' and
'Most Loved' sections of
[Stack Overflow's 2020 Developer Survey](https://insights.stackoverflow.com/survey/2020).
I excluded HTML/CSS and SQL for hopefully obvious reasons
(though `SELECT 'hello, world';` would probably win it all).
This unfortunately means that we miss out on other interesting languages such as Lua,
Zig, Crystal, D, or Nim.

When judging each language I looked at what _should be_ the
idiomatic version of the program.
For those I'm not familiar with, I visited the language's project page and
searched for their recommended example.
If there was none (which I see as a bad sign), I looked at any linked
'Getting Started' resources.
If this failed, I resorted to online tutorials or Stack Overflow answers.
It's sad to have to admit that for some of these I ended up looking up
a reference introductory book to be confident I had found an idiomatic version.

|       |                                    |
| ------| ---------------------------------- |
| **S** | Bash, Python, Swift, JavaScript    |
| **A** | Julia, Ruby, PHP, Perl             |
| **B** | Dart, Kotlin, Rust                 |
| **C** | VBA, Go, C, Haskell, Scala, C++, R |
| **D** | Assembly                           |
| **F** | C#, Java, Objective-C              |

While I'm sure you can infer something about my taste from this list, I think
some versions are undeniably more beautiful than others:
if you take a look at Java and say "Hmmmm, that's one crispy Hello, World!",
I'd argue you should re-evaluate your
[taste as a programmer](http://www.paulgraham.com/taste.html).

Compiling this was an interesting exercise, and I now _definitely_ have a much better
grasp of the programming language landscape.
Joking aside, if you're actually interested in a meaningful analysis of Hello, World!
across programming languages, I recommend Drew DeVault's series of posts
([1](https://drewdevault.com/2020/01/04/Slow.html),
[2](https://drewdevault.com/2020/01/08/Re-Slow.html)).

## Source code

Below are the programs considered for each language,
along with any special notes about their source:

<details>
<summary><span>Bash / Shell / Powershell</span></summary>

```bash
echo 'hello, world'
```
</details>

<details>
<summary><span>Python</span></summary>

```python
print("hello, world")
```
</details>

<details>
<summary><span>Swift</span></summary>

```swift
print("hello, world")
```
</details>

<details>
<summary><span>JavaScript / TypeScript</span></summary>

```javascript
console.log("hello, world");
```
</details>

<details>
<summary><span>Julia</span></summary>

```julia
println("hello, world")
```
</details>

<details>
<summary><span>Ruby</span></summary>

```ruby
puts "hello, world"
```
</details>

<details>
<summary><span>PHP</span></summary>

```php
<?php
    echo 'hello, world';
?>
```
</details>

<details>
<summary><span>Perl</span></summary>

```perl
print "hello, world\n"
```
</details>

<details>
<summary><span>Dart</span></summary>

```dart
void main() {
  print('hello, world');
}
```
</details>

<details>
<summary><span>Kotlin</span></summary>

```kotlin
fun main() {
    println("hello, world")
}
```
</details>

<details>
<summary><span>Rust</span></summary>

```rust
fn main() {
    println!("hello, world");
}
```
</details>

<details>
<summary><span>VBA</span></summary>

```vba
WScript.Echo "hello, world"
```
</details>

<details>
<summary><span>Go</span></summary>

```go
package main

import "fmt"

func main() {
    fmt.Println("hello, world")
}
```
</details>

<details>
<summary><span>C</span></summary>

```c
#include <stdio.h>

int main(void) {
    puts("hello, world");
    return 0;
}
```
</details>

<details>
<summary><span>Haskell</span></summary>

```hs
main = putStrLn "hello, world"
```

source: [Learn you a Haskell for Great Good!](http://learnyouahaskell.com/)
</details>

<details>
<summary><span>Scala</span></summary>

```scala
object Hello {
    def main(args: Array[String]) = {
        println("hello, world")
    }
}
```
</details>

<details>
<summary><span>C++</span></summary>

```c++
#include <iostream>

int main() {
    std::cout << "hello, world\n";
}
```

source: [Bjarne Stroustrup, probably](https://en.wikipedia.org/wiki/C%2B%2B#Language)
</details>

<details>
<summary><span>R</span></summary>

```r
cat("hello, world\n")
```
</details>

<details>
<summary><span>Assembly (x86_64)</span></summary>

```asm
bits 64
section .text
global _start
_start:
    mov rdx, len
    mov rsi, msg
    mov rdi, 1
    mov rax, 1
    syscall

    mov rdi, 0
    mov rax, 60
    syscall

section .rodata
msg: db "hello world", 10
len: equ $-msg
```

source: [sircmpwn](https://drewdevault.com/2020/01/04/Slow.html)
</details>

<details>
<summary><span>C#</span></summary>

```c#
using System;

class Hello
{
    static void Main()
    {
        Console.WriteLine("hello, world");
    }
}
```

source: [A Tour of C#](https://docs.microsoft.com/en-us/dotnet/csharp/tour-of-csharp/#hello-world)
</details>

<details>
<summary><span>Java</span></summary>

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("hello, world");
    }
}
```
</details>

<details>
<summary><span>Objective-C</span></summary>

```objc
#import <Foundation/Foundation.h>

int main (int argc, const char * argv[]) {
    NSAutoreleasePool * pool = [[NSAutoreleasePool alloc] init];
    NSLog (@"hello, world");

    [pool drain];
    return 0;
}
```

source:
[Programming in Objective-C](https://www.pearson.com/us/higher-education/program/Kochan-Programming-in-Objective-C-6th-Edition/PGM106849.html),
and others
</details>
