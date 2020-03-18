<template>
    <v-app>
        <v-content>
            <RepoRows v-on:loadingStart="lunchLoad" v-on:loadingEnd="endLoad"></RepoRows>
            <LoadingRepo :isLoading="isLoading" :error="error"></LoadingRepo>
        </v-content>
    </v-app>
</template>

<script>
    import RepoRows from './components/RepoRows'
    import LoadingRepo from './components/LoadingRepo'

    export default {
        name: 'App',
        components: {RepoRows, LoadingRepo},
        data: () => ({
            isLoading: true,
            error: false,
        }),
        methods: {
            lunchLoad() {
                var vm = this
                vm.isLoading = true
            },

            endLoad(value) {
                var vm = this
                vm.isLoading = false
                if (!value) {
                    vm.error = true
                }
            },

        },
    }
</script>
