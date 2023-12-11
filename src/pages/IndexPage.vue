<script setup>
import axios from 'axios';
import { onMounted, ref } from 'vue';
// 【GET】https://demo.mercuryfire.com.tw:49110/crudTest/a
// 【POST】https://demo.mercuryfire.com.tw:49110/crudTest
// 【PATCH】https://demo.mercuryfire.com.tw:49110/crudTest
// 【DELETE】https://demo.mercuryfire.com.tw:49110/crudTest/{id}

const blockData = ref([]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);

const tempData = ref({
  name: 'test1',
  age: 199,
});

const getData = () => {
  axios
    .get('https://demo.mercuryfire.com.tw:49110/crudTest/a')
    .then((res) => {
      console.log(res);
      blockData.value = res.data.result;
    })
    .catch((err) => {
      console.log(err);
    });
};

const deleteData = (id) => {
  axios
    .delete(`https://demo.mercuryfire.com.tw:49110/crudTest/${id}`)
    .then((res) => {
      console.log(res);
      getData();
    })
    .catch((err) => {
      console.log(err);
    });
};

const patch = async (data) => {
  try {
    const res = await axios.patch(
      'https://demo.mercuryfire.com.tw:49110/crudTest',
      data
    );
    console.log('patch', res);
    await getData();
  } catch (err) {
    console.log('patch', err);
  }
};
const submitBtnStatus = ref('post');
const handleClickOption = async (btn, data) => {
  if (btn.status === 'edit') {
    tempData.value = JSON.parse(JSON.stringify(data));
    submitBtnStatus.value = 'patch';
    console.log('edit', tempData.value);
  } else if (btn.status === 'delete') {
    await deleteData(data.id);
  }
};

const handleSubmit = async () => {
  if (submitBtnStatus.value === 'post') {
    try {
      const res = await axios.post(
        'https://demo.mercuryfire.com.tw:49110/crudTest',
        {
          name: tempData.value.name,
          age: tempData.value.age,
        }
      );
      console.log('submit', res);
      getData();
    } catch (err) {
      console.log('submit', err);
    }
  } else if (submitBtnStatus.value === 'patch') {
    try {
      console.log('tempData.value', tempData.value);
      const res = await axios.patch(
        'https://demo.mercuryfire.com.tw:49110/crudTest',
        {
          id: tempData.value.id,
          name: tempData.value.name,
          age: tempData.value.age,
          createdID: tempData.value.createdID,
        }
      );
      console.error('patch', res);
      getData();
    } catch (err) {
      console.error('patch', err);
    }
  }
};
const onReset = () => {
  tempData.value = {
    name: '',
    age: '',
  };
  submitBtnStatus.value = 'post';
};

const filter = ref();

onMounted(async () => {
  await getData();
});
</script>
<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <!-- quaser form -->
      <div class="q-mb-xl">
        <q-form @submit.prevent="handleSubmit" @reset="onReset">
          <q-input v-model="tempData.name" label="姓名" />
          <q-input v-model.number="tempData.age" label="年齡" />
          <q-btn
            color="primary"
            class="q-mt-md"
            type="submit"
            v-if="submitBtnStatus === 'post'"
            >新增</q-btn
          >
          <q-btn color="primary" class="q-mt-md" type="submit" v-else
            >更新</q-btn
          >
          <q-btn color="primary" class="q-mt-md q-ml-md" type="reset"
            >重置
          </q-btn>
        </q-form>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="tableConfig"
        row-key="id"
        :filter="filter"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:top-right>
          <q-input
            borderless
            dense
            debounce="300"
            v-model="filter"
            placeholder="Search"
          >
            <template v-slot:append>
              <q-icon name="search" />
            </template>
          </q-input>
        </template>
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>
        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
    </div>
  </q-page>
</template>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
