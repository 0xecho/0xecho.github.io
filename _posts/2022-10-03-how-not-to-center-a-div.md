---
title: How not to center a div
date: 2022-10-03 00:00:00 +0000
categories: [humor, css]
tags: [css, centering, div, fun, humor]
image:
  path: assets/2022-10/how-not-to-center-a-div.jpg
  width: 1000
  height: 400
  alt: a slightly tilted blue div with a yellow background
--- 

# How not to center a div

Ever struggled to center a div? Ever googled "how to center a div"? You probably have seen a lot of different ways to center a div. I even googled "how to center a div" and the first result was talking about 11+ ways to center a div. It is, without a doubt, one of the most common questions in web development. So, I decided to write a blog post about it. Some of the ways to center a div are good, some of them are ok, some of them are bad, some can be really funny.

I will do a quick overview of the different ways to center a div and hopefully you will learn something new. Also, please excuse the bad puns, I thought writing them on paper would be tearable.

## The goods

### Time to Flex

They say the day flex box was released, frontend developers everywhere jumped up and down in joy. I may not believe what they say, but I do believe in the power flex box gives us. We software engineers are in the trade of tradeoffs, but once every few years we get a tool that makes us question how we survived without it. Flex box is one of those tools. Flex box is a great tool to perform most of the layout tasks we need to do. It coincidentally also makes it easy to center a div.

To center a div with flex, you just need to add `display: flex` to the parent element and `justify-content: center` and `align-items: center` to the child element. That's it. You have centered a div. It is that easy.

```html
<div class="parent">
    <div class="child">I am centered</div>
</div>

.parent {
    display: flex;
}
.child {
    justify-content: center;
    align-items: center;
}
```

### A Grid-y approach

CSS Grid is the new kid on the block. It is the product of some absolute mad person, who came up with the idea of having a two-dimensional flex box. Flex box is great, and apart of that greatness comes the fact that it is dimensional. Lucky for us CSS Grid is also dimensional. It is a two dimesions, while flex box is one. And last time I checked, two is greater than one. I am not saying that flex box is bad, I am just saying that CSS Grid can be used in some cases where flex box can't, or can but with much more effort. 

Centering a div in a grid is not much different than centering a div in flex. You just need to add `display: grid` to the parent element and `place-items: center center` to the child element. `place-items` is a shorthand for `align-items` and `justify-items`. It is a great shorthand, and it also makes it easy to write (so yaaay for that).

```html
<div class="parent">
    <div class="child">I am centered</div>
</div>

.parent {
    display: grid;
}

.child {
    place-items: center center;
}
```

### Marginally better than the rest?

I am not sure if this is the best way to center a div, but it is definitely the easiest. If you sent the width of the current element to span the entire width of the parent element, you can then use an automatic margin to center it. 

```html
<div class="parent">
    <div class="child">I am centered</div>
</div>

.parent {
    width: 100%;
}

.child {
    width: 100%;
    margin: 0 auto;
}
```

## The okays

### Grandpa's table

80's kids will remember the good old days when they used tables to center things. It was a simpler time, there were no fancy ides, global warming was not a thing, cars couldn't yet fly, and tables were used to center things. While it may seem hacky, it is still one of the valid ways to center a div. What you essentially do is you make the parent element a table, and the child element a table cell. Then you use `vertical-align: middle` to center the child element. 

```html 
<div class="parent">
    <div class="child">I am centered</div>
</div>

.parent {
    display: table;
    margin: 0 auto;
}

.child {
    display: table-cell;
    vertical-align: middle;
}
```

### The poor man's flex

"I just want to center a text inside my div.", I hear you say you magnanimously attentive reader. Well, you can do that too. You can just use `text-align: center` on the child element. It will center all of the text inside the div. It is effectively the same as using centering a div.

```html
<div class="parent">
    <div class="child">I am centered</div>
</div>

.parent {
    text-align: center;
}
```

## The bads

### eManual

There comes a time in every developer's life when they need to make the decision to do as little thinking about a better solution as humanly possible. This is the time when you need to do the eManual. The eManual is a technique where you just set the width and height of the element to a fixed value, and then you set the margin to half of the width and height. It is a bad technique, a really really bad technique but it is a technique nonetheless. 

```html
<div class="parent">
    <div class="child">I am centered</div>
</div>

.parent {
    width: 200px;
    height: 200px;
    margin: 100px 100px;
}
```

### Center tag

Things come and go. Some things come back, some things never go away. The `<center>` tag is one of those things that never went away. It's been obsolete, deprecated, lost, forgotten ... you name it. It is a tag that used to be used to center its contents. Please don't use it. It is a bad idea. P.S. Even VS Code highlights it as red.

```html
<center>
    <div class="child">I am centered</div>
</center>
```

## The uglies

### The math whiz

I am not sure why anyone would ever use absolute positioning to center a div. It is a valid method, that works if you resize the window too. That probably is one of the worst/best ways to say it is responsive. But it is a bit too verbose for my taste, plus it is not very readable for first timers.

```html
<div class="parent">
    <div class="child">I am centered</div>
</div>

.parent {
    position: relative;
}

.child {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

## The weirds

### The eager beaver

I am not sure why anyone would ever use JavaScript to center a div. Maybe it is because they are eager to have just learned JavaScript and they want to use it everywhere. Maybe it is because they are lazy and they don't want to learn CSS. Maybe it is because they are just bad at CSS. Whatever the reason, I would advise seeking help before using JavaScript to center a div.

```html
<div class="parent">
    <div class="child">I am centered</div>
</div>

<script>
    const parent = document.querySelector('.parent');
    const child = document.querySelector('.child');

    child.style.position = 'absolute';
    child.style.top = (parent.offsetHeight - child.offsetHeight) / 2 + 'px';
    child.style.left = (parent.offsetWidth - child.offsetWidth) / 2 + 'px';
</script>
```

## The just plain wrongs

### Eyeballin' it

This is by FAR the best technique to center a div. Bravo, just bravo for whomwver came up with this method. How you may ask? It's plain simple. Just keep adding spaces .... until it looks centered enough.

```html
<div class="parent">
    <div class="child">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;I am centered</div>
</div>
```


## The Honorable Mentions

### Frameworks

The very essence of frameworks is to make your life easier. If someone has already solved a problem, why not use their solution? There are many frameworks that have a built to solve a lot of "problems" over the years, but we all secretly know, they only initially wanted their a div to be centered. 

Bootstrap has a class called `text-center`, `mx-auto` and `my-auto`, an entire library of classes that are used to center things.

## Conclusion

For a determined developer, there is always a way to center a div. But for a good developer, there is always one true way to center a div. And that way should be, always be, remain, and forever be their one and only way to center a div. I chose mine to be Flexbox. **What is yours?** Drop a comment below and let me know.
