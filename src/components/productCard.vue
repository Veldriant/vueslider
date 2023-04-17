
<template>
  <swiper :slidesPerView="1" :spaceBetween="30" :loop="true" :pagination="{
    clickable: true,
  }" class="mySwiper">
    <swiper-slide> <img src="../assets/IMG_7234.jpg" alt=""> </swiper-slide>
    <swiper-slide> <img src="../assets/IMG_7235.jpg" alt=""> </swiper-slide>
    <swiper-slide> <img src="../assets/IMG_7236.jpg" alt=""> </swiper-slide>
  </swiper>
</template>
<script>
// Import Swiper Vue.js components
import { Swiper, SwiperSlide } from "swiper/vue";

// Import Swiper styles
import "swiper/swiper.scss";

import "swiper/components/pagination/pagination.min.css"

import "../assets/styles.css";

// import Swiper core and required modules
import SwiperCore, { Pagination, Navigation } from "swiper/core";

// install Swiper modules
SwiperCore.use([Pagination, Navigation]);

export default {
  components: {
    Swiper,
    SwiperSlide,
  },
  data() {
    return {};
  },
  methods: {},
};
</script>
