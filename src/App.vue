<template>
  <div class="container">
    <app-alert :alert="alert" @close="alert = null"></app-alert>
    <form class="card" @submit.prevent="createPerson">
      <h2>Работа с базой данных</h2>
      <div class="form-control">
        <label for="name">Введите имя</label>
        <input type="text" id="name" v-model.trim="name" />
      </div>
      <button class="btn primary" :disabled="name.length === 0">
        Создать человек
      </button>
    </form>
    <app-loader v-if="loading"></app-loader>

    <AppPeopleList :people="people" @load="loadPeople" @remove="removePeople" />
  </div>
</template>

<script>
import AppPeopleList from "./components/AppPeopleList.vue";
import AppAlert from "./components/AppAlert.vue";
import AppLoader from "./components/AppLoader.vue";
import axios from "axios";

export default {
  components: {
    AppPeopleList,
    AppAlert,
    AppLoader,
  },
  data() {
    return {
      name: "",
      alert: null,
      loading: false,
      people: [],
    };
  },
  methods: {
    async createPerson() {
      const response = await fetch(
        "https://vue-http-b7a31-default-rtdb.firebaseio.com/people.json",
        {
          method: "POST",
          headers: {
            "Content-Type": "aplicaion/json",
          },
          body: JSON.stringify({
            firstName: this.name,
          }),
        }
      );
      const firebaseData = await response.json();
      this.people.push({
        firstName: this.name,
        id: firebaseData.name,
      });
      this.name = "";
      console.log(firebaseData);
    },
    async loadPeople() {
      try {
        const { data } = await axios.get(
          "https://vue-http-b7a31-default-rtdb.firebaseio.com/people.json"
        );
        if (!data) {
          throw new Error("Список людей пуст ");
        }
        setTimeout(() => {
          this.people = Object.keys(data).map((key) => {
            return {
              id: key,
              ...data[key],
            };
          });
          this.loading = false;
        }, 1500);
      } catch (e) {
        this.alert = {
          type: "danger",
          title: "ошибка",
          text: e.message,
        };
        this.loading = false;
      }
    },
    async removePeople(id) {
      try {
        const name = this.people.find((person) => person.id === id).firstName;
        await axios.delete(
          `https://vue-http-b7a31-default-rtdb.firebaseio.com/people/${id}.json`
        );
        this.people = this.people.filter((person) => person.id !== id);
        this.alert = {
          type: "primary",
          title: "Успешно",
          text: `Пользователь  с именем ${name} был успешно удален`,
        };
      } catch (e) {}
    },
  },
  mounted() {
    this.loadPeople();
    this.loading = true;
  },
};
</script>

<style scoped lang="css"></style>
