<script setup lang="ts">
import { ref, reactive } from "vue";
import axios from "axios";
import { Product } from '../types/product';

const EAN = ref('');
const requestMade = ref(false)
const product = reactive<Product>({
  product_name: '',
  brand: [],
  ingredients: [],
  allergens: [],
  carbon_footprint: '',
  nutriscore_grade: '',
  sugar: '',
  weight: '',
  tags: [],
  image_url: ''
});

const EANrequest = async () => {
  try {
    requestMade.value = true
    const response = await axios.get(`https://world.openfoodfacts.org/api/v0/product/${EAN.value}`);
    const result = response.data;
    console.log(result);
    if (result.product) {
      product.product_name = result.product.product_name;
      product.brand = result.product.brands;
      product.sugar = result.product.nutriscore_data.sugars
      product.weight = result.product.quantity
      product.ingredients = result.product.ingredients.map(ingredient => ingredient.text.replace(/_/g, ' '));
      product.allergens = Array.from(new Set(result.product.allergens_from_ingredients.split(', ')))
        .map((allergen: unknown) => (allergen as string).replace('en:', '')) as string[];   
      product.carbon_footprint = result.product.carbon_footprint_percent_of_known_ingredients   
      if (result.product.nutriscore_grade === 'a') {
        product.nutriscore_grade = '../../public/nutriscore-a.svg'
      };
      if (result.product.nutriscore_grade === 'b') {
        product.nutriscore_grade = '../../public/nutriscore-b.svg'
      };
      if (result.product.nutriscore_grade === 'c') {
        product.nutriscore_grade = '../../public/nutriscore-c.svg'
      };
      if (result.product.nutriscore_grade === 'd') {
        product.nutriscore_grade = '../../public/nutriscore-d.svg'
      };
      if (result.product.nutriscore_grade === 'e') {
        product.nutriscore_grade = '../../public/nutriscore-e.svg'
      };
      const transformedTags = result.product.ingredients_analysis_tags.map(tag => {
        switch (tag) {
          case 'en:palm-oil-free':
            return 'Sans huile de palme';
          case 'en:palm-oil': 
            return 'Huile de palme';
          case 'en:vegan-status-unknown':
            return 'Caractère végétalien inconnu'
          case 'en:vegan':
            return 'Végétalien';
          case 'en:non-vegan': 
            return 'Non Végétalien';
            case 'en:vegetarian-status-unknown':
              return 'Caractère végétarien inconnu'
          case 'en:vegetarian':
            return 'Végétarien';
          case 'en:maybe-vegetarian':
            return `Peut-être végétarien`
          default:
            return tag;
        }
      });
      product.tags = transformedTags
      product.image_url = result.product.image_url;
    }
  } catch (error) {
    console.error(error);
  }
  console.log(product);
};
</script>

<template>
  <input type="text" v-model="EAN">
  <button @click="EANrequest">Rechercher</button>
  <div v-if="requestMade">{{ product.product_name }}</div>
  <img v-if="requestMade" :src="product.image_url" alt="">
  <div v-if="requestMade">{{ Array.isArray(product.brand) ? `Marque : ${product.brand.join(', ')}` : `Marques : ${product.brand}` }}</div> 
  <img v-if="requestMade" :src="product.nutriscore_grade"></img>
  <div v-if="requestMade" class="ingredients">
    <span>Ingrédients :</span>
    <span>{{ product.ingredients.join(', ') }}</span>
  </div>
  <div v-if="requestMade" class="allergens">
    <span>Allergènes :</span>
    <span>{{ product.allergens.join(', ') }}</span>
  </div>
  <div v-if="requestMade">Empreinte carbone : {{ product.carbon_footprint }}g CO²</div>
  <span v-if="requestMade">{{ product.tags.join(', ') }}</span>
  <div v-if="requestMade">Taux de sucre : {{ product.sugar }}g pour 100g</div>
  <div v-if="requestMade">Poids: {{ product.weight }}</div>
</template>

<style scoped>
</style>