<script setup>
import { onMounted, ref } from 'vue';

const userId = localStorage.getItem("user_id")

const inventories = ref([]);
const getInventory = () => {
  fetch("http://localhost:8080/inventories")
    .then(response => response.json())
    .then(data => {
      inventories.value = data;
      console.log(inventories.value)
    })
    .catch(error => console.log('error', error));
};

const borrow = (inventoryId) => {
  var myHeaders = new Headers();
  myHeaders.append("Content-Type", "application/json");

  var raw = JSON.stringify({
    "user": {
      "id": userId
    },
    "inventory": {
      "id": inventoryId
    }
  });

  var requestOptions = {
    method: 'POST',
    headers: myHeaders,
    body: raw,
    redirect: 'follow'
  };

  fetch("http://localhost:8080/borrow_record", requestOptions)
    .then(response => response.text())
    .then(result => {
      console.log(result);
      getInventory();
    })
    .catch(error => console.log('error', error));
};

const toAvailable = (inventoryId) => {

  console.log("fsdlflsdfj")

  var myHeaders = new Headers();
  myHeaders.append("Content-Type", "application/json");

  var raw = JSON.stringify({
    "status": "AVAILABLE"
  });

  var requestOptions = {
    method: 'PUT',
    headers: myHeaders,
    body: raw,
    redirect: 'follow'
  };

  fetch("http://localhost:8080/inventory/" + inventoryId + "/update-status", requestOptions)
    .then(response => response.text())
    .then(result => {
      console.log(result);
      getInventory();
    })
    .catch(error => console.log('error', error));
}

const getChipColor = (status) => {
  switch (status) {
    case 'AVAILABLE':
      return 'primary';
    case 'ON_LOAN':
      return 'error';
    case 'PROCESSING':
      return 'warning';
    case 'LOST':
    case 'DAMAGED':
    case 'DISPOSED':
      return 'red';
    default:
      return 'primary';
  }
}

onMounted(() => {
  console.log("onMounted inventroy");
  getInventory();
});
</script>

<template>
  <VRow>
    <VCol cols="12">
      <VCard title="Inventory">
        <VCardText>
        </VCardText>
        <VTable height="100%">
          <thead>
            <tr>
              <th class="text-uppercase">
                id
              </th>
              <th class="text-uppercase text-center">
                ISBN
              </th>
              <th class="text-uppercase text-center">
                storeTime
              </th>
              <th class="text-uppercase text-center">
                status
              </th>
              <th class="text-uppercase text-center">
                Action
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="inventroy in inventories" :key="inventroy.id">
              <td>{{ inventroy.id }}</td>
              <td class="text-center">{{ inventroy.book.isbn }}</td>
              <td class="text-center">{{ inventroy.storeTime }}</td>
              <td class="text-center">
                <v-chip
                  :color="getChipColor(inventroy.status)"
                  @click="toAvailable(inventroy.id)">
                  {{ inventroy.status }}
                </v-chip>
              </td>
              <td>
                <VBtn v-if="inventroy.status === 'AVAILABLE'" 
                  @click="borrow(inventroy.id)">
                  Borrow
                </VBtn>
              </td>
            </tr>
          </tbody>
        </VTable>
      </VCard>
    </VCol>
  </VRow>
</template>
