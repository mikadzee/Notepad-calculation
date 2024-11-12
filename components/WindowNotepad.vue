<template>

  <div class="windows-container">
    <section class="window-products">
      <div class="products">
        <p class="error" v-html="error"></p>
        <transition-group name="slide" tag="div">
          <div class="product" v-for="product, index in products" :key="index">
            <p class="title-product">{{ product.title }}</p>
            <p class="price-product">{{ formatPrice(product.price) }}</p>
            <button @click="deleteProduct(index)">-</button>
          </div>
        </transition-group>
      </div>
      <div class="filed-create-product">
        <form  @submit.prevent="addProduct">
          <input type="text" v-model="createProduct.title" placeholder="Название продукта">
          <input type="number" v-model="createProduct.price" placeholder="Цена продукта">
          <input type="submit" value="Добавить" />
        </form>
      </div>
    </section>
  
    <section class="window-calculation">
      <div class="window-goods">
        <div class="calculation-product" v-for="product in products" >
          <p class="title-product">{{ product.title }}</p>
          <p class="price-product">{{ formatPrice(product.price) }}</p>
        </div>
      </div>

      <div ref="costCalculation" class="cost-calculation">
        <p>          
          <transition-group name="slide" tag="div">
            <span v-for="(product, index) in products" :key="index">{{ formatPrice(product.price) }}<br></span>
          </transition-group>
        </p>
        <p class="result" v-if="calculator">=<br>-{{ formatPrice(calculator) }}</p>
      </div>
    </section>
  </div>
</template>

<style lang="less" scoped>
  .windows-container {
    margin: 50px 10px;
    display: flex;
    height: 100vh;
    justify-content: space-between;

    & .window-products {
      width: 55%;
      position: relative;
  
      & .products {
        padding: 10px;
        width: 100%;
        height: 70%;
        overflow-y: auto;
        overflow-x: hidden;
        border-radius: 15px 15px 0px 0px;
        border: 1px solid #ccc;
        border-bottom: none;
        display: flex;
        flex-direction: column;
        gap: 10px;
  
        & > p {
          position: absolute;
          color: red;
          font-size: 20px;
          background: #f7eaea;
        }
  
        & .product {
          margin-bottom: 2px;
          padding: 10px 50px;
          display: flex;
          justify-content: space-between;
          align-items: center;
          border-radius: 5px;
          border: 1px solid #ccc;
  
          & .title-product {
            font-size: 16px;
            font-weight: 500;
          }
  
          & .price-product {
            font-size: 16px;
            font-weight: 500;
          }
        }
  
      }
      & .filed-create-product {
        padding: 10px;
        display: flex;
        width: 100%;
        justify-content: center;
        flex-direction: column;
        gap: 10px;
        border-radius: 0px 0px 15px 15px;
        border: 1px solid #ccc;
  
        input {
          padding: 10px;
          margin-left: 20px;
          border-radius: 5px;  
          border: 1px solid #ccc;
        }
  
        & > button {
          padding: 10px 20px;
          font-size: 20px;
          letter-spacing: 1.5px;
        }
      }
    }

    & .window-calculation {
      width: 40%;
      position: relative;
      height: 100%;
      
      & .window-goods {
        padding: 10px;
        height: 70%;
        overflow-y: auto;
        border-radius: 15px;
        border: 1px solid #ccc;
        display: flex;
        flex-direction: column;
        gap: 10px;
        
        & .calculation-product {
          padding: 10px 50px;
          border: 1px solid #ccc;
          border-radius: 10px;
          display: flex;
          justify-content: space-between;

          
          & .title-product {
            
          }
  
          & .price-product {
            
          }
        }

      }
      & .cost-calculation {
        padding: 10px 20px;
        overflow-y: auto;
        position: absolute;
        border: 1px solid #ccc;
        bottom: 30%;
        left: 2%;
        height: 0px;

        transition: all 1s;
        background: #eeeded;
        border-radius: 10px;

        & > p {          
          margin-block-end: 0em;
          margin-block-start: 0em;
          text-align: center;
          
          & > span {
          transition: transform 0.8s ease, opacity 0.5s ease;
            font-size: 15px;
          }
        }

        & .result {
          text-align: center;
          font-size: 20px;
          font-weight: 800;
          color: rgb(202, 5, 5);
        }
      }
    }
  }

  .slide-enter-active, .slide-leave-active {
    transition: transform 1s ease, opacity 0.5s ease;
  }

  .slide-enter-from {
    transform: translateX(100%);
    opacity: 0;
  }

  .slide-enter-to {
    transform: translateX(0);
    opacity: 1;
  }

  .slide-leave-from {
    transform: translateX(0);
    opacity: 1;
  }

  .slide-leave-to {
    transform: translateX(100%);
    opacity: 0;
  }
</style>

<script lang="ts" setup>

  interface NewProduct {
    title: string;
    price: number;
  }


  let error = ref<string>('');
  let products = reactive<NewProduct[]>([]);
  let costCalculation = ref<HTMLElement | null>(null);
  let createProduct = ref<NewProduct>({
    title: '',
    price: 0
  });

  const calculator = computed(() => {
    if(!products?.length) {
      return false;
    }
    const arrSummary: number[] = [];
    

    products.forEach(e => {
      arrSummary.push(e.price);
    });

    return arrSummary.reduce((acc,num) => acc + num);
  });

  watch(calculator, (newI) => {
    return !products?.length ? costCalculation.value.style.height = '0%' : costCalculation.value.style.height = '20%';
  });
  const validationRules = computed(() => {
    const errors: string[] = [];
    
    if(createProduct.value.title.length <= 3) {
      errors.push('Название продукта не должно быть меньше трех символов!');
    }

    if (createProduct.value.price <= 0) {
      errors.push('Цена продукта не должна быть меньше нуля!')
    }

    return errors.join('<br />')
  })

  const addProduct = () => {
    error.value=''
    if(validationRules.value) {
      error.value = validationRules.value;
      return;
    }

    const newProduct: NewProduct = {
      title: createProduct.value.title,
      price: createProduct.value.price
    }
    products.push(newProduct);

    console.log(products);
    
    createProduct.value = {title: '', price: 0};
  };

  const formatPrice = (price: number) => {
    const formatter = new Intl.NumberFormat('ru-RU', {
      style: 'currency',
      currency: 'RUB',
      minimumFractionDigits: 0,
    });
    return formatter.format(price);
  };

  const deleteProduct = (id: number) => {
    products.splice(id, 1);

    console.log(products)
  };
</script>