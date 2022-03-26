<template>
  <li class="product-preview">
      <img class="product-preview__image" :src="imgPath" :alt="product.article">

      <div class="product-preview__meta">
        <div class="product-preview__ttl"><strong>{{ product.article }}</strong></div>
        <div class="font-light">{{ description }}</div>
        <div class="font-light" v-if="colorAmount > 0">
            {{ colorAmount }} {{ colorAmount > 1 ? "couleurs" : "couleur" }}
        </div>
        <div class="product-preview__price">{{ product.prix }}</div>
      </div>
  </li>
</template>

<script setup>
    const props = defineProps(['product']);
    const product = props.product;

    const imgPath = "/img/" + product.photo;
    const colorAmount = product.couleur.split(', ').length;
    const description = "Chaussure" + (product.sexe !== "Mixte" ? " pour " + product.sexe : "");
</script>

<style lang="scss">
    .product-preview {

        &__meta {
            margin-top: 1rem;
        }

        &__ttl {
            margin-bottom: 1rem;
            transition: .3s color ease-in-out;
        }

        &__image {
            display: block;
            max-width: 100%;
            width: 100%;
            aspect-ratio: 1;
            object-position: center;
        }

        &__price {
            margin-top: 1rem;
            font-weight: 600;
        }

        .font-light {
            font-weight: 300;
        }

        &:is(:hover) {
            .product-preview__ttl {
                color: red;
            }
        }
    }
</style>