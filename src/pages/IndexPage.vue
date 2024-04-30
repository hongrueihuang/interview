<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <form @submit.prevent.stop="onSubmit">
          <q-input
            v-model="tempData.name"
            label="姓名"
            ref="nameRef"
            lazy-rule
            :rules="nameRules"
          />
          <q-input
            v-model="tempData.age"
            label="年齡"
            ref="ageRef"
            lazy-rule
            :rules="ageRules"
          />
          <q-btn type="submit" color="primary" class="q-mt-md">{{
            isEdit ? '更新' : '新增'
          }}</q-btn>
        </form>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
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

<script setup lang="ts">
import axios from 'axios';
import { QTableProps, useQuasar } from 'quasar';
import { ref, onMounted } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const $q = useQuasar();
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
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

const nameRef = ref();
const ageRef = ref();

const nameRules = [
  (v) => !!v || '請輸入姓名',
  (v) => (v && v.length <= 10) || '姓名最多10個字',
];

const ageRules = [
  (v) => !!v || '請輸入年齡',
  (v) => (v && v >= 0) || '年齡需大於0',
];

const tempData = ref({
  name: '',
  age: '',
});

const isEdit = ref(false);
const editId = ref('');

onMounted(() => {
  getList();
});

async function onSubmit(): Promise<void> {
  nameRef.value.validate();
  ageRef.value.validate();
  if (nameRef.value.hasError || ageRef.value.hasError) {
    return;
  }
  const { name, age } = tempData.value;
  try {
    if (!isEdit.value) {
      await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest', {
        name,
        age,
      });
    } else {
      await axios.patch(`https://dahua.metcfire.com.tw/api/CRUDTest`, {
        name,
        age,
        id: editId.value,
      });
      isEdit.value = false;
      editId.value = '';
    }
    tempData.value = {
      name: '',
      age: '',
    };
    getList();
    nameRef.value.resetValidation();
    ageRef.value.resetValidation();
  } catch (error) {
    console.error(error);
  }
}

function handleClickOption(btn, data) {
  // ...
  const buttonStatus = btn?.status;
  switch (buttonStatus) {
    case 'edit':
      // ...
      const { name, age } = data;
      tempData.value = {
        name,
        age,
      };
      isEdit.value = true;
      editId.value = data.id;
      break;
    case 'delete':
      handleDelete(data.id);
      break;
    default:
      break;
  }
}

async function getList(): Promise<void> {
  const { data }: { data: { id: string; name: string; age: number }[] } =
    await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a');
  blockData.value = data;
}

function handleDelete(id: string): void {
  $q.dialog({
    title: '提示',
    message: '是否確定刪除該筆資料',
    persistent: true,
    ok: {
      label: '確定',
      color: 'negative',
    },
    cancel: {
      label: '取消',
      color: 'grey-8',
    },
  })
    .onOk(async () => {
      try {
        await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
        getList();
      } catch (error) {
        console.error(error);
      }
    })
    .onCancel(() => {
      console.log('Cancel');
    })
    .onDismiss(() => {
      console.log('I am triggered on both OK and Cancel');
    });
  // try {
  //   await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${id}`);
  //   getList();
  // } catch (error) {
  //   console.error(error);
  // }
}
</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
