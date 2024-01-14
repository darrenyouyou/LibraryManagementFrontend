<script setup>

const userId = localStorage.getItem("user_id")

const inventories = ref([]);

const getBorrowingRecord = () => {

  fetch("http://localhost:8080/borrow_record/user/" + userId)
    .then(response => response.json())
    .then(data => {
      inventories.value = data;
    })
    .catch(error => console.log('error', error));
};

const tsToStr = (unixTimestamp) => {
  const date = new Date(unixTimestamp * 1000);
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0'); // 月份从 0 开始，需要加 1
  const day = String(date.getDate()).padStart(2, '0');
  const hours = String(date.getHours()).padStart(2, '0');
  const minutes = String(date.getMinutes()).padStart(2, '0');
  const seconds = String(date.getSeconds()).padStart(2, '0');
  const formattedDateTime = `${year}-${month}-${day} ${hours}:${minutes}`;
  return (unixTimestamp) ? formattedDateTime : '';
}

const returnBook = (id) => {

  var requestOptions = {
    method: 'PUT',
    redirect: 'follow'
  };

  fetch("http://localhost:8080/borrow_record/" + id + "/return", requestOptions)
    .then(response => response.text())
    .then(result => {
      console.log(result);
      getBorrowingRecord();
    })
    .catch(error => console.log('error', error));
};

onMounted(() => {
  console.log("onMounted borrow record");
  getBorrowingRecord();
});
</script>

<template>
  <VRow>
    <VCol cols="12">
      <VCard title="Borrowing Record">
        <VCardText>
          <!-- ... -->
        </VCardText>
        <VTable height="100%">
          <thead>
            <tr>
              <th class="text-uppercase text-center">
                ISBN 
              </th>
              <th class="text-uppercase text-center">
                Borrow Date
              </th>
              <th class="text-uppercase text-center">
                Return Date
              </th>
              <th class="text-uppercase text-center">
                Action
              </th>
            </tr>
          </thead>

          <tbody>
            <tr v-for="item in inventories" :key="item.id">
              <td>
                {{ item.inventory.book.isbn }}
              </td>
              <td class="text-center">
                {{ tsToStr(item.borrowDate) }}
              </td>
              <td class="text-center">
                {{ tsToStr(item.returnDate) }}
              </td>
              <td>
                <VBtn v-if="item.returnDate === null" 
                  @click="returnBook(item.id)">
                  Return
                </VBtn>
              </td>
            </tr>
          </tbody>
        </VTable>
      </VCard>
    </VCol>
  </VRow>
</template>
