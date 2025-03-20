<script setup lang="ts">
import {ref, computed} from 'vue';
import axios from "axios";
import {TailwindPagination} from "laravel-vue-pagination";

axios.get("/links");
definePageMeta({
  middleware: ["auth"],
});
const queries=ref({
  page:1,
  sort:"",
  "filter[full_link]":"",
  ...useRoute().query
})

const page = ref(useRoute().query.page || 1) //TODO: implement pagination (08.2, 08.3)
const data = ref([])
let links = computed(() => data.value?.data);

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
  getLinks()
  useRouter().push({query:queries.value})
},
{deep:true})

getLinks()
</script>
<template>
  <div>
    <nav class="flex justify-between mb-4 items-center">
      <h1 class="mb-0">My Links</h1>
      <div class="flex items-center">
        <SearchInput modelValue="" />
        <NuxtLink to="/links/create" class="ml-4">
          <IconPlusCircle class="inline" /> Create New
        </NuxtLink>
      </div>
    </nav>

    <div v-if="true">
      <table class="table-fixed w-full">
        <thead>
          <tr>
            <TableTh class="w-[35%]">Full Link</TableTh>
            <TableTh class="w-[35%]">Short Link</TableTh>
            <TableTh class="w-[10%]">Views</TableTh>
            <TableTh class="w-[10%]">Edit</TableTh>
            <TableTh class="w-[10%]">Trash</TableTh>
            <TableTh class="w-[6%] text-center">
              <button><IconRefresh /></button>
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
      <TailwindPagination :data="data" @pagination-change-page="queries.page=$event"/>
      <div class="mt-5 flex justify-center"></div>
    </div>

    <!-- No links message for when table is empty -->
    <div
      v-else
      class="border-dashed border-gray-500 p-3 border-[1px] text-center"
    >
      <div class="flex justify-center">
        <IconLink />
      </div>
      <p>
        <!-- Show this if reason for no links is none found in search -->
        <span v-if="false"> No links matching links found. </span>

        <!-- Show this if reason for no links is User has none -->
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
