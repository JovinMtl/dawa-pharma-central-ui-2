<template>
    <main class="pge-setup fl fl-c p-5" style="margin: 0; padding: 0;">
      <jove-loader v-if="showLoader"></jove-loader>
      <detailsPharma v-if="showDetails" 
        @close="turnOffDetails"
        :med="theMed"
        :pharma="thePharma">
      </detailsPharma>
      <div class="ta-c pge-2-set">
        <h1 class="sen">
            <div class="hd">
                Trouver le médicament disponible dans les pharmacies.
            </div>
            Baza umuti kuri
            pharmacie.
        </h1><br>
        
        <form v-on:submit.prevent>
          <input v-model="queryset.query" class="m-5 p-5 ta-c bdr-5 inp-elm da" placeholder="ex: paracetamol, ..." /> 
        <span v-show="shortQuery" for="" class="c-r sm-l">Andika indome zitatu</span>
        <br>
          <button type="submit" class="m-" @click="getFirstPage">Rondeza</button>
        </form>
        
        <div class="sen" v-for="(umuti, index) in imiti" :key="index" data-id="index">
          <div v-show="umuti.price" class="umuti-ctn" :class="index%2 ? 'bg-g1':'bg-g2'"  :data-id="index"  @click="turnOnDetails">
            <div>💊{{ String(umuti.nom_med).slice(0, 30) }} </div>
            <div class="c-w">
              <span v-if="pharmas[umuti.owner]?.name_pharma == 'PharmacieUbuzima'">
                {{ useReadableNumber(umuti.price) }} Fbu
              </span> 
              <span class="hide-price" v-else>
                ... Fbu
              </span>;
              <span>jusqu'au {{ useRemoveBracket(umuti.date_per) }}</span>
            </div>
            <div>
              {{ pharmas[umuti.owner]?.name_pharma }} 
              <!-- <span>({{ useFormatDate(pharmas[umuti.owner]?.last_connected) }}). 
              </span> -->
              <span>(il ya {{ useTellTime(pharmas[umuti.owner]?.last_connected) }}). 
              </span>
            </div>
            <div class="btn">
              Voir plus
            </div>
          </div>
        </div>
        <section>
          <div v-show="toValue(imiti)?.length > 1">
            <button @click="getPrevPage">Précédente</button>
            {{page}}/{{ maxPage }}
            <button @click="getNextPage">Suivante</button>
          </div>
        </section>
      </div>
    </main>
</template>

<script setup lang="ts">
import { useTellTime } from '~/composables/useTellTime'

const showLoader = ref(false)
const pharmas = ref({})
const queryset = reactive({
  'page': 1,
  'query': '',
})
const page = ref(1)
const maxPage = ref(1)
const shortQuery = ref(false)
const cachedResults = reactive({
  0: null,
})
const showDetails = ref(false)
const theMed = ref({})
const thePharma = ref({})

const url_get_pharmas = 'api/gOps/get_pharmas/'
const [responsePharmas, getPharmas] =useGetRequest(url_get_pharmas)

getPharmas()

const url_post_query = 'api/gOps/search_meds_public/'
const [responseQuery, sendPostRequest] = usePostRequest(url_post_query, queryset)

const imiti = ref([
  {
    'nom_med': 'nom du médicament',
    'price': 0,
    'owner': 1,
    'date_per': "2028"
  },
  
])

useHead({
  link: [
    { rel: 'preconnect', href: 'https://fonts.googleapis.com' },
    { rel: 'preconnect', href: 'https://fonts.gstatic.com', crossorigin: '' },
    {
      rel: 'stylesheet',
      href: 'https://fonts.googleapis.com/css2?family=Sen:wght@400..800&display=swap'
    },
    {
      rel: 'stylesheet',
      href: 'https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:wght@400&display=swap'
    }
  ],
  title: 'Pharma BI - Recherchez le médicament disponible dans les pharmacies - Ntahangwa, Bujumbura, Burundi',
  meta: [
    { name: 'description', content: 'Trouvez facilement les médicaments en stock à la Pharmacie près de Vous.' },
    { name: 'robots', content: 'index, follow' },
  ]
})

// Functions
const turnOffDetails = ()=>{
  showDetails.value = false;
}
const turnOnDetails = (e:Event)=>{
  const index = Number(e.target.parentNode.getAttribute('data-id'))
  console.log("The index is : " + JSON.stringify(index))
  theMed.value = imiti.value[index]
  thePharma.value = pharmas.value[theMed.value['owner']]

  showDetails.value = true;
}
// const getSpePage = ()=>{}
const getFirstPage = ()=>{
  if (String(queryset.query).length < 3){
    shortQuery.value = true;
    setTimeout(()=>{
      shortQuery.value = false;
    }, 3000)
    return
  }
  cachedResults.value = {}
  showLoader.value = true;
  queryset.page = 1;
  sendPostRequest();
}
const getPrevPage = ()=>{
  if(queryset.page >= 2){
    queryset.page -= 1;
    if (cachedResults[queryset.page]){
      imiti.value = cachedResults[queryset.page];
      page.value = queryset.page;
      return
    }
    showLoader.value = true;
    sendPostRequest();
  }
}
const getNextPage = ()=>{
  if (page.value < maxPage.value){
    queryset.page += 1;
    if (cachedResults[queryset.page]){
      imiti.value = cachedResults[queryset.page];
      page.value = queryset.page;
      return
    }
    showLoader.value = true;
    sendPostRequest()
  } else{
    return
  }
  
}

//Watchers
watch(responseQuery, (value)=>{
  if(value?.response){
    page.value = value?.page;
    showLoader.value = false;
    cachedResults[page.value] = value?.response;
    imiti.value = value?.response;
    maxPage.value = value?.max_page;
  }
})
watch(responsePharmas, (value)=>{
  pharmas.value = value?.response;
})
</script>
<style scoped>
.pge-setup{
  width: 100%;
  height: 94vh;
  margin: 0;
}
.pge-2-set{
  width: 100%;
  height: 94vh;
  padding: 5px;
  padding-top: 10px;
  overflow-y: auto;
}
.color-title{
    font-family: system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}
.c-b{
    color: blue;
}
.hd{
  color: transparent;
}
.hide-price{
  background-color: black;
  border-radius: 4px;
  padding-left: 4px;
}
.bl-2{
    filter: blur(1px);
}
.da{
  border: 1px dashed black;
}
</style>
