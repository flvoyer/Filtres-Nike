<script setup>
import { computed, ref } from 'vue';
import jsonData from './assets/produits.json';
import ProductPreview from './components/ProductPreview.vue';

const allProducts = jsonData.filter(product => product.article !== "");
let products = allProducts;

let sexFilters = ref([]);
let priceFilters = ref([]);
let sportFilters = ref([]);
let colorFilters = ref([]);
let productsCount = ref(0);

let filterMenuIsVisible = false;

const genders = ['Homme', 'Femme', 'Mixte'];
const prices = [
    {
        value: '0-50',
        label: 'Moins de 50 €'
    },
    {
        value: '50-100',
        label: 'Entre 50€ et 100€'
    },
    {
        value: '100-150',
        label: 'Entre 100€ et 150€'
    },
    {
        value: '>150',
        label: 'Plus de 150€'
    },
];
const colors = ['pourpre', 'noir', 'rouge', 'orange', 'bleu', 'blanc', 'marron', 'vert', 'jaune', 'multicolore', 'gris'];
const sports = ['Football', 'Basket', 'Running'];

/**
 * Filtre les produits
 */
function filteredProducts() {
    products = allProducts;

    // TRI PAR SEXE
    if (sexFilters.value.length > 0) {
        products = products.filter(product => sexFilters.value.includes(product.sexe));
    }

    // TRI PAR SPORT
    if (sportFilters.value.length > 0) {
        products = products.filter(product => sportFilters.value.includes(product.sport));
    }

    // TRI PAR PRIX
    if (priceFilters.value.length > 0) {
        let productsToKeep = [];
        
        products.forEach(product => {
            let shoeInRange = false;
            const shoePrice = parseInt(product.prix.split(' €')[0]);

            priceFilters.value.forEach(priceRange => {
                if (shoeInRange) {
                    return;
                }

                if (priceRange.match(/.*-.*/)) {
                    priceRange = priceRange.split('-');
                    const priceRangeStart = priceRange[0];
                    const priceRangeEnd = priceRange[1];
                    
                    if (shoePrice >= priceRangeStart && shoePrice <= priceRangeEnd) {
                        shoeInRange = true;
                    }
                } else { // >150
                    priceRange = priceRange.split('>')[1];
                    if (shoePrice >= priceRange) {
                        shoeInRange = true;
                    }
                }
            });
            shoeInRange ? productsToKeep.push(product) : null;
        });
        products = productsToKeep;
    }

    // TRI PAR COULEURS
    if (colorFilters.value.length > 0) {
        let productsToKeep = [];

        products.forEach(product => {
            const productColors = product.couleur.replace(' ', '').split(',');

            if (colorFilters.value.some(color => productColors.includes(color))) {
                productsToKeep.push(product);
            }
        });
        products = productsToKeep;
    }
    
    productsCount.value = products.length;

    return products;
}

/**
 * Génère le "headline" en fonction des filtres sélectionnés
 */
function generateHeadline() {
    let headline = "Nouveautés";

    sexFilters.value.forEach((filter) => headline += " " + filter);
    priceFilters.value.forEach((filter) => headline += " " + filter);
    colorFilters.value.forEach((filter) => headline += " " + filter);
    sportFilters.value.forEach((filter) => headline += " " + filter);

    return headline;
}

function showHideFilters () {
    const filterMenu = document.querySelector('.product-filters');

    if (filterMenuIsVisible) {
        filterMenu.classList.remove('visible');
    } else {
        filterMenu.classList.add('visible');
    }
    filterMenuIsVisible = !filterMenuIsVisible;
}
</script>

<template>
    <header class="header">
        <h1 class="h1 headline">{{ generateHeadline() }} <span>({{ productsCount }} produits)</span></h1>
        <button class="header__filter-btn" @click="showHideFilters">Filtres</button>
    </header>
    <main class="container">

        <div class="product-filtering">
            <aside class="product-filters">
                <button class="closeFilters" @click="showHideFilters">Cacher les filtres</button>
                <div class="filter-category">
                    <div class="filter-category__ttl">
                        Sexe <span v-if="sexFilters.length > 0">({{ sexFilters.length }})</span>
                    </div>

                    <template v-for="gender in genders" :key="gender">
                        <label :for="gender" class="form-grp">
                            <input type="checkbox" :name="gender" :id="gender" v-model="sexFilters" :value="gender">
                            {{ gender }}
                        </label>
                    </template>
                </div>

                <div class="filter-category">
                    <div class="filter-category__ttl">
                        Rechercher par prix <span v-if="priceFilters.length > 0">({{ priceFilters.length }})</span>
                    </div>

                    <template v-for="priceRange in prices" :key="priceRange">
                        <label :for="priceRange.value" class="form-grp">
                            <input type="checkbox" :id="priceRange.value" v-model="priceFilters" :value="priceRange.value">
                            {{ priceRange.label }}
                        </label>
                    </template>
                </div>

                <div class="filter-category">
                    <div class="filter-category__ttl">
                        Couleur <span v-if="colorFilters.length > 0">({{ colorFilters.length }})</span>
                    </div>

                    <div class="colors-bubbles">
                        <template v-for="color in colors" :key="color">
                            <label :for="'color-' + color">
                                <input type="checkbox" :id="'color-' + color" :value="color" v-model="colorFilters">
                                <div :class="'bubble bubble--' + color"></div>
                                {{ color.charAt(0).toUpperCase() + color.slice(1) }}
                            </label>
                        </template>
                    </div>
                </div>

                <div class="filter-category">
                    <div class="filter-category__ttl">
                        Sports <span v-if="sportFilters.length > 0">({{ sportFilters.length }})</span>
                    </div>

                    <template v-for="sport in sports" :key="sport">
                        <label :for="sport" class="form-grp">
                            <input type="checkbox" :id="sport" v-model="sportFilters" :value="sport">
                            {{ sport }}
                        </label>
                    </template>
                </div>
            </aside>

            <ul class="product-grid">
                <template v-for="product in filteredProducts()" :key="product.article">
                    <ProductPreview :product="product"></ProductPreview>
                </template>
            </ul>
        </div>
    </main>
</template>

<style lang="scss">
    .header {
        @extend .container;
        display: flex;
        justify-content: space-between;
        align-items: center;
        margin-bottom: 2rem;
        
        &__filter-btn {
            margin: 0;
            padding: 0.5rem 1rem;
            border: none;
            border-radius: 0.5rem;
            background-color: rgb(76, 201, 194);
            color: white;
            transition: 0.3s background-color ease-in-out;
            cursor: pointer;

            &:is(:hover, :active, :focus) {
                background-color: rgb(43, 116, 112);
            }
        }
    }

    .headline {
        font-size: 2rem;
    }

    .container {
        max-width: 1120px;
        margin: 2rem auto;
        padding: 0 2rem;
    }

    .product-filters {
        position: fixed;
        inset: 0;
        overflow-y: scroll;
        transform: translate(-100%);
        background-color: white;
        padding: 4rem 0;
        transition: 0.3s transform ease-in-out;
        width: 100%;

        &.visible {
            transform: translate(0);
        }

        .closeFilters {
            position: absolute;
            top: 1rem;
            right: 1rem;
            margin: 0;
            padding: 0.5rem 1rem;
            border: none;
            border-radius: 0.5rem;
            background-color: rgb(76, 201, 194);
            color: white;
            transition: 0.3s background-color ease-in-out;
            cursor: pointer;

            &:is(:hover, :active, :focus) {
                background-color: rgb(43, 116, 112);
            }
        }
    }

    .product-filtering {
        display: grid;
        grid-template-columns: 1fr;
        gap: 4rem;
    }

    .product-grid {
        padding: 0;
        margin: 0;
        list-style: none;

        display: grid;
        grid-template-columns: 1fr;
        gap: 4rem 1rem;
    }

    .form-grp {
        display: block;
        padding: .5rem 0;
        
        &:first-of-type {
            margin-top: 1rem;
        }
    }

    .filter-category {
        padding: 1rem;

        & + .filter-category {
            margin-top: 2rem;
        }

        &__ttl {
            font-weight: bold;
        }
    }

    .colors-bubbles {
        display: grid;
        grid-template-columns: repeat(3, 33%);
        gap: .5rem;
        margin-top: 1rem;
        text-align: center;

        .bubble {
            width: 25px;
            height: 25px;
            border-radius: 25px;
            border: 4px dashed transparent;
            margin: 0 auto .5rem auto;
            position: relative;

            // VARIANTS 
            &--pourpre {
                background-color: rebeccapurple;
            }
            &--noir {
                background-color: black;
            }
            &--rouge {
                background-color: red;
            }
            &--orange {
                background-color: orange;
            }
            &--bleu {
                background-color: blue;
            }
            &--blanc {
                background-color: white;
                border: 1px solid lightgray;
            }
            &--marron {
                background-color: rgb(124, 51, 18);
            }
            &--vert {
                background-color: green;
            }
            &--jaune {
                background-color: yellow;
            }
            &--multicolore {
                background-color: black;
            }
            &--gris {
                background-color: rgb(116, 116, 116);
            }
        }

        input[type=checkbox] {
            opacity: 0;
        }

        input:checked + .bubble {
            &::after {
                content: '';
                background: center/cover url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSIyNCIgaGVpZ2h0PSIyNCIgdmlld0JveD0iMCAwIDI0IDI0Ij48cGF0aCBkPSJNMjAuMjg1IDJsLTExLjI4NSAxMS41NjctNS4yODYtNS4wMTEtMy43MTQgMy43MTYgOSA4LjcyOCAxNS0xNS4yODV6Ii8+PC9zdmc+");
                width: 15px;
                height: 15px;
                z-index: 10;
                position: absolute;
                left: 50%;
                top: 50%;
                transform: translate(-50%, -50%);
            }
        }
    }

    // RESPONSIVE

    @media (min-width: 576px) {
        .product-grid {
            grid-template-columns: repeat(2, 1fr);
        }
    }

    @media (min-width: 992px) {
        .product-filters {
            position: static;
            padding: 0;
            transform: translate(0);
            overflow: visible;

            .closeFilters {
                display: none;
            }
        }
        .product-filtering {
            grid-template-columns: 200px 1fr;
        }
        .product-grid {
            grid-template-columns: repeat(3, 1fr);
        }

        .header__filter-btn {
            display: none;
        }

        .filter-category {
            border-top: 1px solid lightgray;
            padding: 1rem 0;
        }
    }
</style>