# CSS Image Orientation Lands in Firefox 26

It’s never been easier to share your digital photos on the web, thanks to modern
smartphones and digital cameras. For web developers, however, dealing with user-generated
digital images can be far from simple. A significant mismatch exists between the
web, where the orientation of an image is determined by how its pixels are stored,
and digital photography, which increasingly relies on metadata to mark the image
as rotated, storing the pixels in the same orientation no matter what.

In practice, this means that the user ends up seeing this:

![With EXIF][1]

When they should’ve seen this:

![Correct orientation][2]

In the real world, the images in question are JPEG images, and the metadata is
the EXIF orientation tag. Current releases of Firefox do not support EXIF orientation.
Chrome and Safari support it when the image is the top-level document, but not
in HTML documents, which means web developers who want to avoid displaying their
users’ vacation pictures sideways have had to choose from several workarounds,
none of which are desirable. [Rotating the image using canvas][3] is inefficient.
[Applying a CSS transform to rotate the image][4] interacts with layout in tricky
ways. And [server][5]–[side][6] [solutions][7] require additional work from
developers and don’t work with images hosted outside of the developer’s control.

Edit: It turns out there _is_ one browser that does respect the EXIF orientation
tag by default in HTML documents: Mobile Safari. (And by extension other browsers
on iOS, like Chrome, which use the same underlying engine.) This is surprising,
since this isn’t standards-compliant behavior, and doesn’t match what the
desktop versions of these browsers do.

## CSS To The Rescue

These problems are all solved by the new CSS [image-orientation property][8],
which is now supported by Firefox 26. When the style `image-orientation: from-image`
is applied to a JPEG image, the browser will take its EXIF orientation tag into
account when performing layout and rendering. This means smartphone and digital
camera images can now be displayed in their proper orientation natively by the
browser, just by adding a single CSS property!

Here’s the first image in this article again, but this time with `image-orientation`
applied. If you’re using Firefox 26, you’ll see this image rendered as it was
intended, with the sky on top and the ground on the bottom.

    <img style="image-orientation: from-image" src="fountain.jpg">

![With EXIF][1]

Another way to use `image-orientation` is to specify an orientation directly.
You can provide an angle value to rotate an image by any multiple of 90°:

    image-orientation: 270deg;

This can optionally be followed by a horizontal flip using this syntax:

    image-orientation: 270deg flip;


This is particularly nice since the order the rotation and flip are applied
matches the order they’re written. You can also use `flip` by itself to mean
the same thing as `0deg flip`.

When you specify an orientation manually, `image-orientation` gives CSS the
same capabilities as the EXIF orientation tag. You can use this to manually
correct JPEGs lacking EXIF data, or to reorient PNGs and other types of images
that can’t include orientation information.

The `image-orientation` property is inherited, which is useful when you want to
apply an orientation to images in generated content pseudo-elements like `:before`
and `:after`. It’s not possible to style such images directly, but since they
inherit styles from the element to which they’re attached, you can still use
`image-orientation`.

    <style>
      p:before { content: url(bullet.png); }
      p.ninety { image-orientation: 90deg; }
    </style>

    <p>A generated content image with the default orientation.</p>
    <p class="ninety">A generated content image rotated 90°.</p>


* A generated content image with the default orientation.
* A generated content image rotated 90°.

## Gotchas

All of this is great, but there are a few rough spots which you should be aware
of when using `image-orientation`. The most obvious: right now, it’s only
supported in Firefox 26, though this will hopefully change quickly. Even as it
becomes available in more and more browsers, though, there will still be things
to watch out for.

As long as `image-orientation` isn’t universally supported, you’ll have to be
cautious about its effects on layout. Rotating an image by some angles will
cause its size to change if it isn’t square, because its width and height will
be swapped. This can in turn cause the surrounding layout to change. Most often,
`image-orientation` will be applied to user-provided images which already have
unpredictable sizes, so it’s likely that this won’t break your layout. Still,
it’s something to consider.

A more subtle point is the user experience for large images being loaded over a
slower connection. Users are used to such images appearing starting at the top.
However, using `image-orientation` can mean that the first pixels the browser
decodes will appear on the side or the bottom of the image! If the image is
large enough to extend off the bottom of the user’s browser window, it may be
less obvious than in the past that an image is downloading. This issue should
be taken into account to ensure a good user experience.

Finally, it’s important to be aware that `image-orientation` only applies to
content images. This means it can’t be used with other CSS properties that accept
image values like `background-image` or `list-style-image` — it’s only for `<img>`
tags and generated content.

## Try It Today!

You can try `image-orientation` today in [Firefox Nightly][9]. If you prefer not
to live quite on the bleeding edge, you’ll find it in [Firefox Aurora][10]
starting September 17.

At Mozilla, we’ve already found the property very useful internally. Firefox now
displays image documents with EXIF orientation taken into account. This means that
navigating directly to a JPEG image finally produces the same results in Firefox,
Chrome, and Safari. An upcoming version of Thunderbird will also take advantage
of `image-orientation` to display images in email attachments as they were intended
to be viewed, and we expect to make use of it in the photo gallery on Firefox OS
as well.

As `image-orientation` becomes more ubiquitous I hope that it will make life a
little easier for web developers and users alike. I look forward to the day when
tilting my head to look at someone’s misrotated smartphone pic is a thing of the past!

   [1]: img/fountain-with-exif.jpg
   [2]: img/fountain.jpg
   [3]: http://stackoverflow.com/questions/7584794/accessing-jpeg-exif-rotation-data-in-javascript-on-the-client-side
   [4]: http://stackoverflow.com/questions/10810496/adjusting-parent-element-of-css-transformed-image-to-fit-twitter-bootstrap
   [5]: http://php.net/manual/en/book.exif.php
   [6]: http://exifr.rubyforge.org/api/
   [7]: http://drupalcontrib.org/api/drupal/contributions!imagecache_actions!autorotate!imagecache_autorotate.module/function/imagecache_autorotate_image/6
   [8]: http://dev.w3.org/csswg/css-images/#the-image-orientation
   [9]: http://nightly.mozilla.org/
   [10]: http://www.mozilla.org/en-US/firefox/aurora/
