# Common gotchas with Angular Universal

## Universal, what is it?
TL;DR: Server-side rendering of your Angular project!

Angular Universal takes your Angular project and renders it on the server. By running your component and template compilation on the server, user experience faster load times. Using Angular Universal also allows you to run API calls for data fetching on the server for a quick cache hit response. Server-side rendering is great for SEO and Link Previews!

## The Setup
Most of Angular's problems are due to the setup. That is an entirely different article all together and you for most projects you can use a [starter repo](https://github.com/AngularClass/angular-starter) or [cli](https://github.com/angular/angular-cli) to avoid any setup issues.

## Window Issues
Due to the application being rendered on the server, you lose access to all browser APIs. This includes access to `window` object and the DOM. Generally not haven't access to browser APIs isn't an issue as long as you aren't directly querying the DOM using methods like:
```javascript
document.getElementById()
document.querySelector()
```
Instead, use Angular directives like ```viewChild/viewChildren``` to get access to the element:

```javascript
import { Component, ViewChild } from '@angular/core';

@Component({
  template: `
  <div>
    <p #myChild>Pick me!</p>
  </div>
  `
})
export class PickMe {
  @ViewChild('myChild') pickMe;
}
```
_You should never wrap code in a long setTimeout just to avoid slow your rendering on the server_

Always make sure any external libraries are Universal ready as they can break on the server if they use any browser API.

## Running Browser/Server Only Code
Sometimes you need to limit some code from running unless they're in the correct environment. In Angular we have ways to know when your component is being rendered on the server or the browser and only executing code when needed via `isPlatformBrowser` or `isPlatformServer`:
```typescript
import { PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser, isPlatformServer } from '@angular/common';
 
// ...
  constructor(@Inject(PLATFORM_ID) private platformId: string) {
    // ...
  }
 
  ngOnInit() {
    if (isPlatformBrowser(this.platformId)) {
      // Client only code.
      // ...
    }

    if (isPlatformServer(this.platformId)) {
      // Server only code.
      // ...
    }

  }
//...
```

<!--
Keep in mind API calls in your components will be called on the server and in the browser. It is highly recommended to either wait for the component to render on the browser to make the call or simply cache the api output data on the server. As expected, making multiple calls to an api makes your app sluggish on load.
-->

## ElementRef.nativeElement
Don't manipulate the `ElementRef.nativeElement` directly. We should use the `Renderer2` to ensure that in any environment we're able to change our view.
```typescript
// ...
  constructor(element: ElementRef, renderer: Renderer2) {
    renderer.setStyle(element.nativeElement, 'font-size', 'x-large');
  }
// ...
```

## Jquery & Angular
__DO NOT USE IT__ It's that simple. If you need to query the DOM, set a ```viewChild/viewChildren```. Need to change styles, use ```ngClass/ngStyle```. Need a reference to an element, use ```ElementRef```. Jquery always introduces more problems than it fixes. This does not include libraries such as D3.
