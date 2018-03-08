<template>
    <div>
        <panel :title="`Edit ${ datamodel.model_name ? datamodel.model_name : 'datamodel' }`">
            <template slot="content">
                <v-form v-on:submit.prevent="OnFormSubmit(false)">

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

                                    <v-flex xs12>
                                        <v-select
                                            dark
                                            multiple
                                            label="Fields for select labels"
                                            :items="datamodel.fields"
                                            v-model="datamodel.fields_for_select"
                                            item-text="name"
                                            item-value="name"
                                        ></v-select>
                                    </v-flex>

                                    <v-flex xs12>
                                        <v-select
                                            dark
                                            multiple
                                            label="Fields in lists"
                                            :items="datamodel.fields"
                                            v-model="datamodel.fields_in_lists"
                                            item-text="name"
                                            item-value="name"
                                        ></v-select>
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
                                    <v-flex class="elevation-2 my-3 px-3" xs12 v-for="(field, fieldId) in datamodel.fields" :key="`datafield-${ fieldId }`">
                                        <v-layout row wrap>
                                            <v-flex xs6 sm4 md3>
                                                <v-text-field small
                                                    :disabled="isSubmitting"
                                                    :loading="isSubmitting"
                                                    label="Field name"
                                                    v-model="field.name"
                                                    required
                                                ></v-text-field>

                                                <v-text-field small
                                                    :disabled="isSubmitting"
                                                    :loading="isSubmitting"
                                                    label="Field label"
                                                    v-model="field.label"
                                                    required
                                                ></v-text-field>
                                            </v-flex>
                                            <v-flex xs6 sm4 md3>
                                                <v-select
                                                    dark
                                                    label="Field type"
                                                    :items="fieldtypes"
                                                    v-model="field.type"
                                                    item-value="type"
                                                    item-text="label"
                                                    v-on:input="OnFieldTypeChange(field)"
                                                    :value-comparator="(a,b) => { return a.type == b.type }"
                                                ></v-select>

                                                <v-select v-if="field.type == 'relation_one'"
                                                    dark
                                                    label="Relation to"
                                                    :items="datamodels"
                                                    v-model="field.relation_one"
                                                    item-text="model_name"
                                                    item-value="id"
                                                ></v-select>

                                                <v-select v-if="field.type == 'relation_many'"
                                                    dark
                                                    label="Relation to"
                                                    :items="datamodels"
                                                    v-model="field.relation_many"
                                                    item-text="model_name"
                                                    item-value="id"
                                                ></v-select>
                                                
                                                
                                                
                                            </v-flex>
                                            <v-flex xs6 sm3 md3 pt-4 pl-3>
                                                <div v-if="GetConfig(field)">
                                                    <v-switch v-for="(config, configId) in GetConfig(field)" :key="`config-field-${ fieldId + '_' + configId }`"
                                                        v-model="field.config[config.type]"
                                                        :label="config.label"
                                                    ></v-switch>
                                                </div>

                                                <v-switch
                                                    v-model="field.required"
                                                    label="required"
                                                ></v-switch>
                                                <v-switch
                                                    v-model="field.unique"
                                                    label="unique"
                                                ></v-switch>
                                            </v-flex>
                                            <v-flex xs6 sm1 md3 pt-4 pl-3>
                                                <div class="text-xs-right">
                                                <v-btn right outline icon small color="error" v-on:click.prevent="RemoveModelField(field)">
                                                    <v-icon small>fa-trash</v-icon>
                                                </v-btn>
                                                </div>
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
                                <v-btn color="primary"
                                    :disabled="isSubmitting"
                                    :loading="isSubmitting"
                                    type="button"
                                    v-on:click.prevent="OnFormSubmit(true)"
                                >
                                    <v-icon left dark>save</v-icon>
                                    Save and close
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

import fieldtypes from '~/config/fieldtypes'

export default {

    layout: 'apibuilder',

    async asyncData({ app, params }){
        let datamodel = null
        let roles = null

        try {
            datamodel = await app.$axios.$get(`/api/v1/models/${ params.id }`)
        }catch(e){
            console.log(e);
        }

        let datamodels = null

        try {
            datamodels = await app.$axios.$get(`/api/v1/models`)
            datamodels = datamodels.models;
        }catch(e){
            console.log(e);
        }

        return {
            datamodel,
            datamodels,
            fieldtypes,
            isSubmitting: false,
            errors: []
        }
    },

    methods: {
        async OnFormSubmit(redirect = false){
            this.errors = [];

            this.isSubmitting = true;

            try{
                await this.$axios.$put(`/api/v1/models/${this.$route.params.id}`, this.datamodel);
                if(redirect) this.$router.push('.');
            }catch(e){
                this.errors = [];
                this.errors.push(e.response.data.error.message);
            }

            this.isSubmitting = false;
        },

        OnFieldTypeChange(field){
            if(!this.GetConfig(field)){
                field.config = {};
            }
            if(typeof field.relation_one !== 'undefined' && field.type != 'relation_one'){
                delete(field.relation_one)
            }
            if(typeof field.relation_many !== 'undefined' && field.type != 'relation_many'){
                delete(field.relation_many)
            }
        },

        AddModelField(){
            this.datamodel.fields.push({
                name: '',
                label: '',
                type: '',
                config: {},
                unique: false,
                required: false
            });
        },

        RemoveModelField(field){
            this.datamodel.fields.splice(this.datamodel.fields.indexOf(field), 1)
        },

        GetConfig(field){
            if(!field) return false;
            if(field == '') return false;

            let fieldType = fieldtypes.find((item) => { return item.type == field.type })
            return fieldType ? fieldType.config : false;
        }
    }

}
</script>
