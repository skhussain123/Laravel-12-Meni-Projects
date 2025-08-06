


* https://splidejs.com/guides/autoplay/

<script src="https://cdn.jsdelivr.net/npm/@splidejs/splide@4.1.4/dist/js/splide.min.js"></script>
<link href="https://cdn.jsdelivr.net/npm/@splidejs/splide@4.1.4/dist/css/splide.min.css" rel="stylesheet">


```bash
 <section class="splide">
     <div class="splide__slider">
         <div class="splide__track">
             <ul class="splide__list">
                 <li class="splide__slide"><img style="width: 100%;height:150px"
                         src="{{ asset('images/blog/blog-grid-10.jpg') }}"
                         alt=""></li>

                 <li class="splide__slide"><img style="width: 100%;height:150px"
                         src="{{ asset('images/blog/blog-grid-10.jpg') }}"
                         alt=""></li>
                 <li class="splide__slide"><img style="width: 100%;height:150px"
                         src="{{ asset('images/blog/blog-grid-10.jpg') }}"
                         alt=""></li>

             </ul>
         </div>
     </div>

     <button class="splide__toggle" type="button">
         <span class="splide__toggle__play">Play</span>
         <span class="splide__toggle__pause">Pause</span>
     </button>
 </section>

 <script>
     var splide = new Splide('.splide', {
         type: 'loop',
         perPage: 4, // ⭐ Show 4 cards per row
         perMove: 1,
         gap: '1rem', // ⭐ Optional: space between cards
         autoplay: true,
         pauseOnHover: true,
     });

     splide.on('autoplay:playing', function(rate) {
         console.log(rate); // 0-1
     });

     splide.mount();
 </script>

 ```