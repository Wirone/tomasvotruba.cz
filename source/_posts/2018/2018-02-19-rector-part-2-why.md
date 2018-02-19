---
id: 78
title: "Rector: Part 2 - Why is Needed and Founding Fathers"
perex: '''
    ...  
'''
todo_tweet: "..."
related_items: [63, 77] 
---

## Why was Rector needed - Instant Upgrades? 

2 things you need to upgrade PHP application to new version of the framework you use:

### 1. The Knowledge

Most of you follows changes in Symfony or Nette. You know that [`ProcessBuilder` was removed](https://github.com/symfony/symfony/blob/master/UPGRADE-4.0.md#process) and that you'll have to migrate to `Process` class only. You know that [`Nette\Object` was removed](https://forum.nette.org/cs/26250-pojdte-otestovat-nette-2-4-rc) and that you'll have to rewrite all `@method` annotation to real methods.

You read blogs, follow news on forum, read all `CHANGELOG.md` and `UPGRADE.md` files and sometimes commits, to find out what all has changed. **You have the knowledge**.

### 2. The Resources

You work in a company where up-to-date it very important value. Your employer finances upgrades and also your education in it (pays you to get the knowledge). Once a 6 month you have dedicated paid time to update all packages to most recent versions. **You have the resources**

Do you find yourself in such situation? If so, you're on of 5 % blessed people around me.

## How Expensive are Upgrades Now?

From my experience with consulting over 50 projects is that it can take 80-400 hours per one minor version upgrade, including all deprecations - e.g.from Symfony 2.7 to Symfony 2.8. 

### Teams are not Supported in Upgrades

Teams don't have space to find out what changed between Symfony 2.7 and Symfony 2.8. They don't care, because their employer cares mostly about creating that new e-commerce as fast and as cheap as possible. 
  
That's why most of lectures (@todo link) I make is about giving teams *the knowledge* that they can use with their very limited resources to make the best out of it.
 
That naturally leads to huge legacy code, team performance drop from 100 % to 20 %, which leads to hiring 5x more people etc. to keep productivity the same, more money, and pressure to faster development, which lead to huge legacy code...
  
### Are Deprecations Easy to Find?
 
Let's say you have time to explore the Internet, follow [Symfony News on Twitter](https://twitter.com/symfony_en), read [every news post on Symfony Blog](http://symfony.com/blog/category/living-on-the-edge) or know where on the Nette forum are located [Release Notes](https://forum.nette.org/en/f78-release-announcements-news).

Sometimes if you're lucky there is `UPGRADE-x.md` in project's Github repository, like [`UPGRADE-4.0.md`](https://github.com/symfony/symfony/blob/master/UPGRADE-4.0.md) in Symfony 4 repository. But what if you need upgrade to version 3.x? Could you find it? Well no, but yes in [3.x branch](https://github.com/symfony/symfony/tree/3.4).

Sometimes these changes are in files called `CHANGELOG-x.md`.

But more often they're newer to be seen and you have to go git blame in Github specific line and hope for an answer. And just pray that there is PR with more detailed changes with tests as well and not direct set of commits to the `master` branch without context.

### When We Find Them, are They Valid?

Sometimes there can be bare useless description, like [in `UPGRADE-4.0`](https://github.com/symfony/symfony/blob/master/UPGRADE-4.0.md#process):

*The `Symfony\Component\Process\ProcessBuilder` class has been removed, use the `Symfony\Component\Process\Process` class directly instead.*

So it easy as:

```diff
-use Symfony\Component\Process\ProcessBuilder;
+use Symfony\Component\Process\Process;

-$builder = new ProcessBuilder();
+$builder = new Process();
 $builder->setArguments(['build', '-force', '-var "blah=blah"', 'path/to/json.json'])
    ->getProcess();
```

Unfortunately no and our investigative programming begins. Git blame..., Google? Symfony Docs?

Sometimes they are embodied in the code - the best place to add them. When running this method, you'll be informed: 
 
```php
public function add()
{
    trigger_error('Method add() is deprecated, use addHtml() instead.', E_USER_DEPRECATED);
}
```

### Found them and Valid! But are They Standardized?

Sometimes the maintainer goes as far to deprecate code slowly (!= remove):

```php
trigger_error('Method add() is deprecated, use addHtml() instead.', E_USER_DEPRECATED);
```

Which is good enough for start. Symfony even has [PHPUnit Bridge](https://symfony.com/doc/current/components/phpunit_bridge.html) that tries to detect those deprecations. Would you know what exactly do you need to change?

<img src="https://symfony.com/doc/current/_images/report.png">

But what class? What line?

But that's not the only "standard".

<a href="https://xkcd.com/927/
">
    <img src="https://imgs.xkcd.com/comics/standards.png">
</a>

There is one more way I wrote about in [How to write Open-Source in PHP 3: Deprecating Code](/blog/2017/09/11/how-to-write-open-source-in-php-3-deprecating-code/#today-s-topic-changed-method-name). 

The `@deprecated` annotation:

```php
/**
 * @deprecated Method add() is deprecated, use addHtml() instead
 */
public function add()
{
    // ...
}
```

Which is rather note in the code than helpful to user. The `->add()` method is called, and what happens? Nothing. And in next mayor version you get *calling non-existing method* error.

Similar tool to PHPUnit Bridge is [`deprecation-detector`](https://github.com/sensiolabs-de/deprecation-detector), that tries to catch code with `@deprecated` annotation.

And that's Symfony, my friends, which does the best job for (Backward Compatibility Promise)[http://symfony.com/doc/current/contributing/code/bc.html] in PHP. What about those other 50 packages your application uses?

Would you like to do this job instead of developing your application? Most people wouldn't, so they hire me as a consultant to help them with it.

## Embodied Cognition instead of Investigative Programming

I consulted over 50 projects in great depth of legacy. We always tried to figure out, where to start. 

- "This is how it's done in your project, and this is the way in <the-desired-framework>."

or

- "This is how it's done in Symfony 2.8 and this is the way in Symfony 3.0."

Over and over again, just version numbers and the <the-desired-framework> changed. After few years I started to feel like dump copy-paster. I follow every new feature on Symfony, test it, verify it's usefulness, then destile 100 hours of my work to 3 hour lecture.
 
I'm lazy and this started to itch my mind. Is this the really education I want to encourage in the world? Well, majority of lecturers do exactly same work, well paid work. But is that reason to do it too?

I borrow a term from psychology - embodies cognition. It's something you don't have to remember, cause it's in you. It's like riding a bike. I don't know what words to use and where to find out how to ride a bike - I just know it, cause it's in  my internal reflexes.
 
Could something similar happend to upgrading applications? A single place that knows what to do and doesn't have to explain every programmer over and over again?

## Founding Fathers of Rector

That's how [Rector](https://github.com/rectorphp/rector) was born. At least the ideas.

It's not only about writing code. It's about discussing the idea, finding the right API that not only me but others would understand right away, it's about using other tools. It's about talking with people who already made AST tools, ask for advices and find possible pitfalls. It's about reporting issues and talking with people who maintain  projects you depend on.

### Is PHP Ecosystem is AST Ready?

All this was needed before, but PHP missed crucial parts:  

- parse PHP code to AST - done thanks to Nikic and his [nikic/PHP-Parser](https://github.com/nikic/PHP-Parser) 
- prototype that allows context aware PHP anylisis = "this variable is of this type" - done in 2017 by Ondřej Mirtes and his [PHPStan](/blog/2017/01/28/why-I-switched-scrutinizer-for-phpstan-and-you-should-too/)
- save changed AST back to PHP without any changes - [this feature was added in 2017](https://github.com/nikic/PHP-Parser/blob/master/doc/component/Pretty_printing.markdown#formatting-preserving-pretty-printing) to `nikic/php-parser` and becomes stable with version 4 
- simple coding standard tool, that would cleanup the code after Rector - done in 2017 by [EasyCodingStandard](https://github.com/Symplify/EasyCodingStandard)
- and prototype which would allow it

### FDD: Friendship-Driven-Development

I don't work for at any company so development of Rector doesn't solve my personal issues. That's how most project is born, like PHPStan to check Slevomat's code. That means I needed other motivation - when my frustration of wasted thousands human-hours was not enough. 
 
Here I'd like to thank [Petr Vacha](https://) for cowork weekend in Brno with in summer 2017, where it all started - in those times named as *Refactor*. You've been great friend for years and courage in times, when I needed it the most.
 
And [David Grudl](https://davidgrudl.com/), who gave me the motivation to dig deep and "just try it" when I felt desperate and useless always with lightness of Zen master. 

And also [Nikita Popov](http://nikic.github.com/), who patiently answered, taught me and fixed all my issues on `nikic/php-parser`. 

Without you, I would not make it here today.

