---
title: "Auto-Downscaling Retina Images with Hugo"
date: 2022-09-26
---

## Retina Images

Many modern screens are capable of displaying images with so many dots per inch (DPI) that they meet or exceed the resolving power of the human eye. In other words, they produce detail that's so fine that you couldn't see any more even if it were present. 

Apple calls these [retina displays](https://en.wikipedia.org/wiki/Retina_display), and -- like Kleenex -- the name often gets used for other high DPI displays 

A "retina image" is high-resolution image that is designed to be displayed on a high-DPI screen. They are becoming very popular, even on the Web, because they look crisp and gorgeous on modern screens. For the purposes of this blog post, we're going to focus on images that are twice the standard resolution (2x). 

## A Naive Approach

The dumbest way to display retina images is to stick them in an `<img>` tag and tell the browser to show them at half size. Supposing, for example, that we have a 500 x 500 pixel image, we can render it at 2x simply by telling the browser to put it in a 250 x 250 box. 

```html
<img src="retina.jpeg" height="250" width="250" />
```
Or -- if we want to be slightly fancier -- we can just specify the width or height and tell the browser to figure out the rest. 

```css
img.retina-250 {
    width: 250px;
    height: auto;
}
```

```html
<img class="retina-250" src="retina.jpeg"  />
```

## Challenges

The first problem with this naive approach is that the size of the image must be known ahead of time; our `retina-500` class only works for images that are exactly 250 pixels wide.

The second problem is bandwidth. Retina images are twice as large *and* twice as wide, meaning that they are *four times* as large. 

```goat
   .---------------------.
   |                     |
2x |         Retina      |
   |    ^                |
   +----+-----.          |
   |          |          |
   | Standard +->        |
   |          |          |
   '----------+----------'
                  2x
```

And while retina displays are popular, not everyone uses them. So an awful lot of people who visit our web page are going to waste time and bandwidth on a lot of pixels they'll never see. That's a huge bummer. Can we do better?

## Enter srcset

Fortunately, the [srcset attribute](https://developer.mozilla.org/en-US/docs/Learn/HTML/Multimedia_and_embedding/Responsive_images) allows us to address the bandwidth issue by supplying multiple images and allowing the browser to choose which one to download.

Again, we're going to paper over a lot of complexity by just illustrating the simple case, in which we have only two versions of the image -- one at standard resolution, and one at 2x.

It (almost) couldn't be easier: we just supply an ordinary image and a high-DPI version, and tell the browser to use the latter for displays running at 2x scaling. 

```html
<img srcset="flower-low-dpi.jpeg, flower-high-dpi.jpeg 2x" />
```

Now the browser will download the image most appropriate for the display -- a low-resolution image for low-resolution displays, a high-resolution image for high-resolution displays. Problem solved?

## Automating Downscaling

As is so often the case with technology, we have solved one problem (inefficient use of bandwidth) by creating another: now we have to make *two versions of every image*. Are we going to forget sometimes? Probably. Is it pointless manual labor? Also yes. Let's get Hugo to do it for us.

Hugo has a feature called [Image Processing](https://gohugo.io/content-management/image-processing/) we can use to automate the generation of these downscaled versions. Here's how it will work.



```html
{{ $img2x := $.Page.Resources.GetMatch (.Get 0) }}

{{ $img1x := $img2x.Resize (print (div $img2x.Width 2) "x") }}

<img srcset="{{ $img1x.RelPermalink }}, {{ $img2x.RelPermalink }} 2x" 
   src="{{ $img2x.RelPermalink }}" 
   width="{{ (div $img2x.Width 2) }}"
   height="{{ (div $img2x.Height 2) }}"
   alt="{{ $img2x.Title }}"/>
```

## Auto Downscaling


