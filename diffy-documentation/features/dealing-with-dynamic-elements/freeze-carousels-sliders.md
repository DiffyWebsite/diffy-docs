# Freeze Carousels / Sliders

You can use the following JavaScript snippets to freeze Sliders / Carousels on your sites. Please [let us know](mailto:support@diffy.website) if we miss any slider libraries you use. There are plenty of them!



### Owl Carousel

[https://owlcarousel2.github.io/OwlCarousel2/](https://owlcarousel2.github.io/OwlCarousel2/)

Demo site [https://owlcarousel2.github.io/OwlCarousel2/demos/autoplay.html](https://owlcarousel2.github.io/OwlCarousel2/demos/autoplay.html)

\
Code:

```
jQuery('.owl-carousel').trigger('stop.owl.autoplay');
jQuery('.owl-carousel').trigger('to.owl.carousel', 0, 0);
```



### Slick Slider

[https://github.com/kenwheeler/slick](https://github.com/kenwheeler/slick)

Demo site [https://kenwheeler.github.io/slick/](https://kenwheeler.github.io/slick/)&#x20;

\
Code:

```
jQuery('.slick-slider').slick('slickSetOption', 'autoplay', false, true);
jQuery('.slick-slider').slick('slickPause');
jQuery('.slick-slider').slick('slickGoTo', 0, true);
```

Or if jQuery is not available

```
document.querySelectorAll('.slick-slider').forEach((slider) => {
  slider.slick.goTo(0, true);
  slider.slick.paused = true;
});
```

### Swiper Carousel

[https://github.com/nolimits4web/swiper](https://github.com/nolimits4web/swiper)

Demo site [https://swiperjs.com/demos/280-autoplay/core](https://swiperjs.com/demos/280-autoplay/core)

\
Code:

```
document.querySelector('.swiper').swiper.autoplay.stop();
document.querySelector('.swiper').swiper.setProgress(0, 0);
```



### Smart Slider

[https://smartslider3.com/](https://smartslider3.com/)

Example: [https://pearleventsaustin.com/weddings/](https://pearleventsaustin.com/weddings/)

\
Code

```
_N2.r('#n2-ss-3', function(){
    _N2['#n2-ss-3'].pauseAutoplay();
    _N2['#n2-ss-3'].slide(0);
});
```

<br>
