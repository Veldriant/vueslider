<template>
  <section class="product">
    <div class="info">
      <div class="gallery">
        <img
          class="gallery__main"
          :src="images[mainImage]?.src"
          alt="фотография товара"
        />
        <div class="gallery__smalls">
          <img
            v-for="(image, index) in slicedImages"
            :key="index"
            class="gallery__small"
            :src="image.src"
            alt="фотография товара"
            @mouseover="galleryHoverImageHandler(index)"
          />
        </div>
      </div>
      <div class="actions">
        <div class="actions__product-name">
          {{ title }}
        </div>
        <div class="actions__color">
          <span class="actions__color-title">
            ЦВЕТ
          </span>
          <div class="actions__color-selector">
            <div
              v-for="(color, index) in computedColors"
              :key="index"
              :class="{
                'actions__color-part': true,
                'active': activeColor === index
              }"
              @click="colorClickHandler(index)"
            >
              {{ color }}
            </div>
          </div>
        </div>
        <div class="actions__size">
          <span class="actions__size-title">
            РАЗМЕР
          </span>
          <div class="actions__size-selector">
            <div
              v-for="(size, index) in computedSizes"
              :key="index"
              :class="{
                'actions__size-part': true,
                'active': activeSize === index
              }"
              @click="sizeClickHandler(index)"
            >
              {{ size }}
            </div>
          </div>
        </div>
        <div class="actions__size-table">
          Таблица размеров
        </div>
        <span
          :class="{
            'actions__cost': true,
            'sale': Boolean(computedSaleCost)
          }"
        >
          {{ computedRegularCost }} ₽
        </span>
        <span
          v-if="computedSaleCost"
          class="actions__sale-cost"
        >
          {{ computedSaleCost }} ₽
        </span>
        <div
          :class="{
            'actions__button': true,
            'active': buttonWatch === true && !isInCart,
            'disabled': buttonWatch === false || isInCart || !isInStock
          }"
          @click="addToCartButtonHandler"
        >
          {{ computedButtonText }}
        </div>
      </div>
    </div>
    <div class="description">
      <div class="description__content">
        <span class="description__name">
          {{ title }}
        </span>
        <div class="description__texts">
          <span
            class="description__text"
            v-html="description"
          />
          <div class="description__attributes">
            <div
              v-for="part in compound"
              :key="part"
              class="description__attributes-part"
            >
              <span class="description__attributes-title">
                {{ part.name }}
              </span>
              <span class="description__attributes-content">
                {{ part.options[0] }}
              </span>
            </div>
          </div>
        </div>
      </div>
      <video
        v-if="video"
        class="description__video"
        :src="video"
        muted
        loop
        autoplay
      />
    </div>
    <div
      v-if="recommendedProducts.length"
      class="recommendations"
    >
      <span class="recommendations__title">
        Рекомендуем
      </span>
      <div class="recommendations__container">
        <product-item
          v-for="el in recommendedProducts"
          :is-filter="true"
          :itemId="el.id"
          :item-slug="el.slug"
          :key="el.id"
          :name="el.name"
          :images="el.images"
          :price="el.price"
          :sale_price="el.sale_price"
          :regular_price="el.regular_price"
        />
      </div>
    </div>
  </section>
</template>

<script>
// import ProductItem from '@/components/elements/ProductItem.vue'
// import { mapStores } from 'pinia'
// import { useProductsStore } from '@/stores/products'
// import { useCartStore } from '@/stores/cart'
export default {
  name: "productCard",
  data() {
    return {
      activeSize: 0,
      activeColor: 0,
      productId: '',
      isVariative: false,
      variations: [],
      title: '',
      cost: '',
      priceHtml: '',
      slug: '',
      sizes: [],
      colors: [],
      images: [],
      mainImage: 0,
      video: '',
      description: '',
      compound: []
    };
  },
  computed: {
    // ...mapStores(useProductsStore, useCartStore),
    cart() {
      return this.cartStore.cart;
    },
    products() {
      return this.productsStore.products;
    },
    foundProduct() {
      return this.productsStore.foundProduct;
    },
    recommendedProducts() {
      return this.productsStore.recommendedProducts;
    },
    buttonWatch() {
      return this.activeColor !== null && this.activeSize !== null;
    },
    isInCart() {
      let inCart = [];
      if (this.isVariative && this.activeSize !== null && this.activeColor !== null) {
        inCart = this.cart.filter(product => (product.productId === this.productId && product.variationId === this.variationId));
      } else if (this.activeSize !== null && this.activeColor !== null) {
        inCart = this.cart.filter(product => product.productId === this.productId);
      }
      if (inCart.length) {
        return true;
      } else {
        return false;
      }
    },
    isInStock() {
      let inStock = true;
      if (this.isVariative && this.activeSize !== null && this.activeColor !== null) {
        inStock = this.variations.find(variation => (variation.id === this.variationId))?.stock_status === 'instock';
      } else if (this.activeSize !== null && this.activeColor !== null) {
        inStock = this.foundProduct?.stock_status === 'instock';
      }
      return inStock;
    },
    slicedImages() {
      return this.images.slice(0, 3);
    },
    computedSizes() {
      if (this.sizes.length) {
        return this.sizes;
      } else {
        return ['one size'];
      }
    },
    computedColors() {
      if (this.colors.length) {
        return this.colors;
      } else {
        return ['one color'];
      }
    },
    computedButtonText() {
      if (!this.isInCart && this.isInStock) {
        return 'В корзину';
      } else if (!this.isInStock) {
        return 'Нет в наличии';
      } else {
        return 'Уже в корзине';
      }
    },
    computedCosts() {
      let costString = this.priceHtml.replace(/(&#[0-9]*;)/g,"");
      costString = costString.replace(/[ ]/g,"");
      const costs = costString.match(/[0-9]{1,}/g) || [];
      return costs;
    },
    computedSaleCost() {
      if (this.computedCosts.length <= 1) {
        return false;
      }
      return this.computedCosts[0];
    },
    computedRegularCost() {
      if (this.computedCosts.length < 1) {
        return this.cost;
      } else if (this.computedCosts.length === 1) {
        return this.computedCosts[0];
      } else {
        return this.computedCosts[1];
      }
    },
    variationId() {
      if (!this.isVariative) {
        return
      }
      let id = '';

      this.variations.forEach(variation => {
        const size = variation.attributes.find(attribute => attribute.name === 'Size')?.option;
        const color = variation.attributes.find(attribute => attribute.name === 'Color')?.option;
        
        if (
          (size && color && size === this.sizes[this.activeSize] && color === this.colors[this.activeColor])
          ||
          (!color && size && size === this.sizes[this.activeSize])
          ||
          (!size && color && color === this.colors[this.activeColor])
        ) {
          id = variation.id;
        }
      });

      return id;
    }
  },
  mounted() {
    if (!this.products.length) {
      Promise.allSettled([
        this.productsStore.fetchProducts()
      ])
      .then(() => {
        this.productsStore.searchProductBySlug(this.$route.params.id);
      })
      .then(() => {
        this.productsStore.randomRecommended(this.$route.params.id);
        this.productsStore.randomRecommended(this.$route.params.id);
        this.getData();
        this.getVariations();
      })
    } else {
      this.productsStore.searchProductBySlug(this.$route.params.id);
      this.productsStore.randomRecommended(this.$route.params.id);
      this.getData();
      this.getVariations();
    }
  },
  watch: {
    '$route.params': {
      handler () {
        if (this.$route.name == "product") {
          this.productsStore.searchProductBySlug(this.$route.params.id);
          this.productsStore.randomRecommended(this.$route.params.id);
          this.getData();
          this.getVariations();
          this.activeColor = 0;
          this.activeSize = 0;
        }
      }
    }
  },
  methods: {
    getData() {
      this.productId = this.foundProduct.id;
      this.isVariative = this.foundProduct.type === 'variable' ? true : false;
      this.title = this.foundProduct.name;
      this.cost = this.foundProduct.price;
      this.saleCost = this.foundProduct.saleCost;
      this.video = this.foundProduct.meta_data[0]?.value;
      this.description = this.foundProduct.description;
      this.images = this.foundProduct.images;
      this.priceHtml = this.foundProduct.price_html;
      this.slug = this.foundProduct.slug;

      let sizes = this.foundProduct.attributes.find(elem => elem.name === 'Size')?.options;
      let colors = this.foundProduct.attributes.find(elem => elem.name === 'Color')?.options;
      if (sizes) {
        this.sizes = sizes;
      } else {
        this.sizes = [];
      }
      if (colors) {
        this.colors = colors;
      } else {
        this.colors = [];
      }
      let compleng = 0;
      this.compound = [];
      this.foundProduct.attributes.forEach(attribute => {
        if (attribute.name != "Color" && attribute.name != "Size") {
          this.compound[compleng] = attribute;
          compleng++;
        }
      });
    },
    async getVariations() {
      this.variations = await this.productsStore.fetchVariations(this.foundProduct.id);
    },
    galleryHoverImageHandler(index) {
      this.mainImage = index
    },
    sizeClickHandler(index) {
      if (this.activeSize == index) {
        this.activeSize = null
      } else {
        this.activeSize = index
      }
    },
    colorClickHandler(index) {
      if (this.activeColor == index) {
        this.activeColor = null
      } else {
        this.activeColor = index
      }
    },
    addToCartButtonHandler() {
      if (!this.isInCart && this.isInStock && this.activeSize !== null && this.activeColor !== null) {
        this.cartStore.addToCart(this.productId, this.variationId);
      }
    }
  }
}
</script>

<style lang="scss" scoped>
@media screen and (min-width: 1200px) {
  .product {
    width: 100vw;
    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: var(--white);
    padding-left: calc((100vw - 1224px) / 2);
    padding-right: calc((100vw - 1224px) / 2);
    padding-bottom: 105px;
  }

  .info {
    width: 100%;
    display: flex;
    margin-top: 15px;
  }

  .gallery {
    display: flex;

    &__main {
      width: 600px;
      height: 650px;
      object-position: center;
      object-fit: cover;
    }

    &__smalls {
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      width: 185px;
      height: 650px;
      margin-left: 25px;
    }

    &__small {
      width: 185px;
      height: 185px;
      object-position: center;
      object-fit: cover;
    }
  }

  .actions {
    width: 350px;
    display: flex;
    flex-direction: column;
    margin-left: 65px;

    &__product-name {
      font-family: "Mont", Helvetica;
      font-size: 32px;
      font-weight: 700;
      line-height: 1.2;
    }

    &__size {
      display: flex;
      flex-direction: column;
      margin-top: 35px;

      &-title {
        font-family: "Mont", Helvetica;
        font-size: 12px;
        font-weight: 400;
        text-transform: uppercase;
        color: #C4C4C4;
      }

      &-selector {
        width: 100%;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10px;
        gap: 10px;
      }

      &-part {
        display: flex;
        justify-content: center;
        align-items: center;
        min-width: 57px;
        padding-left: 12px;
        padding-right: 12px;
        height: 32px;
        text-transform: uppercase;
        border-radius: 4px;
        border: 2px #000 solid;
        user-select: none;
        transition: color 0.2s, border 0.2s;
        cursor: pointer;
        color: #000;
        font-family: "Mont", Helvetica;
        font-size: 18px;
        font-weight: 400;
        padding-top: 3px;
        line-height: 1;

        &.active {
          border: 2px #CC0033 solid;
          color: #CC0033;
        }

        &.disabled {
          border: 2px #C4C4C4 solid;
          color: #C4C4C4;
        }
      }
    }

    &__color {
      display: flex;
      flex-direction: column;
      margin-top: 30px;

      &-title {
        font-family: "Mont", Helvetica;
        font-size: 12px;
        font-weight: 400;
        text-transform: uppercase;
        color: #C4C4C4;
      }

      &-selector {
        width: 100%;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10px;
        gap: 10px;
      }

      &-part {
        display: flex;
        justify-content: center;
        align-items: center;
        min-width: 57px;
        padding-left: 12px;
        padding-right: 12px;
        height: 32px;
        text-transform: uppercase;
        border-radius: 4px;
        border: 2px #000 solid;
        user-select: none;
        transition: color 0.2s, border 0.2s;
        cursor: pointer;
        color: #000;
        font-family: "Mont", Helvetica;
        font-size: 18px;
        font-weight: 400;
        padding-top: 3px;
        line-height: 1;

        &.active {
          border: 2px #CC0033 solid;
          color: #CC0033;
        }

        &.disabled {
          border: 2px #C4C4C4 solid;
          color: #C4C4C4;
        }
      }
    }

    &__size-table {
      color: #000;
      font-family: "Mont", Helvetica;
      font-size: 14px;
      font-weight: 400;
      text-decoration: none;
      cursor: pointer;
      margin-top: 15px;

      &:hover {
        text-decoration: underline;
      }
    }

    &__cost {
      color: #000;
      font-family: "Mont", Helvetica;
      font-size: 24px;
      font-weight: 600;
      line-height: 1;
      margin-top: 40px;

      &.sale {
        text-decoration: line-through;
      }
    }

    &__sale-cost {
      color: #CC0033;
      font-family: "Mont", Helvetica;
      font-size: 24px;
      font-weight: 600;
      line-height: 1;
      margin-top: 8px;
    }

    &__button {
      width: 180px;
      height: 45px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 5px;
      padding-top: 2px;
      user-select: none;
      transition: background-color 0.2s;
      margin-top: 45px;

      color: #fff;
      font-family: "Mont", Helvetica;
      font-size: 18px;
      font-weight: 400;

      &.active {
        background-color: #000;
        cursor: pointer;
      }

      &.disabled {
        background-color: #c4c4c4;
        cursor: not-allowed;
      }
    }
  }

  .description {
    display: flex;
    width: 100%;
    margin-top: 50px;

    &__content {
      width: 730px;
      display: flex;
      flex-direction: column;
    }

    &__name {
      font-family: 'Mont', Helvetica;
      font-size: 18px;
      line-height: 1.25;
      font-weight: 700;
    }

    &__texts {
      width: 100%;
      display: flex;
      margin-top: 30px;
    }

    &__text {
      width: 370px;
      display: flex;
      flex-direction: column;
      gap: 10px;
      font-family: 'Mont', Helvetica;
      font-size: 14px;
      font-weight: 400;
      line-height: 1.8;

      &:deep(p) {
        margin: 0;
      }
    }

    &__attributes {
      width: 245px;
      display: flex;
      flex-direction: column;
      gap: 25px;
      margin-left: 45px;

      &-part {
        display: flex;
        flex-direction: column;
      }

      &-title {
        font-family: 'Mont', Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1.8;
        color: #C4C4C4;
      }

      &-content {
        font-family: 'Mont', Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1.8;
        color: #000;
      }
    }

    &__video {
      position: relative;
      width: 420px;
      height: 600px;
      object-fit: cover;
    }
  }

  .recommendations {
    width: 100%;
    display: flex;
    flex-direction: column;
    margin-top: 125px;

    &__title {
      font-family: 'Mont', Helvetica;
      font-size: 36px;
      font-weight: 700;
      color: #CC0033;
    }

    &__container {
      width: 100%;
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      margin-top: 65px;
    }
  }
}
@media screen and (min-width: 640px) and (max-width: 1199px) {
  .product {
    width: 100vw;
    display: flex;
    flex-direction: column;
    align-items: center;
    background-color: var(--white);
    padding-left: calc((100vw - 620px) / 2);
    padding-right: calc((100vw - 620px) / 2);
    padding-bottom: 105px;
  }

  .info {
    width: 100%;
    display: flex;
    margin-top: 25px;
  }

  .gallery {
    display: flex;
    flex-direction: column;

    &__main {
      width: 400px;
      height: 400px;
      object-position: center;
      object-fit: cover;
    }

    &__smalls {
      display: flex;
      justify-content: space-between;
      width: 400px;
      height: 125px;
      margin-top: 12px;
    }

    &__small {
      width: 125px;
      height: 125px;
      object-position: center;
      object-fit: cover;
    }
  }

  .actions {
    width: 185px;
    height: 490px;
    display: flex;
    flex-direction: column;
    margin-left: 30px;

    &__product-name {
      font-family: "Mont", Helvetica;
      font-size: 20px;
      font-weight: 700;
    }

    &__size {
      display: flex;
      flex-direction: column;
      margin-top: 35px;

      &-title {
        color: #C4C4C4;
        font-family: "Mont", Helvetica;
        font-size: 12px;
        font-weight: 400;
        text-transform: uppercase;
      }

      &-selector {
        width: 180px;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10px;
        gap: 10px;
      }

      &-part {
        min-width: 47px;
        height: 28px;
        display: flex;
        justify-content: center;
        align-items: center;
        padding-top: 3px;
        padding-left: 12px;
        padding-right: 12px;
        border-radius: 4px;
        border: 2px #000 solid;

        color: #000;
        font-family: "Mont", Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1;
        text-transform: uppercase;

        user-select: none;
        transition: color 0.2s, border 0.2s;

        &.active {
          border: 2px #CC0033 solid;
          color: #CC0033;
        }

        &.disabled {
          border: 2px #C4C4C4 solid;
          color: #C4C4C4;
        }
      }
    }

    &__color {
      display: flex;
      flex-direction: column;
      margin-top: 30px;

      &-title {
        color: #C4C4C4;
        font-family: "Mont", Helvetica;
        font-size: 12px;
        font-weight: 400;
        text-transform: uppercase;
      }

      &-selector {
        width: 180px;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10px;
        gap: 10px;
      }

      &-part {
        min-width: 57px;
        height: 28px;
        display: flex;
        justify-content: center;
        align-items: center;
        padding-top: 3px;
        padding-left: 12px;
        padding-right: 12px;
        border-radius: 4px;
        border: 2px #000 solid;

        color: #000;
        font-family: "Mont", Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1;
        text-transform: uppercase;

        user-select: none;
        transition: color 0.2s, border 0.2s;

        &.active {
          border: 2px #CC0033 solid;
          color: #CC0033;
        }

        &.disabled {
          border: 2px #C4C4C4 solid;
          color: #C4C4C4;
        }
      }
    }

    &__size-table {
      color: #000;
      font-family: "Mont", Helvetica;
      font-size: 14px;
      font-weight: 400;
      text-decoration: none;
      margin-top: 25px;
    }

    &__cost {
      color: #000;
      font-family: "Mont", Helvetica;
      font-size: 24px;
      font-weight: 600;
      line-height: 1;
      margin-top: 25px;

      &.sale {
        text-decoration: line-through;
      }
    }

    &__sale-cost {
      color: #CC0033;
      font-family: "Mont", Helvetica;
      font-size: 24px;
      font-weight: 600;
      line-height: 1;
      margin-top: 8px;
    }

    &__button {
      width: 180px;
      height: 45px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 5px;
      color: #fff;
      font-family: "Mont", Helvetica;
      font-size: 18px;
      font-weight: 400;
      padding-top: 2px;
      user-select: none;
      margin-top: 45px;
      transition: background-color 0.2s;

      &.active {
        background-color: #000;
        cursor: pointer;
      }

      &.disabled {
        background-color: #c4c4c4;
        cursor: not-allowed;
      }
    }
  }

  .description {
    width: 100%;
    display: flex;
    flex-direction: column-reverse;
    margin-top: 40px;

    &__content {
      width: 100%;
      display: flex;
      flex-direction: column;
    }

    &__name {
      width: 100%;
      font-family: "Mont", Helvetica;
      font-size: 18px;
      font-weight: 700;
    }

    &__texts {
      width: 100%;
      display: flex;
      margin-top: 30px;
    }

    &__text {
      width: 380px;
      display: flex;
      flex-direction: column;
      gap: 10px;
      font-family: "Mont", Helvetica;
      font-size: 14px;
      line-height: 1.4;
      font-weight: 400;

      &:deep(p) {
        margin: 0;
      }
    }

    &__attributes {
      width: 220px;
      display: flex;
      flex-direction: column;
      gap: 35px;
      margin-left: 20px;

      &-part {
        display: flex;
        flex-direction: column;
      }

      &-title {
        font-family: 'Mont', Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1.8;
        color: #C4C4C4;
      }

      &-content {
        font-family: 'Mont', Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1.8;
        color: #000;
      }
    }

    &__video {
      display: none;
    }
  }

  .recommendations {
    position: relative;
    display: flex;
    flex-direction: column;
    margin-top: 50px;

    &__title {
      color: #CC0033;
      font-family: 'Mont', Helvetica;
      font-size: 24px;
      font-weight: 800;
    }

    &__container {
      width: 100%;
      display: flex;
      flex-direction: row;
      flex-wrap: wrap;
      margin-top: 35px;
    }
  }
}
@media screen and (max-width: 639px) {
  .product {
    width: 100vw;
    display: flex;
    flex-direction: column;
    background-color: var(--white);
    padding-left: calc((100vw - 300px) / 2);
    padding-right: calc((100vw - 300px) / 2);
    padding-bottom: 80px;
  }

  .info {
    width: 100%;
    display: flex;
    flex-direction: column;
    margin-top: 15px;
  }

  .gallery {
    display: flex;
    flex-direction: column;

    &__main {
      width: 300px;
      height: 300px;
      object-position: center;
      object-fit: cover;
    }

    &__smalls {
      display: flex;
      flex-direction: row;
      justify-content: space-between;
      width: 100%;
      height: 95px;
      margin-top: 7.5px;
    }

    &__small {
      width: 95px;
      height: 95px;
      object-position: center;
      object-fit: cover;
    }
  }

  .actions {
    width: 300px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    margin-top: 25px;

    &__product-name {
      font-family: "Mont", Helvetica;
      font-size: 20px;
      font-weight: 700;
    }

    &__size {
      display: flex;
      flex-direction: column;
      margin-top: 25px;

      &-title {
        color: #C4C4C4;
        font-family: "Mont", Helvetica;
        font-size: 12px;
        font-weight: 400;
        text-transform: uppercase;
      }

      &-selector {
        width: 270px;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10px;
        gap: 10px;
      }

      &-part {
        min-width: 47px;
        height: 28px;
        display: flex;
        justify-content: center;
        align-items: center;
        padding-top: 3px;
        padding-left: 12px;
        padding-right: 12px;
        border-radius: 4px;
        border: 2px #000 solid;

        color: #000;
        font-family: "Mont", Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1;
        text-transform: uppercase;

        user-select: none;
        transition: color 0.2s, border 0.2s;

        &.active {
          border: 2px #CC0033 solid;
          color: #CC0033;
        }

        &.disabled {
          border: 2px #C4C4C4 solid;
          color: #C4C4C4;
        }
      }
    }

    &__color {
      display: flex;
      flex-direction: column;
      margin-top: 25px;

      &-title {
        color: #C4C4C4;
        font-family: "Mont", Helvetica;
        font-size: 12px;
        font-weight: 400;
        text-transform: uppercase;
      }

      &-selector {
        width: 270px;
        display: flex;
        flex-wrap: wrap;
        margin-top: 10px;
        gap: 10px;
      }

      &-part {
        min-width: 47px;
        height: 28px;
        display: flex;
        justify-content: center;
        align-items: center;
        padding-top: 3px;
        padding-left: 12px;
        padding-right: 12px;
        border-radius: 4px;
        border: 2px #000 solid;

        color: #000;
        font-family: "Mont", Helvetica;
        font-size: 14px;
        font-weight: 400;
        line-height: 1;
        text-transform: uppercase;

        user-select: none;
        transition: color 0.2s, border 0.2s;

        &.active {
          border: 2px #CC0033 solid;
          color: #CC0033;
        }

        &.disabled {
          border: 2px #C4C4C4 solid;
          color: #C4C4C4;
        }
      }
    }

    &__size-table {
      color: #000;
      font-family: "Mont", Helvetica;
      font-size: 14px;
      font-weight: 400;
      text-decoration: none;
      margin-top: 20px;
    }

    &__cost {
      color: #000;
      font-family: "Mont", Helvetica;
      font-size: 24px;
      font-weight: 600;
      line-height: 1;
      margin-top: 25px;

      &.sale {
        text-decoration: line-through;
      }
    }

    &__sale-cost {
      color: #CC0033;
      font-family: "Mont", Helvetica;
      font-size: 24px;
      font-weight: 600;
      line-height: 1;
      margin-top: 8px;
    }

    &__button {
      width: 180px;
      height: 45px;
      display: flex;
      align-items: center;
      justify-content: center;
      border-radius: 5px;
      color: #fff;
      font-family: "Mont", Helvetica;
      font-size: 18px;
      font-weight: 400;
      padding-top: 2px;
      user-select: none;
      margin-top: 30px;
      transition: background-color 0.2s;

      &.active {
        background-color: #000;
        cursor: pointer;
      }

      &.disabled {
        background-color: #c4c4c4;
        cursor: not-allowed;
      }
    }
  }

  .description {
    width: 100%;
    display: flex;
    flex-direction: column-reverse;
    margin-top: 40px;

    &__content {
      width: 100%;
      display: flex;
      flex-direction: column;
    }

    &__name {
      font-family: "Mont", Helvetica;
      font-size: 18px;
      font-weight: 700;
      line-height: 1.25;
    }

    &__texts {
      width: 100%;
      display: flex;
      flex-direction: column;
      margin-top: 25px;
    }

    &__text {
      display: flex;
      flex-direction: column;
      gap: 8px;
      font-family: "Mont", Helvetica;
      font-size: 14px;
      line-height: 1.4;
      font-weight: 400;

      &:deep(p) {
        margin: 0;
      }
    }

    &__attributes {
      width: 245px;
      display: flex;
      flex-direction: column;
      gap: 25px;
      margin-top: 30px;

      &-part {
        display: flex;
        flex-direction: column;
      }

      &-title {
        font-family: "Mont", Helvetica;
        font-size: 14px;
        font-weight: 400;
        color: #C4C4C4;
      }

      &-content {
        font-family: "Mont", Helvetica;
        font-size: 12px;
        font-weight: 400;
        margin-top: 7px;
        color: #000;
      }
    }

    &__video {
      display: none;
    }
  }

  .recommendations {
    display: flex;
    flex-direction: column;
    margin-top: 50px;

    &__title {
      color: #CC0033;
      font-family: "Mont", Helvetica;
      font-size: 22px;
      font-weight: 700;
    }
    
    &__container {
      width: 100%;
      display: flex;
      flex-wrap: wrap;
      margin-top: 35px;
    }
  }
}
</style>