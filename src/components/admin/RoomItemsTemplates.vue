<template>
    <div class="container-fluid">
        <header>
            <template v-if="activeView==='all' || activeView === 'listForItem' || activeView === 'listForTemplate'">
                <h3 class="mt-3">Room item templates</h3>
                <div v-if="activeView !== 'all'" class="btn btn-primary mb-3 mr-3" v-on:click="goToList()">All templates</div>
                <div class="btn btn-primary mb-3 mr-3" v-on:click="createItem()">Create new</div>
                <div>
                    <table class="table table-bordered table-striped">
                        <thead>
                        <tr>
                            <th scope="col">#</th>
                            <th scope="col">Active</th>
                            <th scope="col">Name</th>
                            <th scope="col">template</th>
                            <th scope="col">Actions</th>
                        </tr>
                        </thead>
                        <tbody>
                        <template v-for="(template,index) in templates" :key="template">
                            <tr>
                                <th scope="row">{{ index + 1 }}</th>
                                <td>
                                    <template v-if="!!template.active">
                                        <i class="fa-xl bi bi-check-circle-fill text-success"></i>
                                    </template>
                                    <template v-else>
                                        <i class="fa-xl bi bi-x-circle-fill text-danger"></i>
                                    </template>
                                </td>
                                <td>{{ template.name }}</td>
                                <td><img style="max-width: 25px;" :src="template.template"></td>
                                <td>
                                    <button type="button" class="btn btn-primary mr-3" v-on:click="editItem(template.id)">Edit</button>
                                </td>
                            </tr>
                        </template>
                        </tbody>
                    </table>
                </div>
            </template>
            <template v-else-if="activeView==='update' || activeView === 'create'">
                <div>
                    <h3 class="mt-3">
                        <template v-if="activeView === 'update'">
                            Edit template
                        </template>
                        <template v-else>
                            Create template
                        </template>
                    </h3>
                    <div class="row">
                        <div class="form-group col-12">
                            <div class="form-check">
                                <input class="form-check-input" type="checkbox" v-model="template.active">
                                <label class="form-check-label" for="activeCheckBox">
                                    Active
                                </label>
                            </div>
                        </div>
                        <div class="form-group col-12">
                            <label>Name</label>
                            <input type="text" class="form-control" placeholder="Template name" v-model="template.name">
                        </div>
                        <div class="form-group col-12">
                            <label>File</label>
                            <input type="file" @change="handleFileUpload( $event )"/>
                        </div>
                        <div class="form-group col-12">
                            <img style="max-width: 25px;" :src="template.template">
                        </div>
                    </div>
                    <div>
                        <div class="btn btn-success mr-3" v-on:click="saveTemplate()">
                            <template v-if="activeView === 'update'">
                                Update
                            </template>
                            <template v-else>
                                Create
                            </template>
                        </div>
                        <div class="btn btn-primary mr-3" v-on:click="goToList()">Back</div>
                    </div>
                </div>
            </template>
        </header>
    </div>
</template>

<script>
import ItemTemplateService from '@/services/admin/item-template.service'
import router from '@/router'

export default {
    name: 'room-item-templates',
    data() {
        return {
            activeView: 'all',
            templates: Array,
            template: {},
            templateId: this.propTemplateId,
            itemId: this.propItemId,
            id: {
                type: Number,
                default: 0,
            },
            file: {},
        }
    },
    props: {
        propItemId: {
            type: Number,
            default: 0,
            required: false,
        },
        propId: {
            type: Number,
            default: 0,
            required: false,
        },
        propActiveView: {
            type: String,
            default: 'all',
            required: false,
        },
        propTemplateId: {
            type: String,
            default: 'all',
            required: false,
        },
    },
    methods: {
        goToList() {
            router.push('/admin/room-item-templates')
            this.getTemplates('all')
        },
        getTemplates(type) {
            let self = this

            if (type === 'listForTemplate') {

                ItemTemplateService.allForTemplate(self.templateId).then(
                    (response) => {
                        self.templates = response.data
                        self.activeView = type
                    },
                    (error) => {
                        console.log(error)
                    },
                )

            }

            if (type === 'listForItem') {
                ItemTemplateService.allForItem(self.itemId).then(
                    (response) => {
                        self.templates = response.data
                        self.activeView = type
                    },
                    (error) => {
                        console.log(error)
                    },
                )

            }

            if (type === 'all') {

                ItemTemplateService.all(self.itemId).then(
                    (response) => {
                        self.templates = response.data
                        self.activeView = type
                    },
                    (error) => {
                        console.log(error)
                    },
                )

            }
        },
        getTemplate(type) {
            let self = this

            if (type === 'update') {

                ItemTemplateService.get(this.templateId).then(
                    (response) => {
                        self.template = response.data
                        self.template.active = !!self.template.active
                        self.template.file = {}
                        this.activeView = type
                    },
                    (error) => {
                        console.log(error)
                    },
                )
            }

            if (type === 'create') {
                self.template = {
                    id: 0,
                    active: true,
                    name: '',
                    template: '',
                    file: {},
                }
                this.activeView = type
            }
        },
        saveTemplate() {
            ItemTemplateService.save(this.template).then(
                () => {
                    router.push('/admin/room-item-templates')
                    this.getTemplates('all')
                },
                (error) => {
                    console.log(error)
                },
            )
        },
        editItem(id) {
            this.templateId = id
            router.push('/admin/room-item-templates/' + this.templateId)
            this.getTemplate('update')
        },
        createItem() {
            router.push('/admin/room-item-templates/create')
            this.getTemplate('create')
        },
        handleFileUpload(event) {
            this.template.file = event.target.files[0]
        },
    },
    mounted() {
        if (this.activeView === 'all') {
            this.getTemplates()
        }
    },
    created() {
        this.id = this.propId
        this.itemId = this.propItemId
        this.activeView = this.propActiveView

        if (
            this.activeView === 'create' ||
            this.activeView === 'update'
        ) {
            this.getTemplate(this.activeView)
        }

        if (
            this.activeView === 'listForTemplate' ||
            this.activeView === 'listForItem' ||
            this.activeView === 'all'
        ) {
            this.getTemplates(this.activeView)
        }
    },
}
</script>
