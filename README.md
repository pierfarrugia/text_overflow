# dropdown click



Classical way is to use CSS “sticky” to fix header on top of screen. 

Main pro: it’s only a css (with properties positioning relative, absolute...). 

That’s working well in general. Sometimes you encounter problems: event not working (and not solved with overflow not hidden), position...

**Why using javascript?** 

Here it's a click (not onHover), so you already have javascript to respond the user click. The idea is to add some javascript lines to have a bullet proof dropdown working in all situations.



---
*use ctrl-click to open new window*

[Test it on CodePen with animation](https://codepen.io/pierfarrugia/pen/abKJgqw)

![alt text](https://aonecommunication.ch/content/drop_down_click/drop_down_click.webp)

[Read this on HTML page](https://aonecommunication.ch/blog.html#drop_down_click)

[See it in action full screen](https://aonecommunication.ch/content/drop_down_click/drop_down_click.html)

[See it in action full screen: anim version](https://aonecommunication.ch/content/drop_down_click/drop_down_click_anim.html)


---
## HTML 

First part of HTML is for the demo purpose to have a header with a menu, and some sections.
The element to fire the event must have:
- dropdown-click class to fire the event
- data-dropdown="XXX", XXX is the name of the dropdown linked with this click
```html
    <li class="dropdown-click" data-dropdown="XXX">dropdown</li>
```


Second part have 3 different dropwowns defined.

```html
<!-- class dropdown-content mandatory --> 
<!-- unic ID mandatory same as data-dropdown from the associated click -->
<div id="XXX" class="dropdown-content">
<!-- whatever you want here -->
    <ul>
        <li><a href="#section1">menu 1 section 1</a></li>
        <li><a href="#section2">menu 1 section 2</a></li>
        <li><a href="#section3">menu 1 section 3</a></li>
    </ul>
</div>
...
```

Each dropdown has a class "dropdown-content": 
- dropdown is a regular HTML div
- position absolute
- top -3000px
- opacity 0

This way, absolute at -3000px with opacity 0, the element still have a bounding rect to have its size. If we were using display none, we wouldn't have size: element is removed from rendering. And we need size to know where to render it when fired element is clicked.

Content of dropdown can be amy html you want. For the demo, you have submenu but could be image, video, or whatever you want.

You can add other dropdown to be fired on any HTML regular tag: button, image, icon...


---
## CSS

First part of CSS is for the demo purpose to style header, sections...

Main part needed is the 2 styles in "dropdown styles":

- dropdown-click: class for elements to fire the drop-down
- dropdown-content: class for elements to drop-down

```css
    .dropdown-click { /* class for elements to fire the drop-down, NEEDED */
        position: relative;
        cursor: pointer;
        z-index: 98;
    }

    .dropdown-content { /* class for elements to drop-down, NEEDED */
        position: absolute;
        top: -3000px;
        opacity: 0;
        min-width: fit-content;
        z-index: 99;
        margin: 0;
        padding: 0;
    }
```

Last part is styles for the submenu inside the dropdown content in the demo.


---
## Javascript

```javascript
    const clickDropDown = () => {
    let els_dropdown = document.querySelectorAll('.dropdown-click');
    // adding event listener on all elements with class dropdown-click
    els_dropdown.forEach((e) => e.addEventListener('click', function (e) {
        e.stopImmediatePropagation();
        // closing all opened dropdowns, see function thereafter, needed as function for reusability with scroll
        closeAllDropDown();

        // el_event, element which fire the click and el_event_rect it's bounding rect
        const el_event = e.target;
        const el_event_rect = el_event.getBoundingClientRect();

        // el_ddc, element with the ID of the attribute 'dropdown' from el_event and el_ddc_rect it's bounding rect
        const el_ddc = document.querySelector('#' + el_event.dataset.dropdown);
        const el_ddc_rect = el_ddc.getBoundingClientRect();

        // calculation has to be made for the top, if header bottom of screen
        let el_ddc_top;

        // TOP: compare the bottom of clicked element PLUS its corresponding dropdown height with the window height to fire dropdown under or above
        let diff = (el_event_rect.top + el_event_rect.height + el_ddc_rect.height) - window.innerHeight;
        if (diff < 0) {
            // if diff < 0, dropdown will be under header
            el_ddc_top = el_event.offsetTop + el_event_rect.height;
            el_ddc.style.top = (el_ddc_top - el_ddc_rect.height) + 'px';
        } else {
            // if diff > 0, dropdown will be above header
            el_ddc_top = el_event.offsetTop - el_ddc_rect.height;
            el_ddc.style.top = (el_ddc_top + el_ddc_rect.height) + 'px';
        }

        // LEFT: compare the left of clicked element PLUS its corresponding dropdown width with the window width
        diff = (el_event_rect.left + el_ddc_rect.width) - window.innerWidth;
        if (diff < 0) {
            // if diff < 0, dropdown will be aligned left of click element
            el_ddc.style.left = el_event_rect.left + 'px';
        } else {
            // if diff > 0, dropdown will be aligned right of click element
            el_ddc.style.left = el_event_rect.right - el_ddc_rect.width + 'px';
        }

        // show dropdown
        el_ddc.style.top = el_ddc_top + 'px';
        el_ddc.style.opacity = '1';
    }));
}
window.addEventListener('load', clickDropDown);
```

- first part: we select all dropdown clicks and attach a click event handler on it
- for each, we define the element which fire the event and its bounding rect
- from its data attribute dropwon, we select the element to show and it's bounding rect

Calculation:
- top: to check if there is enough room under the click element to show the dropdown under otherwise on top (here we store in variable, that's for the anim version)
- left: to check if there is enough room right of click element to show the dropdown right otherwise show it on left

Last line we call this function on window load.

```javascript
    const closeAllDropDown = () => {
        let els_dropContent = document.querySelectorAll('.dropdown-content');
        els_dropContent.forEach(e => {
            e.style.opacity = '0';
            e.style.top = '-3000px';
        });
    }
    window.addEventListener('load', closeAllDropDown);

```
Generic function to reset all dropdowns, called in the clicks event, on scroll, and if body clicked.




```javascript
    document.addEventListener('scroll', closeAllDropDown);
```
On scroll, all dropdowns are reset. You can add it or not. Nicer.


```javascript
    document.body.addEventListener('click', closeAllDropDown);
```
On body click, reset all dropdowns. If you stop propagation in click event in your website, a click arriving to body means it was done in empty space.

If you have several things to reset, you could write it this way:
```javascript
    document.body.addEventListener('click', function () {
        closeAllDropDown();
        // something else
        // another one
        //...
    });
```

## Anim dropdown

What about animating dropdown show.

In previous version we stored the dropdown top value (el_ddc_top) to use it at the end of the function.

Just after we reset the transition:
```javascript
    el_ddc.style.transition = '';
```

In fact, we'll make the transition in 2 steps:
- opacity at 0, no transition value, we position the dropdown (example: if enough space under the clicked element, dropdown first position on top of this element minus the height of the dropdown)
- transition value, opacity 1, final dropdown position

If we do the transition in 1 step, dropdown will come from its original position at -3000px, and you'll see it crossing the whole screen.

The little trick is we have step 1 to be done before step 2 is done otherwise we'll have same visual effect as if it were at -3000px. Now it's just 2 values changed on an element with opacity at 0 so really not a lot of calculation. Instead of building an handler to check step one finished, just a small timeout is ok:
Just after we reset the transition:
```javascript
    setTimeout(function () {
        el_ddc.style.transition = '.3s ease-in-out';
        el_ddc.style.top = el_ddc_top + 'px';
        el_ddc.style.opacity = '1';
}, 200)
```
Because we have reset transition before step1, we put it back for step 2. We also reset the transition in close all dropdowns.

Final function for anim:
```javascript
    /* ========== dropdown click ========== */
const clickDropDown = () => {
    let els_dropdown = document.querySelectorAll('.dropdown-click');
    // adding event listener on all elements with class dropdown-click
    els_dropdown.forEach((e) => e.addEventListener('click', function (e) {
        e.stopImmediatePropagation();
        // closing all opened dropdowns, see function thereafter, needed as function for reusability with scroll
        closeAllDropDown();

        // el_event, element which fire the click and el_event_rect it's bounding rect
        const el_event = e.target;
        const el_event_rect = el_event.getBoundingClientRect();

        // el_ddc, element with the ID of the attribute 'dropdown' from el_event and el_ddc_rect it's bounding rect
        const el_ddc = document.querySelector('#' + el_event.dataset.dropdown);
        const el_ddc_rect = el_ddc.getBoundingClientRect();

        // calculation has to be made for the top, if header bottom of screen, needed as variable el_ddc_top
        let el_ddc_top;
        // reset transition, needed for first changes of value, otherwise coming from -3000!
        el_ddc.style.transition = '';

        // TOP: compare the bottom of clicked element PLUS its corresponding dropdown height with the window height to fire dropdown under or above
        let diff = (el_event_rect.top + el_event_rect.height + el_ddc_rect.height) - window.innerHeight;
        if (diff < 0) {
            // if diff < 0, dropdown will be under header
            el_ddc_top = el_event.offsetTop + el_event_rect.height;
            el_ddc.style.top = (el_ddc_top - el_ddc_rect.height) + 'px';
        } else {
            // if diff > 0, dropdown will be above header
            el_ddc_top = el_event.offsetTop - el_ddc_rect.height;
            el_ddc.style.top = (el_ddc_top + el_ddc_rect.height) + 'px';
        }

        // LEFT: compare the left of clicked element PLUS its corresponding dropdown width with the window width
        diff = (el_event_rect.left + el_ddc_rect.width) - window.innerWidth;
        if (diff < 0) {
            // if diff < 0, dropdown will be aligned left of click element
            el_ddc.style.left = el_event_rect.left  + 'px';
        } else {
            // if diff > 0, dropdown will be aligned right of click element
            el_ddc.style.left = el_event_rect.right - el_ddc_rect.width  + 'px';
        }

        // little timeout: time for previous value to be applied before opacity on
        // little transition of only dropdown height otherwise transition arrives from -3000px
        setTimeout(function () {
            el_ddc.style.transition = '.3s ease-in-out';
            el_ddc.style.top = el_ddc_top + 'px';
            el_ddc.style.opacity = '1';
        }, 200)
    }));
}
window.addEventListener('load', clickDropDown);

/* ========== if scroll close all dropdown ========== */
const closeAllDropDown = () => {
    let els_dropContent = document.querySelectorAll('.dropdown-content');
    els_dropContent.forEach(e => {
        e.style.transition = '';
        e.style.opacity = '0';
        e.style.top = '-3000px';
    });
}
window.addEventListener('load', closeAllDropDown);

// on scroll close all dropdowns
document.addEventListener('scroll', closeAllDropDown);
// on click body close all dropdowns. Meaning event bubbling arrived to body, Other events has to stop propagation or not
document.body.addEventListener('click', closeAllDropDown);

```





Thanks for reading

*(tested on Chrome, Firefox, Edge, Safari)*
