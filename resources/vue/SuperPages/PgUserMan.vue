<script setup lang="ts">
import axios from 'axios';
import { ref, onMounted } from 'vue';
import { timeGreetings, UserDataInterface } from '../AppCommon';
import { useApiStore, useMainStore } from '../AppState';

import CmpToast from '../Components/CmpToast.vue';
import CmpLayout from '../Components/CmpLayout.vue';

import Dialog from '../volt/Dialog.vue';
import DataTable from '../volt/DataTable.vue';
import Column from 'primevue/column';
import InputText from '../volt/InputText.vue';
import { FilterMatchMode } from '@primevue/core/api';

import DialogUserMan from '../DialogComponents/DialogUserMan.vue';

const props = defineProps<{
    appName: string;
    greetings: string;
    expandedKeysProps: string;
}>();
const api = useApiStore();
const main = useMainStore();
const timeGreet = timeGreetings();
const toastchild = ref<typeof CmpToast>();

const userListData = ref(Array<UserDataInterface>());
const loading = ref<boolean>(false);

const filters = ref({
    global: { value: null, matchMode: FilterMatchMode.CONTAINS },
    username: { value: null, matchMode: FilterMatchMode.CONTAINS },
    name: { value: null, matchMode: FilterMatchMode.CONTAINS },
    roles_array: { value: null, matchMode: FilterMatchMode.CONTAINS },
    permissions_array: { value: null, matchMode: FilterMatchMode.CONTAINS },
});

const getUserListData = () => {
    loading.value = true;
    axios
        .post(api.getUserList)
        .then((response) => {
            userListData.value = response.data;
            loading.value = false;
        })
        .catch((error) => {
            toastchild.value?.toastDisplay({
                severity: 'error',
                summary: error.response.data.title,
                detail: error.response.data.message,
                response: error,
            });
            loading.value = false;
        });
};

const showViewButton = (data: string | null | undefined): boolean => {
    if (data !== '' && data !== null && data !== undefined) {
        return true;
    } else {
        return false;
    }
};

const dialogOpen = ref<boolean>(false);
const dialogData = ref<UserDataInterface | null>(null);
const dialogHeader = ref<string>('Create User');

const openEditUserDialog = (data: UserDataInterface | null) => {
    dialogOpen.value = true;
    dialogData.value = data;
    if (data === null) {
        dialogHeader.value = 'Create User';
    } else {
        dialogHeader.value = 'Edit User';
    }
};

onMounted(() => {
    getUserListData();
    main.updateExpandedKeysMenu(props.expandedKeysProps);
});
</script>

<template>
    <CmpLayout>
        <CmpToast ref="toastchild" />
        <Dialog v-model:visible="dialogOpen" modal :header="dialogHeader">
            <DialogUserMan
                v-model:dialogOpen="dialogOpen"
                :dialogData="dialogData"
                :dialogTypeCreate="dialogData === null ? true : false"
                @closeDialog="getUserListData()"
            />
        </Dialog>
        <div
            class="my-2 md:my-3 mx-2 md:mx-5 p-3 md:p-5 bg-surface-200 dark:bg-surface-800 rounded-lg drop-shadow-lg"
        >
            <div class="flex flex-col md:flex-row gap-2 md:gap-0">
                <div class="flex flex-col w-full my-auto">
                    <h2 class="title-font font-bold">
                        {{ timeGreet + greetings }}
                    </h2>
                    <h3 class="title-font">User Role Management</h3>
                </div>
                <div class="flex justify-end w-full my-auto mt-2 md:mt-0">
                    <UButton size="xl" label="Create User" @click="openEditUserDialog(null)" />
                </div>
            </div>
        </div>
        <div
            class="my-2 md:my-3 mx-2 md:mx-5 p-3 md:p-5 bg-surface-200 dark:bg-surface-800 rounded-lg drop-shadow-lg overflow-x-auto"
        >
            <DataTable
                v-model:filters="filters"
                class="p-datatable-sm editable-cells-table"
                :value="userListData"
                showGridlines
                :loading="loading"
                paginator
                :rows="10"
                paginatorTemplate="CurrentPageReport FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink RowsPerPageSelect"
                :rowsPerPageOptions="[10, 20, 50, 100]"
                :globalFilterFields="['username', 'name']"
                filterDisplay="menu"
            >
                <template #header>
                    <div class="flex flex-col sm:flex-row gap-2 justify-between">
                        <div class="flex w-full">
                            <InputText
                                v-model="filters['global'].value"
                                placeholder="Search by username or name"
                                class="p-inputtext-sm w-full"
                            />
                        </div>
                    </div>
                </template>
                <template #footer>
                    <div class="flex text-sm">Total records: {{ userListData.length }}</div>
                </template>
                <template #empty>
                    <div class="flex justify-center">No data found</div>
                </template>
                <template #loading>
                    <i class="pi pi-spin pi-spinner mr-2.5"></i> Loading data. Please wait.
                </template>
                <Column field="action" header="Actions" class="text-sm">
                    <template #body="slotProps">
                        <div v-if="showViewButton(slotProps.data.id)" class="flex justify-center">
                            <UButton size="xl" @click="openEditUserDialog(slotProps.data)"
                                ><i class="pi pi-angle-double-right"
                            /></UButton>
                        </div>
                    </template>
                </Column>
                <Column field="username" header="User Name" class="text-sm">
                    <template #body="slotProps">
                        <div class="text-center">
                            {{ slotProps.data.username }}
                        </div>
                    </template>
                    <template #filter="{ filterModel, filterCallback }">
                        <InputText
                            v-model="filterModel.value"
                            class="w-full"
                            placeholder="Search by username"
                            @input="filterCallback()"
                        />
                    </template>
                </Column>
                <Column field="name" header="Name" class="text-sm">
                    <template #body="slotProps">
                        <div class="text-center">{{ slotProps.data.name }}</div>
                    </template>
                    <template #filter="{ filterModel, filterCallback }">
                        <InputText
                            v-model="filterModel.value"
                            class="w-full"
                            placeholder="Search by name"
                            @input="filterCallback()"
                        />
                    </template>
                </Column>
                <Column field="roles_array" header="Roles" class="text-sm">
                    <template #body="slotProps">
                        <div class="text-center">
                            {{ slotProps.data.roles_array.join(', ') }}
                        </div>
                    </template>
                    <template #filter="{ filterModel, filterCallback }">
                        <InputText
                            v-model="filterModel.value"
                            class="w-full"
                            placeholder="Search by role"
                            @input="filterCallback()"
                        />
                    </template>
                </Column>
                <Column field="permissions_array" header="Permission" class="text-sm">
                    <template #body="slotProps">
                        <div class="text-center">
                            {{ slotProps.data.permissions_array.join(', ') }}
                        </div>
                    </template>
                    <template #filter="{ filterModel, filterCallback }">
                        <InputText
                            v-model="filterModel.value"
                            class="w-full"
                            placeholder="Search by permission"
                            @input="filterCallback()"
                        />
                    </template>
                </Column>
            </DataTable>
        </div>
    </CmpLayout>
</template>
