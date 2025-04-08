<script setup lang="ts">
import {ref, computed} from 'vue';
import {TailwindPagination} from "laravel-vue-pagination";
const route = useRoute();
const router = useRouter();

definePageMeta({
  middleware: ["auth"],
});
const {data, index:getLinks, destroy} = useLinks({ })
const page = ref(Number(route.query.page) || 1) //TODO: implement pagination (08.2, 08.3)

const queries = ref({
  page: Number(route.query.page) || 1,
  sort: route.query.sort?.toString() || "",
  "filter[full_link]": route.query["filter[full_link]"]?.toString() || "",
  ...route.query
});

async function handleDelete(id: number) {
    await destroy(id);
    if(data.value){
      data.value.data = data.value.data.filter((link) => link.id !== id);
    }
    getLinks();
  
}

let links = computed(() => data.value?.data || []);

watch (queries, () => useRouter().push({query: queries.value}),
{deep:true})

await getLinks()
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
              <button @click="getLinks()"><IconRefresh class="w-[15px] relative top-[2px]"/></button>
            </TableTh>
          </tr>
        </thead>
        <tbody>
          <tr v-for="link in links">
            <td :title="`created ${useTimeAgo(link.created_at).value}`">
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
              <button @click="handleDelete(link.id)"><IconTrash /></button>
            </td>
            <td></td>
          </tr>
        </tbody>
      </table>
      <TailwindPagination :data="data" @pagination-change-page="queries.page=$event"/>
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
