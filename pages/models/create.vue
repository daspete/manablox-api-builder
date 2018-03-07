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
                                            <v-flex xs4>
                                                <v-text-field small
                                                    :disabled="isSubmitting"
                                                    :loading="isSubmitting"
                                                    label="Field name"
                                                    v-model="field.name"
                                                    required
                                                ></v-text-field>
                                            </v-flex>
                                            <v-flex xs3>
                                                <v-select
                                                    dark
                                                    label="Field type"
                                                    :items="fieldtypes"
                                                    v-model="field.type"
                                                    item-value="type"
                                                    item-text="label"
                                                    :value-comparator="(a,b) => { return a.type == b.type }"
                                                ></v-select>
                                                
                                                <v-select v-if="field.type == 'relation_one'"
                                                    dark
                                                    label="Relation to"
                                                    :items="datamodels"
                                                    v-model="field.relation_one"
                                                    item-text="model_name"
                                                    :value-comparator="(a,b) => { return a.id == b.id }"
                                                ></v-select>

                                                <v-select v-if="field.type == 'relation_many'"
                                                    dark
                                                    label="Relation to"
                                                    :items="datamodels"
                                                    v-model="field.relation_many"
                                                    item-text="model_name"
                                                    :value-comparator="(a,b) => { return a.id == b.id }"
                                                ></v-select>
                                                
                                            </v-flex>
                                            <v-flex xs2 pt-4 pl-3>
                                                <v-switch
                                                    v-model="field.required"
                                                    label="required"
                                                ></v-switch>
                                            </v-flex>
                                            <v-flex xs2 pt-4 pl-3>
                                                <v-switch
                                                    v-model="field.unique"
                                                    label="unique"
                                                ></v-switch>
                                            </v-flex>
                                            <v-flex xs1 pt-4 pl-3>
                                                <v-btn right outline icon small color="error" v-on:click.prevent="RemoveModelField(field)">
                                                    <v-icon small>fa-trash</v-icon>
                                                </v-btn>
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
                type: 'boolean',
                label: 'Boolean'
            },
            {
                type: 'integer',
                label: 'Number (integer)'
            },
            {
                type: 'float',
                label: 'Number (float)'
            },
            {
                type: 'string',
                label: 'String (max. 255 characters)'
            },
            {
                type: 'text',
                label: 'Text'
            },
            {
                type: 'date',
                label: 'Date'
            },
            {
                type: 'datetime',
                label: 'Date and time'
            },
            {
                type: 'time',
                label: 'Time'
            },
            {
                type: 'relation_one',
                label: 'Relation to one'
            },
            {
                type: 'relation_many',
                label: 'Relation to many'
            }
        ]

        let datamodels = null

        try {
            datamodels = await app.$axios.$get(`/api/v1/models`)
            datamodels = datamodels.models;
        }catch(e){
            console.log(e);
        }

        return {
            datamodels,
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
                type: '',
                unique: false,
                required: false
            });
        },

        RemoveModelField(field){
            this.datamodel.fields.splice(this.datamodel.fields.indexOf(field), 1)
        }
    }

}
</script>
