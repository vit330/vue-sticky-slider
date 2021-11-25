# Sticky-slider

```template```
```html
<ui-sticky-slider>
  <template #nav>
    <button :class="[$style.nav, $style.nav_prev]" type="button">←</button>
    <button :class="[$style.nav, $style.nav_next]" type="button">→</button>
  </template>
  
  <div :class="$style.slideWrapper">
    <div :class="$style.slide">
      3
    </div>
  </div>
  <div :class="$style.slideWrapper">
    <div :class="$style.slide">
      2
    </div>
  </div>
  <div :class="$style.slideWrapper">
    <div :class="$style.slide">
      1
    </div>
  </div>
</ui-sticky-slider>
```

```script```
```js
import UiStickySlider from '~/components/ui/sticky-slider'

export default {
  components: {
    UiStickySlider
  }
}
```

```style```
```scss
.slideWrapper {
  .slide {
    width: 30em;
    height: 25em;
    color: #000;
    box-shadow: 0 -0.3em 1em rgba(#000, 0.15);
    transform-origin: top;
    transition: transform 0.2s;
  }

  &:nth-child(1) .slide {
    background-color: tomato;
  }
  &:nth-child(2) .slide {
    background-color: wheat;
  }
  &:nth-child(3) .slide {
    background-color: pink;
  }
}

.nav {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  width: 2em;
  height: 2em;
  border: 1px solid #fff;
  transition: opacity $transition-duration;

  &_prev {
    left: 5em;
  }
  &_next {
    right: 5em;
  }
  &:disabled {
    pointer-events: none;
    opacity: 0.5;
  }
}
```
