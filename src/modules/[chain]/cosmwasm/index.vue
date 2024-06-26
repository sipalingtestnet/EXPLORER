<script lang="ts" setup>
import { useBlockchain, useFormatter, useTxDialog } from '@/stores';
import { useWasmStore } from './WasmStore';
import { ref } from 'vue';
import type { PaginabledCodeInfos } from './types';
import { PageRequest } from '@/types';
import PaginationBar from '@/components/PaginationBar.vue';
import router from '@/router';

const props = defineProps(['chain']);

const codes = ref({} as PaginabledCodeInfos);

const pageRequest = ref(new PageRequest())
const wasmStore = useWasmStore();
const dialog = useTxDialog()
const creator = ref("")

function pageload(pageNum: number) {
    pageRequest.value.setPage(pageNum)
    wasmStore.wasmClient.getWasmCodeList(pageRequest.value).then((x) => {
        codes.value = x;
    });
}
pageload(1)

function myContracts() {
    router.push(`/${props.chain}/cosmwasm/${creator.value}/contracts`)
}
</script>
<template>
    <div class="bg-[#ffffff] dark:bg-[#222222] px-4 pt-3 pb-4 rounded mb-4 shadow">
        <h2 class="card-title truncate w-full">{{ $t('cosmwasm.title') }}</h2>

        <div class="join border border-[#f7921e]">
            <input v-model="creator" type=text class="input input-bordered dark:bg-[#303030] w-40 join-item" placeholder="creator address" />
            <button class="join-item join-item inline-flex items-center justify-center flex-wrap cursor-pointer select-none text-center transition duration-200 bg-[#f7921e] text-black hover:bg-[#ffffff] dark:hover:bg-[#202020] dark:hover:text-white hover:shadow-[0_0_10px_3px_rgba(255,234,110,1)] uppercase font-bold px-4" @click="myContracts()">{{ $t('cosmwasm.btn_query') }}</button>
        </div>
        <div class="overflow-x-auto">
            <table class="table table-compact w-full mt-4 text-sm">
                <thead>
                    <tr>
                        <th>{{ $t('cosmwasm.code_id') }}</th>
                        <th>{{ $t('cosmwasm.code_hash') }}</th>
                        <th>{{ $t('cosmwasm.creator') }}</th>
                        <th>{{ $t('cosmwasm.permissions') }}</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-for="(v, index) in codes.code_infos" :key="index">
                        <td>{{ v.code_id }}</td>
                        <td>
                            <RouterLink :to="`/${props.chain}/cosmwasm/${v.code_id}/contracts`"
                                class="truncate max-w-[200px] block text-[#f7921e] dark:invert" :title="v.data_hash">
                                {{ v.data_hash }}
                            </RouterLink>
                        </td>
                        <td>{{ v.creator }}</td>
                        <td>
                            {{ v.instantiate_permission?.permission }}
                            <span>{{ v.instantiate_permission?.address }}
                                {{ v.instantiate_permission?.addresses.join(', ') }}</span>
                        </td>
                    </tr>
                </tbody>
            </table>
            <div class="flex justify-between">
                <PaginationBar :limit="pageRequest.limit" :total="codes.pagination?.total" :callback="pageload" />
                <label for="wasm_store_code" class="inline-flex items-center justify-center flex-wrap cursor-pointer select-none text-center transition duration-200 rounded-lg btn-sm bg-[#f7921e] text-black hover:bg-[#ffffff] dark:hover:bg-[#202020] dark:hover:text-white hover:shadow-[0_0_10px_3px_rgba(255,234,110,1)] font-bold uppercase px-4  my-6 mr-7" @click="dialog.open('wasm_store_code', {})">{{ $t('cosmwasm.btn_up_sc') }}</label>
            </div>
        </div>
    </div>
</template>

<route>
    {
      meta: {
        i18n: 'cosmwasm'
      }
    }
</route>
