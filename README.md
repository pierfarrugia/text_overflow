# text overflow


Web design is often broken by too big or too small content.

If we are managing content, it's quite easy to re-write part to have good amount of text, good number of images at right sizes to maintain web design on different screen sizes.

When content coming from elsewhere (database filled by users, scraping...), web design can be easily broken.

CSS tried to address this kind of problems with clamp

https://developer.mozilla.org/en-US/docs/Web/CSS/clamp

Or with text-overflow

https://developer.mozilla.org/en-US/docs/Web/CSS/text-overflow

But that doesn't solve the whole problem, you still have to make some calculation to avoid broken design.


Here with 3 functions you can cut and read too big content. Options are:
- 3 sizes to cut for a block of cards: min, average, max
- 3 way to view full content of a card: overflow, grow (open 1), full (grow all)

It has been tested on Chrome, Firefox, Safari and Edge. Now, it certainly won't work in any situation. First, it's working with CSS Grid. The cut function can certainly be used outside grid, but I didn't test it.


---
*use ctrl-click to open new window*

[Test it on CodePen](https://codepen.io/pierfarrugia/pen/jOKLmzG)

![alt text](https://aonecommunication.ch/content/drop_down_click/drop_down_click.webp)

[Read this on web page](https://aonecommunication.ch/blog.html#text_overflow)

[See it in action full screen](https://aonecommunication.ch/content/text_overflow/text_overflow.html)



---
## HTML Structure

HTML structure is the following:

```HTML
    <div class="cards-size">
        <div class="card-size">
            <div class="card-samesize">
            </div>
        </div>
    </div>
```

- cards-size is the container for a block of card
- card-size is the container for 1 card
- card-samesize is the content of the card

In the demo you also have other tag surrounding those 3. 

```HTML
<section id="section1">
    <div class="container">
        <div class="cards-size">
            <div class="card-size">
                <div class="card-samesize">
                </div>
            </div>
        </div>
    </div>
</section>
```

Data attribute in the "cards-size" defines the way you want the block of card to be cut, and the way you want the content to be read at full size

```HTML
        <div class="cards-size" data-size="min" data-ellipsis="overflow">
```

Here cut size is min, and ellipsis is overflow

You have 3 options for data-size:
- min: all card inside this cards-size will be cut to the smallest height 
- average: all card inside this cards-size will be cut to the average height
- max: all card inside this cards-size will be cut to the biggest height

You have 3 options for data-ellipsis:
- overflow: scroll bar will be added to the card
- grow: card will take its original height
- full: all card inside this cards-size will take the biggest height



---
## CSS

CSS needed:

```css
.cards-size {
    margin: 0;
    padding: 0;
    display: grid;
    grid-template-columns: 1fr;
    grid-template-rows: 1fr;
    grid-column-gap: 0;
    grid-row-gap: 1em;
    align-items: flex-start;
}
@media (min-width: 35.5em) { /* 568px */
}
@media (min-width: 48em) { /* 768px */
    .cards-size {
        grid-template-columns: repeat(2, 1fr);
        grid-column-gap: 0.5em;
    }
}
@media (min-width: 64em) { /* 1024px */
    .cards-size {
        grid-template-columns: repeat(4, 1fr);
        grid-column-gap: 1em;
    }
}
@media (min-width: 85em) { /* 1360px */
}

.card-size {
    position: relative;
    padding: 1em;
    color: #202020;
    background-color: #CCCCCC;
    transition: all 0.3s ease-in-out;
}
.card-size button.ellipsis {
    position: absolute;
    bottom: 0;
    right: 0;
    padding: 0;
    margin: 0;
    width: 2em;
    line-height: 1em;
    border: none;
    background: #E1EEF4;
    z-index: 9;
    font-size: 1.25em;
    font-weight: 700;
    cursor: pointer;
}
.card-samesize {
    overflow: hidden;
    transition: all 0.3s ease-in-out;
}

```

cards-size is defined in CSS grid with 1 column (mobile, small screen), media-query put 2 columns in medium screen, and 4 on large. 

You can change this to any value you need, add xsmall or xlarge if needed.






---
## Javascript

Javascript has 3 main functions:
- heightCardCalc: height calculation on card block
- heightCardSet: apply size and reading type to card + listening event
- ellipsisMore: to react to user clicked on card ellipsis


### heightCardCalc
For a block of card, it calculates the smallest height, the average height and the biggest height. It put those 3 values in a data attribute cutheight, each value separated by a comma.

```javascript
    const heightCardCalc = function (el) {
    let els = el.querySelectorAll('.card-samesize');
    let hmin = 10000;
    let haverage = 0;
    let nbel = 0;
    let hmax = 0;
    // loop through elements of this cards-size container to calculate sizes
    els.forEach(e => {
        // h real size of this card
        let h = e.getBoundingClientRect().height;
        //calculate min
        if (h < hmin) {
            hmin = Math.round(h);
        }
        // calculate average part 1
        haverage += h;
        nbel += 1;
        //calculate max
        if (h > hmax) {
            hmax = Math.round(h);
        }
    })
    // calculate average part 2
    haverage = Math.round(haverage / nbel);
    // attribute cutHeight in cards-size with the 3 values
    el.dataset.cutheight = hmin + ',' + haverage + ',' + hmax;
}
```

### heightCardSet
For a block of card, it first applies the requested size (min, average, max) from data-size of the block cards-size. Value to cut is taken from the corresponding data cutheight calculated in previous function.

For each card, it also apply kind of reading content card will have from the data ellipsis attribute (overflow, grow, full). If not present, and needed (card cut), it also adds the ellipsis button.

Final, it adds the click event on ellipsis button.

```javascript
    const heightCardSet = function (el) {
    // retrieve the sizes
    const ar = (el.dataset.cutheight).split(',');
    // retrieve size wanted
    const size = el.dataset.size;
    let hm = '';
    // select right size for the cut wanted
    if (size === 'min') {
        hm = ar[0];
    } else if (size === 'average') {
        hm = ar[1];
    } else if (size === 'max') {
        hm = ar[2];
    }
    let els = el.querySelectorAll('.card-samesize');
    //retrieve kind of growth wanted
    let how = el.dataset.ellipsis;
    if (how === '') {
        how = 'overflow';
    }
    // loop through elements of this cards-size container
    els.forEach(e => {
        // h real size of this card
        let h = e.getBoundingClientRect().height;
        // apply height selected to element
        e.style.height = hm + 'px';
        // reset the overflow of element to hidden
        e.style.overflowY = 'hidden';
        // if real size higher than cut size
        if (h > hm) {
            // check if ellipsis exist in card, if not create it with the right class
            if (!e.querySelector('.ellipsis')) {
                let btn = document.createElement("button");
                btn.classList.add('ellipsis');
                btn.classList.add('more');
                btn.innerHTML = '&#8943;'; // unicode medium 3 dots
                e.appendChild(btn);
            }
            // loop add event listener to each ellipsis, click call ellipsismore function
            e.querySelector('.ellipsis').addEventListener('click', function (e) {
                e.stopImmediatePropagation();
                ellipsisMore(e.target, how);
            })
        }
    })
}
```

### ellipsisMore
For 1 card, respond to user click to read the full content or to close back the card.

```javascript
    function ellipsisMore(e, how) {
    // test if the card is cut
    if (e.classList.contains('more')) {
        // 3 ways to open the card: overflow, grow (open this only), full (open all card of cards-size container
        if (how === 'overflow') {
            // if overflow, scroll y of card clicked
            e.parentElement.style.overflowY = 'scroll';
            e.innerHTML = '&#8211;'; // unicode minus
            e.classList.remove('more');
        } else if (how === 'grow') {
            // if grow, height of card clicked initial height
            e.parentElement.style.height = 'auto';
            e.innerHTML = '&#8211;'; // unicode minus
            e.classList.remove('more');
        } else if (how === 'full') {
            // if full, height of the tallest card of this cards-size container for all card of this container
            const econtainer = e.closest('.cards-size');
            const ar = (econtainer.dataset.cutheight).split(',');
            const hmax = ar[2];
            const els = econtainer.querySelectorAll('.card-samesize');
            els.forEach(e => {
                e.style.height = hmax + 'px';
                const eellipsis = e.querySelector('.ellipsis');
                if (eellipsis) {
                    eellipsis.innerHTML = '&#8211;'; // unicode minus
                    eellipsis.classList.remove('more');
                }
            })
        }
    } else {
        // closing the card
        // container of this clicked card
        const econtainer = e.closest('.cards-size');
        // 3 sizes values from attribute cutheight
        const ar = (econtainer.dataset.cutheight).split(',');
        // way to cut value from attribute size
        const size = econtainer.dataset.size;
        //hcut size to cut
        let hcut = 0;
        if (size === 'min') {
            hcut = ar[0];
        } else if (size === 'average') {
            hcut = ar[1];
        }
        if ((how === 'overflow') || (how === 'grow')) {
            if (how === 'overflow') {
                // if overflow, scroll y of card hidden
                e.parentElement.scrollTop = 0;
                e.parentElement.style.overflowY = 'hidden';
            } else { //grow
                // if grow, height of card clicked height wanted
                const ar = (e.closest('.cards-size').dataset.cutheight).split(',');
                e.parentElement.style.height = hcut + 'px';
            }
            e.innerHTML = '&#8943;';
            e.classList.add('more');
        } else if (how === 'full') {
            // if full, all card from this container cards-size back to their cut height
            let els = econtainer.querySelectorAll('.card-samesize');
            els.forEach(e => {
                e.style.height = hcut + 'px';
                const eellipsis = e.querySelector('.ellipsis');
                if (eellipsis) {
                    eellipsis.innerHTML = '&#8943;';
                    eellipsis.classList.add('more');
                }
            })
        }
    }
}
```


Try the demo, to see it working.

Javascript here is deliberately detailed. Not compression, or reduction of 2-3 lines in one to be more understandable. You can easily reduce it.

There would be another easy "opening" option using a lightbox, but personnaly I don't like so much opening modal windows when not necessary.

Hope that can help.

Thanks for reading

*(tested on Chrome, Firefox, Edge, Safari)*
