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

## Attributes vs. Properties
HTML is a markup language that gets converted from a string into the DOM in JavaScript. When we query the DOM we are really traversing the JavaScript tree for a Node where we can run even more methods or change values. When we change the value of a single Node we are really changing the properties

```typescript
const input = document.querySelector('input');

input['value'] = 'newValue';
```
this is different from attributes which are the initial values of our Node when it's transformed into the DOM

```typescript
<input value="initialValue">
```
On the server if we change any of these properties then the values won't be reflected as HTML when we convert the server's version of the DOM into a string. For most of the basic attributes any changes to these properties are reflected correctly for example the `src` property.
```typescript
<img [src]="http://example.com/image.png">
```
In Angular we are also able to bind to the attributes using `attr`
```typescript
<img [attr.src]="http://example.com/image.png">
```
Keep your directives stateless as much as possible. For stateful directives, you may need to provide an attribute that reflects the corresponding property with an initial string value such as `src` in img tag. For our native element the `src` attribute is reflected as the `src` property of the element type `HTMLImageElement`.

# Flickering and Lazy Routes
Set
`{initialNavigation: 'enabled'}` 
in 
`RouterModule.forRoot([], {initialNavigation: 'enabled'})`

Here https://angular.io/api/router/ExtraOptions#initialNavigation
Or here from the definition file:
``` javascript
/**
 * @whatItDoes Represents an option to configure when the initial navigation is performed.
 *
 * @description
 * * 'enabled' - the initial navigation starts before the root component is created.
 * The bootstrap is blocked until the initial navigation is complete.
 * * 'disabled' - the initial navigation is not performed. The location listener is set up before
 * the root component gets created.
 * * 'legacy_enabled'- the initial navigation starts after the root component has been created.
 * The bootstrap is not blocked until the initial navigation is complete. @deprecated
 * * 'legacy_disabled'- the initial navigation is not performed. The location listener is set up
 * after @deprecated
 * the root component gets created.
 * * `true` - same as 'legacy_enabled'. @deprecated since v4
 * * `false` - same as 'legacy_disabled'. @deprecated since v4
 *
 * The 'enabled' option should be used for applications unless there is a reason to have
 * more control over when the router starts its initial navigation due to some complex
 * initialization logic. In this case, 'disabled' should be used.
 *
 * The 'legacy_enabled' and 'legacy_disabled' should not be used for new applications.
 *
 * @experimental
 */
export declare type InitialNavigation = true | false | 'enabled' | 'disabled' | 'legacy_enabled' | 'legacy_disabled';
```

## Jquery & Angular
__DO NOT USE IT__ It's that simple. If you need to query the DOM, set a ```viewChild/viewChildren```. Need to change styles, use ```ngClass/ngStyle```. Need a reference to an element, use ```ElementRef```. Jquery always introduces more problems than it fixes. This does not include libraries such as D3.
