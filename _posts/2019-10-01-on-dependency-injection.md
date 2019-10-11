---
layout: post
title: On Dependency Injection
comments: false
---

Lots of development frameworks now treats *Dependency Injection* as a first-class citizen in their implementations (for example, dotnet core's ASP and Java's Spring...). Therefore, **knowing how to use it** is guaranteed at lots of software engineers, but **knowing why and when to use it** is what made me write this post as all of my posts :P.
<!--more-->

Dependency Injection **is not a design pattern**, it's a technique to apply the design pattern that it was intended to do, and it's **Dependency Inversion**. And actually, understanding Dependency Inversion makes the former a lot easier to understand and even apply without using any third-party libraries (IoC containers) or even in a framework that doesn't use it.

## Problem

Let's take an example, suppose we have a class called `Painter`, this class has a method called `Paint()` which its responsibility is to paint something on the screen:

```c#
public class Painter {
    public void Paint(string whatToPaint){
        //some logic
        //...

        Screen screen = new Screen();
        screen.WriteOut(whatToPaint);
    }
}
```

As we can see, the class `Painter` uses *(depends)* on class `Screen` to be able to draw on the screen, all looks good. Now, let's suppose that the product owner walks in and says: I want our software to be able to paint on a printer or a screen based on some configurations... so it's time to hit the keyboard typing.

At first glance, one would suggest to create a `PrinterPainter` class along with the previous `ScreenPainter` class to achieve the requirements. But if the logic of the painting is changed, then we'll have two places to adjust (bugs here we come), further more if new requirements where added, for example the need for a new class `ImageFilePainter` that draws what to print on a `jpg` file, then we'll have 3 places to adjust in the future... not good.

## Dependency Inversion to the Rescue

All of our painting utilities paint out something (`screen.WriteOut()`), so let's create an interface that defines that painting behavior:

```c#
public interface IPaintingUtility {
    void WriteOut(string _);
}
```

We make our utility class inherit that behavior: 

```c#
public class Screen : IPaintingUtility {
    public void WriteOut(string whatToWrite){
        //screen painting logic
    }
}

public class Printer : IPaintingUtility {
    public void WriteOut(string whatToWrite){
        //printer painting logic
    }
}

public class JPGImage : IPaintingUtility {
    public void WriteOut(string whatToWrite){
        //jpg image painting logic
    }
}
```

Now, we make our `Painter` depends on our interface `IPaintingUtility` in the following manner:

```c#
public class Painter{
    private IPaintingUtility paintingUtility;

    public Painter(IPaintingUtility paintingUtility){
        this.paintingUtility = paintingUtility;
    }

    public void Paint(string whatToPaint){
        //some logic
        //...

        this.paintingUtility.WriteOut(whatToPaint);
    }
}
```

As we see, `Painter` is no longer depending on the concrete implementation of the painting utility, it depends on the interface, so the dependency is **inverted**. In addition, it's no longer responsible for creating the painting utility, it gets **injected** to it.

```c#
public class Program {
    public static void main(...args){
        IPaintingUtility paintingUtility = new Screen();

        Painter painter = new Painter(paintingUtility); // inject the paintingUtility to Painter
        painter.paint();
    }
}
```

There are a lot of situations on our days of coding that can leverage the Dependency Inversion principle, for example creating a logger that logs to the console in the dev environment while it logs to a file in staging or doesn't log at all in production.

The previous technique is called *Constructor-Based Dependency Injection*, and there are a lot more techniques, and in case of heavy use of Dependency Injection, there is something called Inversion of Control (IoC) containers that are responsible of initiating your objects and their dependency graphs, let's leave that for another post.

But one thing we should consider, using this principle requires a good amount of requirements analyzing because using it extensively you will end up with an inverted software hell, specially that it can be used simply every where. Just simply ask yourself, Am I going to change my logger any soon? be careful of the Second System Syndrome, and know that premature optimization is the root of all evil.