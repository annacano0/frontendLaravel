<script setup lang="ts">
import {ref, computed} from 'vue';
import axios from "axios";
import {TailwindPagination} from "laravel-vue-pagination";
const route = useRoute();
const router = useRouter();

definePageMeta({
  middleware: ["auth"],
});

const page = ref(Number(route.query.page) || 1) //TODO: implement pagination (08.2, 08.3)
const data = ref([])

const queries=ref({
  page:page.value,
  sort: "",
  "filter[full_link]": "",
  ...route.query
})

let links = computed(() => data.value?.data || []);


async function getLinks() {
  try {
    // @ts-expect-error page is a number so this is ok
    const qs = new URLSearchParams(queries.value).toString(); 
    const {data:res}=await axios.get(`/links?${qs}`)
    data.value=res;
  } catch (err) {
    console.log(err);
  }
}


watch (page, () => {
  router.push({query:queries.value})
  getLinks()
},
{deep:true})

getLinks()
</script>
<template>
  <div>
    <nav class="flex justify-between mb-4 items-center">
      <h1 class="mb-0">My Links</h1>
      <div class="flex items-center">
        <SearchInput v-model="queries['filter[full_link]']" />
        <NuxtLink to="/links/create" class="ml-4">
          <IconPlusCircle class="inline" /> Create New
        </NuxtLink>
      </div>
    </nav>

    <div v-if="true">
      <table class="table-fixed w-full">
        <thead>
          <tr>
            <TableTh class="w-[29%]" name="" :modelValue="queries.sort">Full Link</TableTh>
            <TableTh class="w-[29%]" name="" :modelValue="queries.sort">Short Link</TableTh>
            <TableTh class="w-[16%]" name="" :modelValue="queries.sort">Views</TableTh>
            <TableTh class="w-[10%]" name=""  modelValue="">Edit</TableTh>
            <TableTh class="w-[10%]" name=""  modelValue="">Trash</TableTh>
            <TableTh class="w-[6%] text-center" name="" modelValue="" >
              <button @click="getLinks"><IconRefresh class="w-[15px] relative top-[2px]"/></button>
            </TableTh>
          </tr>
        </thead>
        <tbody>
          <tr v-for="link in links">
            <td>
              <a :href="link.full_link" target="_blank">
                {{ link.full_link.replace(/^http(s?):\/\//, "") }}</a
              >
            </td>
            <td>
              <a
                :href="`${useRuntimeConfig().public.appURL}/${link.short_link}`"
                target="_blank"
              >
                {{
                  useRuntimeConfig().public.appURL.replace(
                    /^http(s?):\/\//,
                    ""
                  )
                }}/{{ link.short_link }}
              </a>
            </td>
            <td>{{ link.views }}</td>
            <td>
              <NuxtLink class="no-underline" :to="`/links/${link.id}`"
                ><iconEdit
              /></NuxtLink>
            </td>
            <td>
              <button><IconTrash /></button>
            </td>
            <td></td>
          </tr>
        </tbody>
      </table>
      <TailwindPagination :data="data" @pagination-change-page="page=$event"/>
      <div class="mt-5 flex justify-center"></div>
    </div>

    <div
      v-else
      class="border-dashed border-gray-500 p-3 border-[1px] text-center"
    >
      <div class="flex justify-center">
        <IconLink />
      </div>
      <p>
        <span v-if="false"> No links matching links found. </span>

        <span v-else>
          No links created yet
          <NuxtLink to="/links/create" class="text-green-600"
            >Go create your first link!</NuxtLink
          >
        </span>
      </p>
    </div>
  </div>
</template>
