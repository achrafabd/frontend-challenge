<template>
    <v-container>
        <v-row>
            <v-col
                    class="subtitle-1 text-center"
                    cols="12"
                    max-width="1200"
                    v-if="isLoading"
            >
                Loading more Github repos
            </v-col>
            <v-col class="subtitle-1 text-center"
                   cols="12" v-if="isLoading">
                <v-progress-linear
                        color="deep-purple accent-4"
                        indeterminate
                        rounded
                        height="6"
                ></v-progress-linear>
            </v-col>
            <v-col class="subtitle-1 text-center"
                   cols="12" v-if="error">
                <v-alert
                        outlined
                        type="warning"
                        prominent
                        border="left"
                >
                    Network error unable to connect to server please contact the service center or refresh
                </v-alert>
            </v-col>
        </v-row>
    </v-container>
</template>

<script>
    export default {
        name: 'LoadingRepo',
        props: {
            isLoading: {
                type: Boolean,
                default: true
            },
            error: {
                type: Boolean,
                default: false
            },
        },
    }
</script>
