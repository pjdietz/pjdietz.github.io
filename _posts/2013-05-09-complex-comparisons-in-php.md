---
layout: post
title: Complex Comparisons in PHP
---

I had a project recently where I needed to do some custom array sorting using usort(). Nothing tricky about writing a custom sort function or two, but what do you do when you need several, and you don’t know ahead of time which sort functions you’ll need?

For example, say you have a dataset that you’d like to sort in PHP, and you want to leave it to the user (or configuration file, etc.) to determine the sort order. One configuration may order the data by firstName, then age, then favorite color, while another may order by favorite color, then firstName, etc.

I came up with a solution that allows you to bake an array of compare functions into one single compare function. It looks like this:

{% gist 5292681 compare.php %}

## Using It

To use this, start by defining some compare functions. If you’re not familiar with how to do this, see [the usort() page in the PHP manual](http://php.net/manual/en/function.usort.php). The basic idea is that you provide a function that takes two parameter and returns 1 if the first parameter should sort before the second, -1 if the second should short before the first. 0 indicates they’re the same.

```php
<?php
// From the PHP manual.
function cmp($a, $b)
{
    if ($a == $b) {
        return 0;
    }
    return ($a < $b) ? -1 : 1;
}
```

Once you have a few defined, create an array containing references to the functions or strings containing the names of the functions. (PHP considers either to be a “callable”). Pass this array to `makeComplexSortFunction()`, and use the returned callable in `usort()`.

Here’s an example script:

{% gist 5292681 demo.php %}

Happy sorting!
