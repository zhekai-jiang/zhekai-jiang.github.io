---
layout: page
title: Generalizing Transformations
---
<script>
  document.getElementById("page-link-Linear Algebra").classList.add("active")
</script>

## Introduction

Let us recall that in the review class for your first midterm exam, you were asked to come up with intuitive ideas from the context of your everyday life that capture the concept of bases. In the notes posted on myCourses, the "subspace" could be  "all burritos" and the "basis" could be rice, bean, tortilla, etc. I remember that in the exact same lecture two years ago in Fall 2019, we were also talking a lot about food ingredients here! For me personally, this is indeed an impressive and inspiring example that shows how those concepts in linear algebra can be applied in a context much wider than the vectors and matrices in the forms we are familiar with. Jing's post last week gave a brief introduction to field and vector space, and in this write-up, I will expand by giving some more examples about how we could generalize the concept of transformations, operators, and algebra for functions and even languages.

## Differential Operators

In your calculus course, you must have seen all the different notations for derivative. In particular, your instructor must have taught you a way to view $$\frac{\text{d} y}{\text{d} x}$$ as an *operator* $$\frac{\text{d}}{\text{d} x}$$ applied on the function $$y$$. To express this idea more explicitly, we could use $$D$$, or $$D_x$$, to denote this operator and write $$Dy$$, or $$D_x y$$, to represent the derivative of $$y$$. In the language of MATH 133, we can think of $$D$$ to *transform* the function $$y$$ to its derivative, i.e. this operator represents a transformation.

Now, again, since we are in MATH 133, can we say this is a *linear transformation*? It might look strange to talk about this for functions instead of vectors in $$\mathbb{R}^n$$, but we know that $$D(cf) = \frac{\text{d} (cf)}{\text{d} x} = c \frac{\text{d} f}{\text{d} x} = cDf$$ for all $$c \in \mathbb{R}$$ and $$f : \mathbb{R} \rightarrow \mathbb{R}$$, and $$D(f + g) = \frac{\text{d} }{\text{d} x} (f + g) = \frac{\text{d} f}{\text{d} x} + \frac{\text{d} g}{\text{d} x} = Df + Dg$$ for all $$f, g : \mathbb{R} \rightarrow \mathbb{R}$$. This is known as the *linearity* of differentiation. So, adopting a more generalized definition, we could say this is a linear transformation and the differential operator $$D$$ is a *linear operator*.

This linearity allows us to take derivatives of polynomial functions easily and also to perform what we are familiar with in linear algebra! For example, we can *compose* differential operators to make new operators like $$D \circ D$$, or more commonly $$D^2$$, to denote applying differentiation twice, i.e., taking the second derivative. We can go even further and use $$(D^2 + D)y$$ to denote $$D^2y + Dy$$, i.e., $$\frac{\text{d}^2 y}{\text{d} x^2} + \frac{\text{d} y}{\text{d} x}$$, or $$(D^2 + D + 1)y$$ to denote $$D^2y + Dy + y$$, i.e., $$\frac{\text{d}^2 y}{\text{d} x^2} + \frac{\text{d} y}{\text{d} x} + y$$. As we stack more and more differential operators, we even define polynomials for operators. For example, we can define $$p(D) = D^2 + D + 1$$ and the previous expression becomes $$p(D)y$$—you will see a lot of this when you learn differential equations.

## Functional Programming and Higher-Order Functions

In MATH 133, for a transformation $$T$$ that maps a vector in $$\mathbb{R}^n$$ to a vector in $$\mathbb{R}^m$$, we often use the notation $$T: \mathbb{R}^n \rightarrow \mathbb{R}^m$$. In what we call functional programming in computer science, we have similar notations if we think of $$\mathbb{R}^n$$ and $$\mathbb{R}^m$$ as *types* of data and $$T$$ as a function. Now let us try to make a similar notation for the differentiation operator $$D$$ in the previous section. $$D$$ transforms a function $$f$$ to its derivative, so it should be something like $$D: \texttt{<function>}\rightarrow\texttt{<function>}$$. But note that a function we have in calculus is itself a mapping from $$\mathbb{R}$$ to $$\mathbb{R}$$, so we can think of $$\mathbb{R} \rightarrow \mathbb{R}$$ as the type of such a function, and $$D: (\mathbb{R} \rightarrow \mathbb{R}) \rightarrow (\mathbb{R} \rightarrow \mathbb{R})$$!

This is really a crazy notation with three arrows, but this is an important concept in what we call functional programming: a function can be treated the same way as for data. In particular, a function can be taken as an input parameter of a function and/or be returned as output. Such a function that uses, manipulates, or produces another function is called a *higher-order function*. This could be useful when, for example, we wish to do some operations on some data, but the exact operation is unknown or can vary. In such cases, we can make an argument a function and apply this function on the data in our higher-order function. In this way, we can reuse this exact same higher-order function when the function (as the argument) vary, without having to modify the implementation inside the higher-order function. There are similar ideas in non-functional programming languages such as Java, Python, and JavaScript, where there could also be functions (or methods) used as parameters.

## Kleene Algebra

Another example I would like to give is the Kleene Algebra, which is an algebra defined on *languages* (more specifically, *regular expressions*) and is applied in the theory of computation and programming languages. Here, we will formally define a *language* as a set of all *words* (each word is essentially string of characters) that are allowed in the language (such a strict and formal definition is used in linguistics sometimes and theory of computation, but it may not really work so well for our natural languages). Then, we can define the following operations:

$$+$$: $$R+S$$ represents a new language with all words in $$R$$ and all words in $$S$$.

$$\cdot$$: $$R \cdot S$$ represents a new language with all words obtained by concatenating any word in $$R$$ and any word in $$S$$.

$$^*$$: $$R^*$$ represents a new language with all words obtained by taking an arbitrary number of words in $$R$$ (which can be repeated) and concatenating them together.

There are also two *constants*: $$\varnothing$$ for an empty language (not containing any word) and $$\varepsilon$$ for an empty word (not containing any character).

With these rules, we can come up with a lot of *equations* for this algebra:

$$R + \varnothing = \varnothing + R = R$$, similar to adding $$0$$ to a number;

$$R + S = S + R$$, similar to the commutativity of addition;

$$R + (S + T) = (R + S) + T$$, similar to the associativity of addition;

$$R + R = R$$, which is not quite similar as our usual addition;

$$R \cdot \varnothing = \varnothing \cdot R = \varnothing$$, similar to multiplying a number with 0;

$$R \cdot \varepsilon = \varepsilon \cdot R = R$$, similar to multiplying a number with 1;

$$R \cdot (S + T) = R \cdot S + R \cdot T$$, similar to the distributive law;

and so much more! The whole theory is very deep and interesting. As you can imagine, we could even define linear equations, vectors, matrices, and a whole new linear algebra here!

## References

This write-up again points to a lot about more advanced linear algebra and abstract algebra. If you are interested in the linear algebra on everything like food ingredients (in your review class), colour space (an earlier post of mine), etc., be sure to check out more advanced courses about linear algebra and abstract algebra like MATH 223 Linear Algebra, MATH 247 Honours Applied Linear Algebra, MATH 236 Algebra 2, and MATH 251 Honours Algebra 2.

Ideas of differential operators as well as linear dynamical systems which you will see soon, are in fact applied in solving ordinary differential equations, which are indeed seemingly unrelated to linear algebra at all! You will see them in related courses like MATH 315 Ordinary Differential Equations or MATH 263 Ordinary Differential Equations for Engineers.

For computer science and software engineering students, you will see the concept of functional programming and all about higher-order functions in COMP 302 Programming Languages and Paradigms, and I hope you will gain great enlightenment as I did! COMP 330 Theory of Computation also covers interesting topics about languages and computation theory, including the theory of regular expressions and Kleene algebra I mentioned. COMP 330 will give a perspective you may have never had on computer science! By the way, for lovers of formal logic, computing theory, and perhaps some philosophical aspects of human mind and machine intelligence, I would strongly recommend a book which was recommended by a professor of mine: *Goödel, Escher, Bach: an Eternal Golden Braid* by Douglas R. Hofstadter.