<template>
  <div id="app">
    <Header
      ><Button :isDisabled="buttonDisabled" @button-click="setRecords"
        >Показать пользователей</Button
      ></Header
    >
    <Loader v-if="loading" />
    <Error v-if="errored">Произошла ошибка: {{ errorMessage }}</Error>
    <DataTable
      v-if="tableShown"
      :recordsProp="records"
      :columnsProp="columns"
      @record-click="showOpenedRecord"
    />
    <Overlay v-if="popupShown" @overlay-click="togglePopup" />
    <Popup v-if="popupShown" @close-click="togglePopup">
      <p v-for="field in popupFields" :key="`popup${popupFields.indexOf(field)}`">
        {{ `${field.name}: ${openedRecord[field.dataProp]}` }}
      </p>
    </Popup>
  </div>
</template>

<script>
import axios from 'axios';
import Popup from './components/Popup';
import Error from './components/Error';
import DataTable from './components/DataTable.vue';
import Header from './components/Header.vue';
import Loader from './components/Loader.vue';
import Button from './components/Button.vue';
import Overlay from './components/Overlay.vue';
import { DATA_API_URL } from './config/index.js';

export default {
  components: {
    Error,
    DataTable,
    Button,
    Overlay,
    Header,
    Loader,
    Popup,
  },
  data() {
    return {
      records: [],
      columns: [
        { name: 'Full name', dataProp: 'fullname' },
        { name: 'Username', dataProp: 'uname' },
        { name: 'Company', dataProp: 'company' },
        { name: 'E-mail', dataProp: 'email' },
        { name: 'State', dataProp: 'address.state' },
      ],
      popupFields: [
        { name: 'Full name', dataProp: 'fullname' },
        { name: 'Street address:', dataProp: 'address.streetAddress' },
        { name: 'City', dataProp: 'address.city' },
        { name: 'State', dataProp: 'address.state' },
        { name: 'Zip', dataProp: 'address.zip' },
      ],
      // columns: [
      //   { name: 'Name', dataProp: 'name' },
      //   { name: 'Username', dataProp: 'username' },
      //   { name: 'Street', dataProp: 'address.street' },
      //   { name: 'Email', dataProp: 'email' },
      //   { name: 'Latitude', dataProp: 'address.geo.lat' },
      //   { name: 'Longitude', dataProp: 'address.geo.lng' },
      // ],
      // popupFields: [
      //   { name: 'Name', dataProp: 'name' },
      //   { name: 'Username', dataProp: 'username' },
      //   { name: 'Email', dataProp: 'email' },
      //   { name: 'Street', dataProp: 'address.street' },
      //   { name: 'City', dataProp: 'address.city' },
      //   { name: 'Zipcode', dataProp: 'address.zipcode' },
      //   { name: 'Latitude', dataProp: 'address.geo.lat' },
      //   { name: 'Longitude', dataProp: 'address.geo.lng' },
      // ],
      popupShown: false,
      loading: false,
      buttonDisabled: false,
      tableShown: false,
      errored: false,
      errorMessage: '',
      openedRecord: {},
    };
  },
  methods: {
    togglePopup() {
      this.popupShown = !this.popupShown;
    },
    showOpenedRecord(record) {
      this.togglePopup();
      this.openedRecord = record;
    },
    setRecords() {
      this.errored = false;
      this.loading = true;
      this.buttonDisabled = true;
      this.tableShown = false;
      axios
        .get(DATA_API_URL)
        .then(response => {
          this.records = response.data;
          this.tableShown = true;
        })
        .catch(error => {
          this.errored = true;
          this.errorMessage = error.message;
        })
        .finally(() => {
          this.loading = false;
          this.buttonDisabled = false;
        });
    },
  },
  created() {
    const onEscape = e => {
      if (e.keyCode === 27) {
        this.popupShown = false;
      }
    };
    document.addEventListener('keydown', onEscape);
    this.$once('hook:destroyed', () => {
      document.removeEventListener('keydown', onEscape);
    });
  },
};
</script>

<style>
body {
  margin: 0;
  width: 100%;
  height: 100%;
  box-sizing: border-box;
}

#app {
  min-width: 1024px;
  display: flex;
  flex-direction: column;
  width: 100%;
  height: 100%;
  text-align: center;
  background-color: #fff;
}
</style>
