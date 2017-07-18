# Common dev mistakes with Angular Universal

## Universal, what is it?
TL;DR: Server side rendering of your Angular project!

Universal takes your Angular 2 project and renders it on the server. By running your component and template compilation on the server, user experience faster load times. Using AoT(Ahead of time) compilation also allows you to run api calls for data fetching. This is great for SEO!

### The setup
Universal requires you define what pages are server rendered.

### Window Issues
Due to the application being rendered on the server, you lose access to all things browser. This includes access to the Window object and the DOM. This generally isn't an issue as long as you aren't directly querying the DOM using methods like:
```javascript
  document.getElementById()
  document.querySelector()
```
instead use Angular directives like viewChild/viewChildren:
```javascript
import {Component, ViewChild} from '@angular/core'
@Component({
  template: `
  <div>
    <p #myChild>Pick me!</p>
  </div>
  `
})
export class PickMe{
  @ViewChild('myChild') pickMe;
}
```
_You should never wrap code in a setTimeout just to avoid build errors as it will slow your rendering_

