<template>
  <div class="container">
    <h1 class="title">Domain Information Lookup</h1>
    <form @submit.prevent="fetchDomainInfo">
      <input
        v-model="domainName"
        type="text"
        placeholder="Enter domain name"
        class="input-field"
      />
      <select v-model="infoType" class="select-field">
        <option value="domain">Domain Information</option>
        <option value="contact">Contact Information</option>
      </select>
      <button type="submit" class="lookup-button" :disabled="loading">
        <b-spinner v-if="loading" small type="grow"></b-spinner>
        <span v-if="loading">Loading...</span>
        <span v-else>Lookup</span>
      </button>
    </form>
    <!-- Display Error Message if Exists -->
    <div v-if="errorMessage" class="error-message">
      <p>{{ errorMessage }}</p>
    </div>
    <!-- Display Domain Data if No Error -->
    <div v-if="domainData" class="results-container">
      <h2>Results for {{ domainResponse }}</h2>
      <table class="results-table">
        <thead>
          <tr>
            <th v-for="(header, index) in tableHeaders" :key="index">
              {{ header }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(value, key) in domainData" :key="key">
            <td>{{ key }}</td>
            <td>{{ value }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

const domainName = ref("");
const infoType = ref("domain");
const domainData = ref(null);
const errorMessage = ref(null);
let domainResponse = ref(null);
const tableHeaders = ref([]);
const loading = ref(false);
async function fetchDomainInfo() {
  try {
    const apiUrl = `http://127.0.0.1:8000/whois`;
    loading.value = true;
    domainData.value = null;
    errorMessage.value = null;
    const response = await axios.get(apiUrl, {
      params: {
        domain: domainName.value,
        type: infoType.value,
      },
    });
    domainResponse = domainName.value;
    if (response.data.error) {
      errorMessage.value = response.data.error;
    } else {
      if (infoType.value === "domain") {
        domainData.value = response.data;
        tableHeaders.value = ["Field", "Value"];
      } else if (infoType.value === "contact") {
        domainData.value = response.data;
        tableHeaders.value = ["Field", "Contact Details"];
      }
    }
  } catch (error) {
    console.error("Error fetching domain info:", error);
    errorMessage.value =
      "An error occurred while fetching data. Please try again.";
  } finally {
    loading.value = false;
  }
}
</script>

<style scoped>
.container {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
  background-color: #f7f8fa;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.title {
  font-size: 24px;
  text-align: center;
  margin-bottom: 20px;
}

.input-field,
.select-field {
  width: 100%;
  padding: 10px;
  margin-bottom: 10px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

.lookup-button {
  width: 100%;
  padding: 10px;
  background-color: #3490dc;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.lookup-button:hover {
  background-color: #2779bd;
}

.results-container {
  margin-top: 20px;
}

.results-table {
  width: 100%;
  border-collapse: collapse;
}

.results-table th,
.results-table td {
  padding: 10px;
  border: 1px solid #ddd;
  text-align: left;
}
.error-message {
  color: red;
  background-color: #ffe6e6;
  padding: 10px;
  border: 1px solid red;
  margin-top: 10px;
  border-radius: 5px;
}
</style>
