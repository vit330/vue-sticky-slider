<template>
  <div :class="$style.container" ref="container">
    <div :class="$style.sticky">
      <div :class="$style.nav" ref="nav">
        <slot name="nav" />
      </div>

      <div :class="$style.slider" ref="slider">
        <slot />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'UiStickySlider',
  props: {
    /** Размер шага translateY при наложении слайдов */
    transformSize: {
      type: Number,
      default: 0.5
    },

    /** Размер шага scale при наложении слайдов */
    scaleSize: {
      type: Number,
      default: 0.05
    }
  },
  data() {
    return {
      /** Ссылка на слайды */
      slides: [],

      /** Ссылка на кнопку "назад" */
      prevNav: null,

      /** Ссылка на кнопку "вперёд" */
      nextNav: null,

      /** Величина прокрутки слайдера */
      containerOffset: 0,

      /** Высота окна */
      windowHeight: 0,

      /** Множитель скорости прокрутки слайдов */
      speedFactor: 0,

      /** Номер верхнего слайда */
      topSlideNumber: 0
    }
  },
  computed: {
    activeIndex() {
      const index = Math.floor((this.containerOffset + 1) / this.windowHeight)
      return (index < 0) ? 0 : index
    }
  },
  methods: {
    /** Инициализация. Устанавливает высоту контейнера, вычисляет множитель скорости, записывает слайды и кнопки навигации */
    init() {
      const sliderRect = this.$refs.slider.getBoundingClientRect()

      this.slides = [...this.$refs.slider.children].reverse()
      this.windowHeight = window.innerHeight
      this.$refs.container.style.height = `${this.windowHeight * this.slides.length}px`
      this.speedFactor = sliderRect.right / this.windowHeight

      this.prevNav = this.$refs.nav.children[0]
      this.nextNav = this.$refs.nav.children[1]

      this.onScroll()
    },

    /** Устанавливает значение прокрутки слайдера */
    setOffset() {
      const containerRect = this.$refs.container.getBoundingClientRect()

      if (containerRect.top > 0) {
        this.containerOffset = 0
        return
      }
      if (-containerRect.top > containerRect.height - this.windowHeight) {
        this.containerOffset = containerRect.height - this.windowHeight
        return
      }
      this.containerOffset = -containerRect.top
    },

    /** Анимация слайдов. Устанавливает положение и трансформации наложения слайдов */
    moveSlides() {
      this.slides.forEach((slide, index) => {
        const realOffset = -this.containerOffset * this.speedFactor + (this.windowHeight * this.speedFactor * index) - 1
        const offset = (realOffset < -1) ? realOffset : 0
        slide.style.transform = `translateX(${offset}px)`

        const slideRect = slide.getBoundingClientRect()
        const nextSlide = slide.previousElementSibling
        const nextScaleElement = nextSlide?.firstElementChild

        if (-offset > slideRect.width) {
          if (nextScaleElement) {
            nextScaleElement.style.transform = `none`
          }
          this.topSlideNumber = index + 1
        } else {
          if (nextScaleElement) {
            const offsetY = (index + 1 - this.topSlideNumber) * this.transformSize * -1
            const scale = 1 - (index + 1 - this.topSlideNumber) * this.scaleSize
            nextScaleElement.style.transform = `translateY(${offsetY}rem) scale(${scale})`
          }
          if (index === 0) {
            this.topSlideNumber = 0
          }
        }
      })
    },

    /**
     * Прокручивает страницу
     * @param {number} offset Величина прокрутки
     */
    scrollTo(offset) {
      this.$scroll.scrollTo({
        y: offset,
        duration: 600
      })
    },

    /** Обработчик события прокрутки */
    onScroll() {
      requestAnimationFrame(() => {
        this.setOffset()
        this.moveSlides()
        this.checkNavDisabled()
      })
    },

    /** Обработчик клика на кнопку "назад" */
    onPrevClick() {
      const containerRect = this.$refs.container.getBoundingClientRect()

      if (-containerRect.top < this.windowHeight) {
        this.scrollTo(containerRect.top)
        return
      }
      const offset = containerRect.top + this.windowHeight * (this.activeIndex - 1)
      this.scrollTo(offset)
    },

    /** Обработчик клика на кнопку "вперёд" */
    onNextClick() {
      const containerRect = this.$refs.container.getBoundingClientRect()
      const offset = containerRect.top + this.windowHeight * (this.activeIndex + 1) + 1

      this.scrollTo(offset)
    },

    /** Проверяет величину прокрутки слайдера и отключает кнопки навигации */
    checkNavDisabled() {
      const containerRect = this.$refs.container.getBoundingClientRect()

      if (this.prevNav) {
        this.prevNav.disabled = this.containerOffset <= 0
      }
      if (this.nextNav) {
        this.nextNav.disabled = this.containerOffset >= containerRect.height - this.windowHeight
      }
    }
  },

  mounted() {
    this.init()

    window.addEventListener('scroll', this.onScroll)
    this.prevNav?.addEventListener('click', this.onPrevClick)
    this.nextNav?.addEventListener('click', this.onNextClick)
  },

  beforeDestroy() {
    window.removeEventListener('scroll', this.onScroll)
    this.prevNav?.removeEventListener('click', this.onPrevClick)
    this.nextNav?.removeEventListener('click', this.onNextClick)
  }
}
</script>

<style lang="scss" module>
.container {
  position: relative;
}

.sticky {
  position: sticky;
  top: 0;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  height: vh(100);
}

.slider {
  position: relative;

  > * {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    will-change: transform;

    &:last-child {
      position: relative;
    }
  }
}
</style>
