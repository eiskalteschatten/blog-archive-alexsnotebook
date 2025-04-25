Since Angular’s new [Signals API](https://angular.dev/guide/signals) became stable with the release of version 17, I have been a little confused about when to use them versus standard class variables in components and services. To be honest, I just hadn’t gotten around to reading the documentation properly and had therefore only seen them on the periphery of my work. That changed yesterday, though, when I thought I might have a good use case for them and so I finally read the documentation.

However, they weren’t exactly what I was expecting. In my naïve understanding of what they were, I thought they would basically be a drop-in replacement for [RxJS Observables](https://rxjs.dev/guide/observable). While you can certainly use them instead of Observables in some cases, the more striking thing for me was that they seem to be more oriented towards replacing class variables in components and services.

So, I asked Microsoft Copilot about when to use class variables vs Signals. This was part of its answer:

> **When to Use Angular Signals**
> 
> -   **Automatic Reactivity:** Signals are ideal for state that directly feeds into your component’s templates. When state changes, signals automatically update any computed values or dependent views without having to hook into Angular’s change detection manually. This means less boilerplate and a more intuitive reactivity model. *Example: When you have a counter or any state where you want the UI to instantly reflect the latest value, a signal is the natural choice.*
> -   **Computed and Derived State:** If your component has derived values that depend on one or more pieces of state, signals let you define computed signals that recalculate as soon as their dependencies change. This pattern reduces the need for manual subscriptions or extra logic to update dependent data. *Example: In a shopping cart, recalculating the total price automatically based on item signals makes the code both elegant and error-free.*
> -   **Simplifying Change Detection:** Signals reduce the overhead of Angular’s change detection mechanism by providing fine-grained reactivity. Rather than re-evaluating large portions of your component tree, only those parts bound to a signal update when the signal changes. This is particularly useful in performance-sensitive applications.
> 
> **When to Use Normal Class Variables**
> 
> -   **Static or Low-Churn Data:** If the data is either static or not tightly bound to the template (i.e., it doesn’t need to trigger an automatic view update), normal class variables are sufficient. They work well for values that don’t change frequently, or when you’re handling state using other reactive libraries like RxJS for asynchronous operations.
> -   **Legacy Code and Explicit Control:** In applications already using class variables—with change detection either running manually or via zone.js—it might not be necessary to refactor everything over to signals. Class variables can give you the explicit control needed in scenarios where you manage state changes imperatively.
> -   **Complex Asynchronous Streams:** For handling advanced asynchronous workflows (say, with operators like `map`, `filter`, or complex subscription logic), Observables (often stored as class variables or service members) might be a better fit than signals. Signals are synchronous by nature, so if your state management involves a lot of asynchronous behavior, you may continue to lean on established patterns with normal variables and RxJS.
> 
> \[…\]
> 
> **Summary**
> 
> -   **Angular Signals:** Best used when you need **fine-grained, automatic reactivity** for state that’s frequently updated and directly impacts the UI. They shine in contexts where derived or computed state is beneficial, simplifying the logic and reducing the need for manual change detection.
> -   **Normal Class Variables:** Ideal for **static or infrequently updated state**, or when you need **explicit control** over change detection, or are already integrated with other reactive patterns (such as full RxJS flows for handling asynchronous data).
> 
> Ultimately, the decision hinges on the nature and complexity of your state management needs. If you’re building a component where reactivity is a priority, signals can streamline your code, but for scenarios that call for explicit control or complex asynchronous orchestration, traditional variables (often in combination with RxJS Observables) remain robust and reliable.
> 
> [Microsoft Copilot](https://copilot.microsoft.com/shares/mYg3gptgbpf4sZ8KX8uUu)

Say what you may about AI, but this does seem to make a lot of sense to me. I will, of course, have to test it out in my daily development and run some tests on our frontend to see if I can establish any performance gains since we deal with large quantities of data and data manipulation in our components and services.

Beyond the explanation, Copilot also included some code examples for components and services. Having them in context is nice since the code in [Angular’s documentation](https://angular.dev/guide/signals) is isolated and provides no context whatsoever.

**In a component:**

```typescript
import { Component, signal } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div>Count: {{ count() }}</div>
    <button (click)="increment()">Increment</button>
  `
})
export class CounterComponent {
  count = signal(0);

  increment() {
    this.count.update((value) => value + 1);
  }
}
```

**In a service:**

```typescript
import { Injectable, signal } from '@angular/core';

@Injectable({ providedIn: 'root' })
export class CartService {
  // A signal to hold the shopping cart items
  items = signal<string[]>([]);

  addItem(item: string) {
    this.items.update((currentItems) => [...currentItems, item]);
  }
}
```

I hope they will bring the promised performance boost when manipulating data, but testing will certainly be necessary. I will also have to play around with them to see in which contexts they work best.

I’ve included most of Copilot’s answer in this post, but I still recommend taking a look at the original answer here: [https://copilot.microsoft.com/shares/mYg3gptgbpf4sZ8KX8uUu](https://copilot.microsoft.com/shares/mYg3gptgbpf4sZ8KX8uUu)