<template>
  <div id="app" class="container-fluid">
    <div class="row">
      <div id="add-form" class="col-md-5 px-0">

        <div class="row">
          <div style="width:90px; height: 75px; margin: 0 auto; background: #1B2428; text-align: center; font-size: 3em; color: #95A5A6" class="mb-0">
            P
          </div>
        </div>

        <form class="form-inline mb-4 " v-on:submit.prevent="addSkuToCart">
          <label class="sr-only" for="sku">SKU</label>
          <input type="text" class="form-control form-control-lg col-md-10 mx-0" id="sku" placeholder="SKU" v-model="addToCart.sku">

          <label class="sr-only" for="qty">Qty</label>
          <input type="text" class="form-control form-control-lg form-qty col-md-1 mx-0" id="qty" placeholder="QTY" style="text-align:center" v-model="addToCart.qty">

          <button class="add-btn btn btn-lg btn-primary col-md-1">
            <i class="ion-android-add"></i>
          </button>
        </form>

        <div id="totals">
          <div class="total mb-0 py-4">
            {{ cartItems.meta.display_price.with_tax.formatted }}
          </div>

          <div>
            <button class="btn btn-lg btn-block btn-buy py-3">BUY</button>
          </div>
        </div>
      </div>
      <div id="product-list" class="col-md-7">
        <div v-for="item in cartItems.data" class="row item-row py-3">
          <div class="col-md-2">
            <img src="http://lorempixel.com/92/92" class="item-thumb mx-auto"/>
          </div>
          <div class="col-md-8">
            <strong>{{ item.name }}</strong>
            <p class="item-qty">{{ item.sku }} x{{ item.quantity }}</p>
            <h5>{{ item.meta.display_price.with_tax.value.formatted }}</h5>
          </div>
          <div class="col-md-2">
            <button v-on:click.prevent="deleteCartItem(item.id)" class="delete-btn">
              <i class="ion-android-close"></i>
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { gateway as MoltinGateway } from '@moltin/sdk'

const Moltin = MoltinGateway({
  client_id: 'j6hSilXRQfxKohTndUuVrErLcSJWP15P347L6Im0M4'
})

export default {
  name: 'app',
  data () {
    return {
      cartItems: {meta:{display_price:{with_tax:{formatted: 0}}}},
      addToCart: {
        sku: '',
        qty: 1
      }
    }
  },
  beforeMount () {
    this.populateCart();
  },
  methods: {
    populateCart: function() {
      Moltin.Cart.Items().then((response) => {
        this.cartItems = response
      })
    },
    addSkuToCart: function () {
      Moltin.Products.Filter({
      eq: {
        sku: this.addToCart.sku
      }
      }).Limit(1).All().then((response) => {

        if (response.data.length != 1) {
          console.log(response.data)
          alert('Not found!');
          return
        }

        Moltin.Cart.AddProduct(response.data[0].id, this.addSkuToCart.qty).then((response) => {
          this.populateCart();
        })
      });

      this.addToCart.sku = '';
    },
    deleteCartItem: function(itemId) {
      Moltin.Cart.RemoveItem(itemId).then((response) => {
        this.populateCart();
      })

    }
  }
}
</script>

<style>
  body {
    background-color: #2B3E50;
    color: #fff;
  }

  .btn, .btn-lg, .form-control {
    border-radius: 0;
  }

  #app {
    position: relative
  }

  #add-form {
    background-color: #263238;
  } 

  #product-list {}
  #totals {}

  .total {
    background-color: #1B2428;
    font-size: 4em;
    text-align: center;
    color: #626668;
    font-weight: bold;
  }

  .item-row {
    border-bottom: 2px solid #233442;
  }

  .add-btn {
    cursor: pointer;
  }

  .delete-btn {
    background-color: #2B3E50;
    border: 0;
    color: #fff;
    display: block;
    z-index: 0;
    display: inline-block;
    padding: 22px;
    width: 42px;
    border-radius: 4px;
    list-style: none;
    text-align: center;
    font-weight: normal;
    font-size: 32px;
    cursor: pointer;
  }

  .item-qty {
    font-size: 0.9em;
    color: #94A4A5;
  }

  .btn-buy {
    display: block;
    width: 100%;
    background-color: #26CC6E;
    color: #fff;
    font-weight: bold;
  }

  .form-qty {
    background-color: #F29A26;
    color: #fff;
  }

  .form-control-lg, .btn-lg {
    padding: 1rem 1rem;
    border: 0;
  }

  .item-thumb{
    height: 100%;
    max-width: 100%;
    max-height: 100%;
  }
</style>
