---
layout: post
title: "Mistune custom lexers - we are going deeper"
date:   2015-12-21
tags:
  - code
  - python
---

[Mistune](https://github.com/lepture/mistune) is the fastest markdown parser
written in pure Python. And it has good documentation. At least so it seemed
until I tried to write a custom lexer (hint: tests are a good source of inspiration).

## Vocabulary

Even though this is a recipe for solving a specific problem, I'll start off with
some basics and a definition list.

**Grammar** - set of rules for rewriting strings. In case of Markdown we are
rewriting rules like `**some string**` to **some string**. Basically I'm saying
"if I use two asterisks followed by any random string
and end with another 2 asterisks, wrap the string into an HTML `<strong`> tag".
The full list of those rules is available on [Daring Fireball website](https://daringfireball.net/projects/markdown/).
A formalized spec of Markdown, a big community effort is also available on
[CommonMark website](http://commonmark.org/). For example, it goes into much bigger detail [when
and how strong emphasis should be applied](http://spec.commonmark.org/0.22/#emphasis-and-strong-emphasis).

**Inline grammar** - rules of the grammar that when rewritten will appear, well, inline.
Examples of inline rules in Markdown are links, emphasizes, images, etc.

**Block grammar** - conversely to inline grammar, these rules of the grammar will
take up a whole block. Examples are headers, paragraphs, code blocks, lists, tables,
etc. Inline and block grammar rules directly translate to HTML definition of
inline vs block.

**Lexer** - program that is able to convert a series of characters into a
sequence of tokens according to the grammar, and it's doing that using regular
expressions.

**Renderer** - program that controls the output of grammar rules. There is no
one fixed way to go from `**strong**` to `<strong>strong</strong>`, but rather an
infinite number of ways to adjust that `<strong>strong</strong>` output.

Mistune offers a way to have custom renderers and custom lexers.
Add whatever specific rules and outputs you want. [The example from the docs](https://github.com/lepture/mistune/blob/master/README.rst#lexers) picks GitHub Wiki links and creates a custom inline renderer to output it as an HTML anchor
tag and an inline lexer that defines a regular expression capable of parsing
`[[Page 2|Page 2]]`.

## Scratching my itch

I want to be able to refer to people by their twitter username and link to their
twitter accounts when I'm writing in Markdown right on my blog (this is a theoretical
desire, I'm actually not using mistune on this blog yet :sad_face:). This is an inline grammar and it's
going to be composed of `@` and a sequence of characters that only accepts letters,
numbers and underscore. Replace twitter with any website that has accounts and direct
URLs to them.

Basically the thing twitter is already good at: recognizing @username and linking
to this account (if it exists).

Following the docs available on mistune README I came up with this:

{% highlight python %}
{% raw %}
import re
import mistune


class UsernameRenderer(mistune.Renderer):
    def username_link(self, link, username):
        return '<a href="%s">%s</a>' % (link, username)


class UsernameInlineLexer(mistune.InlineLexer):
    def enable_username_link(self):
        self.rules.username_link = re.compile(r'@(\w+)')
        self.default_rules.insert(3, 'username_link')

    def output_username_link(self, m):
        username = m.group(1)
        link = 'https://twitter.com/%s/' % username
        return self.renderer.username_link(link, username)


if __name__ == '__main__':
    renderer = UsernameRenderer()
    inline = UsernameInlineLexer(renderer)
    inline.enable_username_link()
    markdown = mistune.Markdown(renderer, inline=inline)
    print markdown('Hello @username')
    # <p>Hello @username</p>
{% endraw %}
{% endhighlight %}

Aaaand it doesn't do its job. It follows the example of the wiki link
quite closely: a custom renderer, a custom inline lexer which adds a new rule
using a really simply regular expression and then all the initialization. **Why
doesn't it work then?**

![Y U NO WORK]({{ site.baseurl }}/assets/img/mistune-lexers/yunowork.jpg)

## Going deeper

Surprise surprise! It doesn't even match my regular expression. To find out why I've
pdb-ed a couple of times into the one-file mistune package and learned the
following:

1. Mistune takes the initial string and runs it through all the rules (by the way
the `username_link` rule is there) to match at least something. It uses `re.match()`,
which specifically is looking for a pattern match at the beginning of the file
(for finding a match anywhere within the string `re.search()` is used). And yes, if we
try to do `print markdown('@username')` it will work as expected and return
`<p><a href="https://twitter.com/username/">username</a></p>`.
2. Once something was matched it will slice the matched bit from the string
`text = text[len(m.group(0)):]` and continue with point 1 until the whole string
is exhausted.

What happens in our case is that the string "Hello @username" is treated as a
whole paragraph. How come `Hello [[Link Text|Wiki Link]]` is recognized as a paragraph
that also contains a wiki link?

**The solution lies in the text pattern of the `InlineGrammar` class.** Look at this
``text = re.compile(r'^[\s\S]+?(?=[\\<!\[_*`~]|https?://| {2,}\n|$)')``. Zoom in
``(?=[\\<!\[_*`~]|https?://| {2,}\n|$)``. This is called a lookahead assertion, which
matches if the stuff between `(?=...)` will match, but doesn't actually consume it.
The lookahead assertion contains a couple of characters, which will make mistune
slice the string and try to render the token. `[` symbol is there, but `@` is not.
That's why the wiki link example works just fine and the username link example
doesn't.

## Another way

This other way involves overriding the text pattern to take into consideration
`@` symbol and tokenize the string per our wish. This is my ugly solution (code
below is scrollable to the right).


{% highlight python %}
{% raw %}
import copy
import re
import mistune


class UsernameInlineGrammar(mistune.InlineGrammar):
    username_link = re.compile(r'^@(\w+)')
    # Override the text grammar pattern to contain the `@` as a stop element in the lookahead assertion
    text = re.compile(r'^[\s\S]+?(?=[\\<!\[_*`~@]|https?://| {2,}\n|$)')


class UsernameUrlInlineLexer(mistune.InlineLexer):
    """Inline lexer for @<username> which links to the user twitter page."""
    default_rules = copy.copy(mistune.InlineLexer.default_rules)
    default_rules.insert(3, 'username_link')
    default_rules.append('text')

    def __init__(self, renderer, rules=None, **kwargs):
        if rules is None:
            rules = UsernameInlineGrammar()

        super(UsernameUrlInlineLexer, self).__init__(renderer, rules, **kwargs)

    def output_username_link(self, m):
        alt = m.group(0)
        username = m.group(1)
        url = 'https://twitter/{}/'.format(username)
        return self.renderer.link(url, False, alt)

if __name__ == '__main__':
    markdown = mistune.Markdown(inline=UsernameUrlInlineLexer)
    print markdown('Hello @username')
    # <p>Hello <a href="https://twitter/username/">@username</a></p>
{% endraw %}
{% endhighlight %}


In this case we are extending the `InlineGrammar`, adding a new rule and
overriding the existing `text` rule, which isn't a very pleasant experience.
Notice the `username_link` grammar regular expression has `^` (caret) symbol,
which tells it look from the very beginning of the string. Also notice the lack
of any custom renderer - I just decided to use the link renderer, which it yield
the same result.

## Fin

Congratulations! You've made it to the end! Got a better solution? Share it with
everybody or take a nap :)

![Cat nap]({{ site.baseurl }}/assets/img/mistune-lexers/catnap.jpeg)


