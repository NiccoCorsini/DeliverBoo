<template>
  <div v-if="clientToken" class="container">
    <div class="col-12">
      <div>
        <div>
          <form @submit.prevent="payWithCreditCard">
            <div class="row">
              <div class="mb-3 col-md-6 col-sm-12">
                <label for="customerName" class="form-label">Nome</label>
                <input
                  type="text"
                  v-model="name"
                  class="form-control"
                  id="customerName"
                  required="required"
                  maxlength="20"
                />
              </div>
              <div class="mb-3 col-md-6 col-sm-12">
                <label for="customerSurname" class="form-label">Cognome</label>
                <input
                  type="text"
                  v-model="surname"
                  class="form-control"
                  id="customerSurname"
                  required="required"
                  maxlength="20"
                />
              </div>
            </div>

            <div class="row">
              <div class="mb-3 col-md-6 col-sm-12">
                <label for="customerMail" class="form-label">Email</label>
                <input
                  type="email"
                  v-model="email"
                  class="form-control"
                  id="customerMail"
                  required="required"
                />
              </div>
              <div class="mb-3 col-md-6 col-sm-12">
                <label for="customerPhone" class="form-label">Telefono</label>
                <input
                  type="tel"
                  v-model="phone"
                  class="form-control"
                  id="customerPhone"
                  required="required"
                />
              </div>
            </div>

            <div class="mb-3">
              <label for="customerAddress" class="form-label">Indirizzo</label>
              <input
                type="text"
                v-model="address"
                class="form-control"
                id="customerAddress"
                minlength="5"
                maxlength="255"
                required="required"
              />
            </div>

            <div class="row">
              <div class="mb-3 col-md-6 col-sm-12">
                <label for="customerZipCode" class="form-label">CAP</label>
                <input
                  type="text"
                  v-model="zip_code"
                  class="form-control"
                  id="customerZipCode"
                  minlength="5"
                  maxlength="5"
                  required="required"
                />
              </div>
              <div class="mb-3 col-md-6 col-sm-12">
                <label for="customerCity" class="form-label">Città</label>
                <input
                  type="text"
                  v-model="city"
                  class="form-control"
                  id="customerCity"
                  required="required"
                />
              </div>
            </div>
            <div class="form-group">
              <label for="amount">Totale</label>
              <div class="">
                <div class="input-group-prepend"></div>
                <div
                  id="amount"
                  class="mt-2 price"
                  placeholder="Totale"
                  :value="amount()"
                >
                  {{ amount().toFixed(2) }} €
                </div>
              </div>
            </div>
            <hr />
            <div class="form-group">
              <label for="creditCardNumber">Numero carta di credito</label>
              <div
                id="creditCardNumber"
                name="creditCardNumber"
                class="form-control"
              ></div>
            </div>
            <div class="form-group">
              <div class="row">
                <div class="col-6">
                  <label>Data di scadenza</label>
                  <div id="expireDate" class="form-control"></div>
                </div>
                <div class="col-6">
                  <label>CVV</label>
                  <div id="cvv" class="form-control"></div>
                </div>
              </div>
            </div>
            <div class="cards">
              <img src="../../assets/img/visa.png" alt="visa" />
              <img src="../../assets/img/mastercard.png" alt="mastercard" />
              <img src="../../assets/img/americanexpress.png" alt="amex" />
            </div>

            <div v-if="nonce || error" class="advises mb-2 mt-2">
              <div class="alert alert-success" v-if="nonce">
                Il pagamento è andato a buon fine.
              </div>
              <div class="alert alert-danger" v-if="error" v-show="!nonce">
                Il pagamento è stato respinto. Riprova.
              </div>
            </div>
            <button
              v-if="nonce == ''"
              class="btn btn-block text-white mt-3"
              type="submit"
            >
              Paga
            </button>
          </form>
        </div>
      </div>
    </div>
  </div>
  <div v-else class="loader-container">
    <Loader />
  </div>
</template>

<script>
import braintree from "braintree-web";
import axios from "axios";

import Loader from "./Loader.vue";

export default {
  created() {
    this.getClientToken();
  },

  beforeUpdate() {
    if (this.nonce != "" || this.error != "") {
      this.getOrderInfo();
    }
    if (this.nonce != "") {
      this.$emit("orderPassed", true);
    }

    if (this.clientToken != "") {
      this.braintreeSystem();
    }
    if (this.orderObj.name) {
      this.sendOrder();
    }
  },

  updated() {
    if (this.orderPassed) {
      localStorage.setItem("orderdetails", JSON.stringify({}));
      localStorage.setItem("cart", JSON.stringify([]));
    }
  },

  mounted() {},

  components: {
    Loader,
  },

  data() {
    return {
      hostedFieldInstance: false,
      nonce: "",
      error: "",
      clientToken: "",

      name: "",
      surname: "",
      city: "",
      zip_code: "",
      address: "",
      email: "",
      phone: "",
      status: false,
      foods: {},
      orderPassed: false,

      rerender: true,
      render: 0,

      orderObj: {},

      cart: [],
    };
  },

  methods: {
    payWithCreditCard() {
      if (this.hostedFieldInstance) {
        this.hostedFieldInstance
          .tokenize()
          .then((payload) => {
            this.nonce = payload.nonce;
          })
          .catch((err) => {
            this.error = err.message;
          });
      }
      if (this.nonce) {
        this.status = true;
      }
    },

    braintreeSystem() {
      braintree.client
        .create({
          authorization: this.clientToken,
        })
        .then((clientInstance) => {
          let options = {
            client: clientInstance,
            styles: {
              input: {
                "font-size": "16px",
                "font-family": "sans-serif",
              },
            },
            fields: {
              number: {
                selector: "#creditCardNumber",
                placeholder: "Inserisci la carta di credito",
              },
              cvv: {
                selector: "#cvv",
                placeholder: "Inserisci il CVV",
              },
              expirationDate: {
                selector: "#expireDate",
                placeholder: "00 / 0000",
              },
            },
          };
          return braintree.hostedFields.create(options);
        })
        .then((hostedFieldInstance) => {
          this.hostedFieldInstance = hostedFieldInstance;
        });
    },
    getOrderInfo() {
      if (this.nonce) {
        this.status = true;
      }

      this.orderObj = {
        name: this.name,
        surname: this.surname,
        phone: this.phone,
        email: this.email,
        address: this.address,
        zip_code: this.zip_code,
        city: this.city,
        status: this.status,
        food: this.itemQnt(),
        amount: this.amount(),
        transation: this.nonce,
      };

      if (!localStorage.getItem("orderdetails")) {
        localStorage.setItem("orderdetails", JSON.stringify({}));
      }

      localStorage.setItem("orderdetails", JSON.stringify(this.orderObj));
    },
    itemQnt() {
      const items = JSON.parse(localStorage.getItem("cart"));

      const itemsId = [];

      let restaurantID = 0;

      items.forEach((element) => {
        itemsId.push(element.id);
        restaurantID = element.restaurant_id;
      });

      const final = {
        items: itemsId,
        restaurant_id: restaurantID,
      };

      return final;
    },
    amount() {
      const array = [];

      JSON.parse(localStorage.getItem("cart")).forEach((element) => {
        array.push(element.price);
      });

      let amount = 0;

      array.forEach((v) => {
        amount += v;
      });

      return amount;
    },
    sendOrder() {
      axios
        .get("http://127.0.0.1:8000/api/orders/get", {
          params: {
            order: localStorage.getItem("orderdetails"),
          },
        })
        .then((res) => {
          this.orderPassed = res.data;

          if (res.data) {
            localStorage.setItem("cart", JSON.stringify([]));
            localStorage.setItem("orderdetails", JSON.stringify({}));
          }
        });
    },

    getClientToken() {
      axios.get("http://127.0.0.1:8000/api/orders/generate").then((res) => {
        this.clientToken = res.data.token;
      });
    },
  },
};
</script>
<style scoped lang="scss">
@import "node_modules/bootstrap/scss/bootstrap.scss";
@import "@/style/vars.scss";

#creditCardNumber,
#expireDate,
#cvv {
  margin: 10px 0px;
}

.price {
  font-size: 1.5rem;
  font-weight: bold;
  color: $btn-color;
}

.form-control {
  height: 35px;
}

.btn {
  width: 100%;
  background: $btn-color;
}
.cards {
  display: flex;
  justify-content: center;
  margin-top: 20px;
}

.loader-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
