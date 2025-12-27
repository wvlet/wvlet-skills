---
name: airframe-rx-html
description: Builds interactive Web UI in Scala.js using airframe-rx-html. Use when creating reactive web components, DOM manipulation, event handling, or when the user mentions airframe-rx-html, Scala.js UI, or reactive web development.
---

# Building Web UI with airframe-rx-html

## Project Setup

**build.sbt:**
```scala
enablePlugins(ScalaJSPlugin)
libraryDependencies += "org.wvlet.airframe" %%% "airframe-rx-html" % "(version)"
```

## Entry Point

```scala
import scala.scalajs.js.annotation.*
import wvlet.airframe.rx.html.all.*

@JSExportTopLevel("MyApp")
object MyApp {
  @JSExport
  def main(): Unit = {
    div(
      h1("My App"),
      p("Welcome!")
    ).renderTo("app")  // Renders to <div id="app">
  }
}
```

## HTML Elements and Attributes

```scala
import wvlet.airframe.rx.html.all.*

div(cls -> "container",
  a(href -> "/page", "Link"),
  button(tpe -> "button", cls -> "btn", onclick -> { () => println("clicked") }, "Click"),
  input(tpe -> "text", placeholder -> "Enter text")
)
```

## Reactive State

```scala
import wvlet.airframe.rx.Rx

val count = Rx.variable(0)

div(
  button(onclick -> { () => count.update(_ + 1) }, "Increment"),
  count.map { c => span(s"Count: ${c}") }  // Auto-updates on change
)
```

## Custom RxElement

```scala
class Counter(initial: Int) extends RxElement {
  private val count = Rx.variable(initial)

  override def render: RxElement = div(
    button(onclick -> { () => count.update(_ - 1) }, "-"),
    count.map { c => span(c.toString) },
    button(onclick -> { () => count.update(_ + 1) }, "+")
  )

  // Lifecycle hooks (optional)
  override def beforeRender: Unit = {}
  override def onMount(node: Any): Unit = {}
  override def beforeUnmount: Unit = {}
}
```

## RxComponent (Layout Wrapper)

```scala
class Card(title: String) extends RxComponent {
  override def render(content: RxElement): RxElement =
    div(cls -> "card",
      div(cls -> "card-header", title),
      div(cls -> "card-body", content)
    )
}

// Usage: new Card("Title")(p("content"), button("action"))
```

## Collections and Conditionals

```scala
val items = Rx.variable(Seq("A", "B", "C"))
val show = Rx.variable(true)

div(
  items.map { list => ul(list.map(li(_))) },
  show.map { if (_) div("Visible") else span() }
)
```

## SVG

```scala
import wvlet.airframe.rx.html.svgTags.*
import wvlet.airframe.rx.html.svgAttrs.*

svg(width -> "200", height -> "200",
  circle(cx -> "100", cy -> "100", r -> "50", fill -> "#3366CC")
)
```
