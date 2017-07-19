+++
draft = false
categories = []
tags = []
bigimg = ""
subtitle = "A Look Inside Marked's Source Code"
date = "2017-07-16T23:16:22-07:00"
title = "How Does Markdown Parsers Work"

+++

[Marked](https://github.com/chjj/marked) is a popular markdown parser written in JavaScript. I looked into its source code and found that it was actually not hard to understand. All the important code is in a single JavaScript file which contains merely a thousand lines.

The source code is made up of the following parts:

1. Rules/Grammar for block level components (such as headings, blockquotes) as well as a `Lexer`. 
2. Rules/Grammar for inline components (such as strong tag, links) as well as an `InlineLexer`.
3. A `Renderer` that outputs final html code
4. A `Parser` that uses the results from lexer and pass them to Renderer.
5. Some helper functions.

Alright, I know there are some words that might sounds unfamiliar. I will explain in the section below.

### A Brief and Incomplete Introduction of Some Compiler Concepts

I have never taken a compiler design class. So I can't go into details of how a compiler work. Basically, a program needs to go through 5 stages before they can executed.

#### Lexer
Lexer converts a program into a list of tokens. Tokens are anything that has meaning in a program, for example, variables, operators, semicolons, or even indentations. A simple program like below,
```
a = 12;
b = 2;
c = a + b;
```
can be converted into the following tokens: `a`, `=`, `12`, `;`, `b`, `=`, `2`, `;`, `c`, `=`, `a`, `+`, `b`, `;`.

If you want to write a new language, most likely you don't need to implement a Lexer. You can find a lexer implementation in any language of your choice.

```
var lexer = new Lexer(); // create a lexer object

// Add a rule, so that when lexer finds this pattern in the code,
// it knows to associate this part of code with the tokenType.
// TokenType code be variable, operator, etc. 
lexer.addRule(pattern, tokenType); 
// Add more rules
lexer.addRule(pattern2, tokenType2);
...

// To use a lexer, usually you can just pass the program in string
lexer.lex("a = 12; b = 2; c = a + b;");
```

How does the lexer work internally? Intuitively, it just read through the code from beginning to end, try each pattern, see if one of them match, if it does, spit out a token and move the read head to the next position.

For markdown, the tokens are the elements, for example
```markdown
# Beautiful Day

This is a beautiful day, because finally someone reads my blog

> Thank You
```

are made of three tokens, a level 1 heading (value: `"Beautiful Day"`), a paragraph (value: `"This is a beautiful day, because finally someone reads my blog"`) and a blockquote (value: `"Thank You"`).

#### Parser

TO BE CONTINUED.

(Gosh, when can I finish a post in one go)
