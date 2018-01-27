---
layout: post
title: "Julia sets in Julia"
excerpt: "ploting Julia sets in Julia during the 231st AAS"
modified: 2018-01-12
tags: [julia, julia-lang, julia sets, complex maths]
comments: true
image:
  feature: gsoc_post.jpg
  credit: Hubble Space Telescope
---

My first experience using the Julia language was great. Especially because it
was during the 231st AAS hack day ;)

My hack was pretty simple: make pretty pictures of Julia sets in Julia :)

According to Wikipedia, a Julia set of a complex-valued entire function $$f$$, denoted
as $$J(f)$$, is <i>the smallest closed set containing at least three points which is
completely invariant under</i> $$f$$ (if you clearly understand what that means,
please explain it to me bellow in the comments).

Julia sets can be used to create those pretty pictures:
<br>
<center><img src="../images/galaxy_julia_set.gif" style="width:237px;height:320px;"></center>
commonly known as fractals.

The code to create this "galaxy" Julia set is available on my <a href="https://github.com/mirca/hackaas231">
hack-aas repository</a>.

A very nice thing about Julia is that you can use it on Jupyter notebooks
(thanks goes to Lia Corrales for telling me that), which I think makes life
a lot easier for Python users.

That's all for now folks, maybe in future I will write something else in Julia ;-)
