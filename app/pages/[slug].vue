<template>
    <div>
        The slug : {{ slug }} + {{ remoteURL }}
        <hr>
        the response: {{ data }}
    </div>
</template>
<script setup lang="ts">
    import { remoteURL } from '../composables/urlHost'

    // Route
    const route = useRoute()
    const slugRaw:string = route.params.slug as string;
    const slugArray = String(slugRaw).split("=")
    const slug = slugArray[0];
    const page = slugArray[1]


    const url_post_query = remoteURL + 'api/gOps/search_meds_public/'
    const { data, status, error, refresh, clear } = useFetch(url_post_query, {
        params:{
            'query': slug,
            'page': page
        }
    })

</script>