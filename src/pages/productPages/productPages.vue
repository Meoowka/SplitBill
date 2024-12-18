<script>
import {nextTick} from 'vue'
import {mapState, mapActions} from 'vuex'


export default {
  data() {
    return {
      editName: "",
      editPrice: null,
      editId: null
    }
  },
  computed: {
    ...mapState('userStore', {
      user: state => state.user,
    }),
    ...mapState('productStore', {
      products: state => state.products,
    }),
  },
  methods: {
    ...mapActions('productStore', [
      'addProduct',
      'editProduct',
      'removeProduct',
      'editBuyer',
      'toggleUsers',
      'toggleAllUsers'
    ]),
    handleCalculateCost() {
      if (!this.products.length) {
        alert("Сначала добавьте хотя бы один продукт!");
        return;
      }

      this.$router.push("/bill");
    },
    remove(id) {
      this.stopEdit();
      this.removeProduct({id});
    },
    startEdit(id) {
      this.stopEdit();
      this.editId = id;
      const product = this.products.find(p => p.id === id);
      this.editName = product.name;
      this.editPrice = product.price;
      nextTick(() => this.$el.querySelector('#name-inp').focus());
    },
    stopEdit() {
      if (this.editId === null) {
        return;
      }
      this.editProduct({id: this.editId, name: this.editName.trim(), price: this.editPrice || 0});
      this.editId = null;
    },
    add() {
      this.addProduct({name: "", price: null, buyer: 0});
      this.startEdit(this.products[this.products.length - 1].id);
      nextTick(() => {
        const ul = this.$el.querySelector('ul');
        ul.scrollTop = ul.scrollHeight;
        this.$el.querySelector('#name-inp').focus();
      });
    },
    setBuyer(id, buyer) {
      this.editBuyer({id, buyer})
    },
    handleInputKey(e, target, focus = false) {
      if (!['Tab', 'Enter'].includes(e.code)) {
        return;
      }
      e.preventDefault();
      const el = this.$el.querySelector(target);
      focus ? el.focus() : el.click();
    }
  },
  unmounted() {
    this.stopEdit();
  },
}
</script>

<template>
  <div class="container">
    <h1>Продукты</h1>
    <ul v-if="user.length">
      <li
          class="list-item"
          v-for="{id, name, price, buyer, consumers} in products"
          :class="{editing: id === editId}"
          :key="id"
      >
        <div class="details">
          <div class="details-top">
            <div class="info">
              <p v-if="id !== editId" class="name">{{ name }}</p>
              <input
                  v-else
                  v-model="editName"
                  type="text"
                  name="name-inp"
                  id="name-inp"
                  placeholder="Введите названия..."
                  @keydown="(e) => this.handleInputKey(e, '#price-inp', true)"
              />
              <p v-if="id !== editId" class="price">{{ price }}рубликов</p>
              <input
                  v-else
                  v-model="editPrice"
                  type="number"
                  name="price-inp"
                  id="price-inp"
                  placeholder="Введите цену..."
                  @keydown="(e) => this.handleInputKey(e, '.bi-check-lg')"
              />
            </div>
            <div class="buyer-select">
              <i class="bi bi-credit-card-2-back-fill"></i>
              <select
                  name="buyer"
                  class="form-select"
                  @change="(e) => setBuyer(id, +e.target.value)"
                  :value="buyer"
              >
                <option
                    v-for="user in user"
                    :value="user.id">{{ user.name }}
                </option>
              </select>
            </div>
          </div>
          <div class="details-bottom">
            <div class="user-list">
              <div
                  class="user-list-item user-list-select-all"
                  :class="{selected: consumers.length === user.length}"
                  @click="toggleAllUsers({id, users: user})"
              >
                <div class="icon">+</div>
                <p class="user-name">All</p>
              </div>
              <div
                  class="user-list-item"
                  :class="{selected: consumers.includes(user.id)}"
                  v-for="user in user"
                  @click="toggleUsers({id, user: user.id})"
              >
                <div class="icon">{{ user.name[0] }}</div>
                <p class="user-name">{{ user.name }}</p>
              </div>
            </div>
          </div>
        </div>

        <div class="edit-buttons">
          <i
              v-if="id !== editId"
              class="bi bi-pencil-fill i-pointer"
              @click="startEdit(id)"
          ></i>
          <i
              v-else
              class="bi bi-check-lg i-pointer"
              @click="stopEdit()"></i>
          <i class="bi bi-trash-fill i-pointer" @click="remove(id)"></i>
        </div>
      </li>
      <li class="list-btn" @click="add()" :key="-1">
        <v-btn class="bi bi-bag-plus-fill">
          Добавить новый продукт
        </v-btn>
      </li>
    </ul>
    <h4 class="info-message" v-else>Нету добавленных пользователей</h4>
    <v-btn
        v-if="user.length && products.length"
        class="link next-btn"
        @click.prevent="handleCalculateCost"
    >
      Рассчитать стоимость
    </v-btn>
    <router-link
        v-if="!user.length"
        class="link next-btn"
        to="/users"
    >
      <v-btn class="link next-btn">
        Добавить пользователей
      </v-btn>
    </router-link>
  </div>
</template>

<style scoped lang="scss">
@import "./style.scss";
</style>