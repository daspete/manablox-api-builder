<template>
    <div>
        <panel :title="`Create ${ datamodel.model_name ? datamodel.model_name : 'new datamodel' }`">
            <template slot="content">
                <v-form v-on:submit.prevent="OnFormSubmit">
                    <v-tabs color="grey darken-2" slider-color="primary">
                        <v-tab ripple>
                            Basic data
                        </v-tab>
                        <v-tab ripple>
                            Model fields
                        </v-tab>

                        <v-tab-item>
                            <v-container grid-list-sm fluid>
                                <v-layout row wrap>

                                    <v-flex xs12 md6>
                                        <v-text-field box
                                            :disabled="isSubmitting"
                                            :loading="isSubmitting"
                                            label="Title"
                                            v-model="datamodel.model_name"
                                            :counter="30"
                                            required
                                        ></v-text-field>
                                    </v-flex>

                                    <v-flex xs12 md6>
                                        <v-text-field box
                                            :disabled="isSubmitting"
                                            :loading="isSubmitting"
                                            label="Slug"
                                            v-model="datamodel.model_slug"
                                            :counter="30"
                                        ></v-text-field>
                                    </v-flex>

                                    <v-flex xs12>
                                        <v-text-field box
                                            :disabled="isSubmitting"
                                            :loading="isSubmitting"
                                            label="Description"
                                            v-model="datamodel.model_description"
                                            :counter="150"
                                        ></v-text-field>
                                    </v-flex>


                                </v-layout>
                            </v-container>

                        </v-tab-item>
                        <v-tab-item>
                            <v-container grid-list-sm fluid>
                                <v-layout row wrap>
                                    <v-flex xs12>
                                        <v-btn outline small color="primary" v-on:click.prevent="AddModelField">
                                            <v-icon left small>fa-plus</v-icon> add field
                                        </v-btn>
                                    </v-flex>
                                    <v-flex xs12 v-for="(field, fieldId) in datamodel.fields" :key="`datafield-${ fieldId }`">
                                        <v-layout row wrap>
                                            <v-flex xs6>
                                                <v-text-field box small
                                                    :disabled="isSubmitting"
                                                    :loading="isSubmitting"
                                                    label="Field name"
                                                    v-model="field.name"
                                                    required
                                                ></v-text-field>
                                            </v-flex>
                                            <v-flex xs6>
                                                <v-select
                                                    dark
                                                    label="Field type"
                                                    :items="fieldtypes"
                                                    v-model="field.type"
                                                    item-text="label"
                                                    :value-comparator="(a,b) => { return a.type == b.type }"
                                                ></v-select>
                                            </v-flex>
                                        </v-layout>
                                    </v-flex>
                                    <v-flex xs12 v-if="datamodel.fields.length > 5">
                                        <v-btn outline small color="primary" v-on:click.prevent="AddModelField">
                                            <v-icon left small>fa-plus</v-icon> add field
                                        </v-btn>
                                    </v-flex>
                                </v-layout>
                            </v-container>
                        </v-tab-item>
                        
                    </v-tabs>

                    
                    <v-container grid-list-sm fluid>
                        <v-layout row wrap>
                            <v-flex xs6>
                                <v-btn color="error"
                                    nuxt to="."
                                    :disabled="isSubmitting"
                                    :loading="isSubmitting"
                                >
                                    <v-icon left dark>cancel</v-icon>
                                    Cancel
                                </v-btn>
                            </v-flex>
                            <v-flex xs6 class="text-xs-right">
                                <v-btn color="primary"
                                    :disabled="isSubmitting"
                                    :loading="isSubmitting"
                                    type="submit"
                                >
                                    <v-icon left dark>save</v-icon>
                                    Save
                                </v-btn>
                            </v-flex>
                        </v-layout>
                    </v-container>

                </v-form>
            </template>
        </panel>
    </div>
</template>


<script>
export default {

    layout: 'apibuilder',

    async asyncData({ app, params }){
        let datamodel = {
            model_name: '',
            model_slug: '',
            model_description: '',
            fields: [],
            relations: []
        }

        let fieldtypes = [
            {
                type: 'text',
                label: 'Text'
            },
            {
                type: 'number',
                label: 'Number'
            }
        ]

        let roles = null

        return {
            datamodel,
            fieldtypes,
            isSubmitting: false,
            errors: []
        }
    },

    methods: {
        async OnFormSubmit(){
            this.errors = [];

            this.isSubmitting = true;

            try{
                await this.$axios.$post(`/api/v1/models`, this.datamodel);
                this.$router.push('.');
            }catch(e){
                this.errors = [];
                this.errors.push(e.response.data.error.message);
            }

            this.isSubmitting = false;
        },

        AddModelField(){
            this.datamodel.fields.push({
                name: '',
                type: ''
            });
        }
    }

}
</script>
