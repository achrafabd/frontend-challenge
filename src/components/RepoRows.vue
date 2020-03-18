<template>
    <div>
        <v-card v-for="item in displayedObject"
                :key="item.id" class="mt-2 ml-2 mr-2" dense>
            <v-list-item v-if="displayedObject">
                <v-list-item-avatar size="120">
                    <v-img v-if="item.owner" :src="item.avatar_url"></v-img>
                </v-list-item-avatar>

                <v-list-item-content>
                    <v-list-item-title class="headline 2" v-text="capitalize(item.name)"></v-list-item-title>
                    <v-list-item-subtitle v-text="item.description"></v-list-item-subtitle>
                    <v-list-item-subtitle>
                        <v-row align="center">
                            <v-col sm="2">

                                <v-icon color="success" left>mdi-star</v-icon>
                                {{ displayNumber(item.stargazers_count) }}

                            </v-col>
                            <v-col sm="2">
                                <v-icon color="warning" left>mdi-alert-circle-outline</v-icon>
                                {{ displayNumber(item.open_issues_count) }}
                            </v-col>
                            <v-col sm="8" v-if="item.owner">
                                Submitted {{item.updated_at}} by {{item.owner}}
                            </v-col>
                        </v-row>
                    </v-list-item-subtitle>
                </v-list-item-content>
            </v-list-item>
        </v-card>
    </div>
</template>

<script>
    import axios from 'axios'
    import Vue from 'vue'
    import moment from 'moment'

    export default {
        name: 'App',
        components: {},
        data: () => ({
            displayedObject: null,
            displayedRowNb: 0,
            i: 0,
            nbRep: 10,
            data: [],
            max: 30,
            lazyData: null,
            lazyLoad: 20,
            pageId: 1,
            isLoading: true,
            error: false,
            loadingCheck: 3000,
            gitUrl: ""
        }),
        methods: {
            //Format Text
            capitalize(string) {
                return string.charAt(0).toUpperCase() + string.slice(1)
            },
            displayNumber(number) {
                if (number >= 1000) {
                    return (number / 1000).toFixed(1) + "k"
                } else if (number >= 1000000) {
                    return (number / 1000000).toFixed(1) + "M"
                }
                return number
            },

            //Load Git repo
            requestGit() {
                var vm = this
                return new Promise(function (resolve, reject) {
                    axios.get(vm.gitUrl + "&page=" + vm.pageId)
                        .then(function (response) {
                            vm.lazyData = response.data
                            resolve(1)
                        })
                        .catch(function () {
                            reject(0)
                        });
                })
            },
            initialize() {
                var vm = this
                vm.date = moment().subtract(30, "days").format("YYYY-MM-DD")
                vm.gitUrl = "https://api.github.com/search/repositories?q=created:>" +
                    vm.date + "&sort=stars&order=desc"
                vm.$emit('loadingStart')
                vm.requestGit().then(function (value) {
                    vm.isLoading = false
                    if (value) {
                        Object.assign(vm.data, vm.lazyData);
                        vm.displayedObject = []
                        vm.displayRows()
                        vm.$emit('loadingEnd', 1)
                        return
                    }
                }).catch(function (value) {
                    vm.isLoading = false
                    vm.error = true
                    vm.$emit('loadingEnd', value)
                })
            },

            //Display Git repo
            displayRows() {
                var vm = this
                for (var i = vm.i; i < vm.i + vm.nbRep; i++) {
                    var gitRepo = {}
                    gitRepo.name = vm.data.items[i].name
                    gitRepo.description = vm.data.items[i].description
                    gitRepo.owner = vm.data.items[i].owner.login
                    gitRepo.avatar_url = vm.data.items[i].owner.avatar_url
                    gitRepo.stargazers_count = vm.data.items[i].stargazers_count
                    gitRepo.open_issues_count = vm.data.items[i].open_issues_count
                    gitRepo.updated_at = moment(vm.data.items[i].created_at).fromNow()
                    Vue.set(vm.displayedObject, vm.displayedRowNb, gitRepo)
                    vm.displayedRowNb++
                }
                vm.i = i
            },
            more() {
                var vm = this
                if (vm.isLoading) {
                    setTimeout(function () {
                        vm.more()
                    }, vm.loadingCheck);
                    return
                }
                if (vm.i == vm.max) {
                    vm.i = 0
                    Object.assign(vm.data, vm.lazyData);
                }
                if (vm.error) {
                    vm.$emit('loadingEnd', vm.error)
                    return
                }
                vm.displayRows()
                if (vm.i >= vm.lazyLoad) {
                    vm.pageId++
                    vm.isLoading = true
                    vm.requestGit().then(function (value) {
                        vm.isLoading = false
                        if (value) {
                            vm.$emit('loadingEnd', value)
                            return
                        }
                    }).catch(function (value) {
                        vm.isLoading = false
                        vm.error = true
                        vm.$emit('loadingEnd', value)
                    })
                }
            },

            //Event Handler
            scroll() {
                var vm = this
                window.onscroll = () => {
                    let bottomOfWindow = Math.max(window.pageYOffset, document.documentElement.scrollTop, document.body.scrollTop) + window.innerHeight === document.documentElement.offsetHeight
                    if (bottomOfWindow) {
                        vm.$emit('loadingStart')
                        vm.more()
                    }
                }
            }
        },
        mounted() {
            this.scroll()
        },
        created() {
            this.initialize()
        },
    }
</script>
