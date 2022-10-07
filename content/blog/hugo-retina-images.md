---
title: "Auto-Downscaling Retina Images with Hugo"
date: 2022-10-07
---

## Retina Images

Many modern screens are capable of displaying images with so many dots per inch (DPI) that they meet or exceed the resolving power of the human eye. In other words, they produce detail that's so fine that you couldn't see any more even if it were present. 

Apple calls these [retina displays](https://en.wikipedia.org/wiki/Retina_display), and the name often gets used for other high DPI displays. 

A "retina image" is high-resolution image that is designed to be displayed on a high-DPI screen. They are becoming very popular, even on the Web, because they look crisp and gorgeous on modern screens. Image scaling and DPI is a complex topic; for the purposes of this blog post, we're going to focus on images that are twice the standard resolution (2x). 

## A Naive Approach

The dumbest way to display retina images at high DPI is to stick them in an `<img>` tag and tell the browser to show them at half size. Supposing, for example, that we have a 500 x 500 pixel image, we can render it at 2x simply by telling the browser to put it in a 250 x 250 box. 

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

This gets the job done: Retina and non-Retina screens alike will show the image at the same size, but Retina screens will show it at high DPI. 

## Challenges

So, why don't we just do this? Well, the first problem with this naive approach is that the size of the image must be known ahead of time; our `retina-500` class only works for images that are exactly 250 pixels wide.

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

1. We will add each Retina image to the page as a [page resource](https://gohugo.io/content-management/page-resources/).
2. We will use a custom Hugo shortcode called `retina-img` to refer to the Retina image in the body of page.
3. Hugo will generate multiple copies of the image for us and write HTML that inserts them along with the requisite metadata.

### The Post

Here's what a post with Retina images might look like. Notice the image metadata in the page's `resources` block, and the `retina-img` shortcode that brings the rendered image into the page.

```yaml
---
title: "Fun with Fractals"
resources:
  - name: "mandelbrot"
    src: "mandlebrot.png"
    title: "An image showing the Mandelbrot Set fractal with a rainbow palette."
  - name: "julia"
    src: "julia-set.jpeg"
    title: "The Julia Set fractal image, rendered with a rainbow palette."
---

Let's talk about the Mandelbrot Set.

{{</* retina-img "mandelbrot" */>}}

And the Julia Set, too.

{{</* retina-img "julia" */>}}
```

### The Shortcode

Now, `retina-img` isn't part of Hugo; we'll have to build it ourselves. This shortcode goes into `layouts/shortcodes/retina-img.html`:

```html
{{/* retina-img.html: shortcode for creating Retina images */ }}

{{/* Look up the page resource given in the shortcode's first argument */}}
{{ $img2x := $.Page.Resources.GetMatch (.Get 0) }}

{{/* Create a downsampled verrsion of the image */}}
{{ $img1x := $img2x.Resize (print (div $img2x.Width 2) "x") }}

<img srcset="{{ $img1x.RelPermalink }}, {{ $img2x.RelPermalink }} 2x" 
   src="{{ $img2x.RelPermalink }}" 
   width="{{ (div $img2x.Width 2) }}"
   height="{{ (div $img2x.Height 2) }}"
   alt="{{ $img2x.Title }}"/>
```

Notes:

- The `src` attribute is still present, in case you have to support very old browsers that don't understand `srcset`.
- We place `width` and `height` on the `img` element, which allows the browser to allocate the right amount of space for the image before it has downloaded any of the image's content. This, in turn, reduces the number of layout passes, decreases page layout instability on load, and increases page load speed. It's usually a chore to add these, but Hugo can do it for us!
- The `alt` attribute supplies alternate text for the image; this shortcode is written 

## Fin.

Your Retina images are now sent at the appropriate scale for your users' displays, and all have precomputed widths and heights as well as proper `alt` tags. Happy website creating!

