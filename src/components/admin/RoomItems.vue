<template>
    <div class="container-fluid">
        <h3 class="mt-3">
            <template v-if="activeView === 'list'">
                <h3 class="mt-3">Room items</h3>
            </template>
            <template v-else>
                <template v-if="issetRoomItem">
                    Edit room item
                </template>
                <template v-else>
                    Create room item
                </template>
            </template>
        </h3>
        <template v-if="activeView === 'list'">
            <div class="btn btn-primary mb-3 mr-3" v-on:click="create()">Create new</div>
        </template>
        <template v-if="activeView === 'list'">
            <div>
                <table class="table table-bordered table-striped">
                    <thead>
                    <tr>
                        <th scope="col">#</th>
                        <th scope="col">Active</th>
                        <th scope="col">Name</th>
                        <th scope="col">type</th>
                        <th scope="col">Project</th>
                        <th scope="col">Actions</th>
                    </tr>
                    </thead>
                    <tbody>
                    <template v-for="(roomItem,index) in roomItems" :key="roomItem">
                        <tr>
                            <th scope="row">{{ index + 1 }}</th>
                            <td>
                                <template v-if="roomItem.active">
                                    <i class="fa-xl bi bi-check-circle-fill text-success"></i>
                                </template>
                                <template v-else>
                                    <i class="fa-xl bi bi-x-circle-fill text-danger"></i>
                                </template>
                            </td>
                            <td>{{ roomItem.name }}</td>
                            <td>{{ roomItem.type }}</td>
                            <td>
                                <router-link v-if="roomItem.project?.id" :to="{ name: 'edit-project', params: { propProjectId: roomItem.project.id }}">{{ roomItem.project.name }}</router-link>
                            </td>
                            <td>
                                <button type="button" class="btn btn-primary mr-3" v-on:click="edit(roomItem.id)">Edit</button>
                                <button type="button" class="btn btn-warning mr-3" v-on:click="getTemplates(roomItem.id)">Templates</button>
                            </td>
                        </tr>
                    </template>
                    <pagination
                            v-if="pagination.total"
                            :total-items="pagination.total"
                            :items-per-page="pagination.perPage"
                            :current-page="pagination.currentPage"
                            @page-changed="changePage"
                    />
                    </tbody>
                </table>
            </div>
        </template>
        <template v-if="activeView === 'edit' || activeView === 'create'">
            <div>
                <div class="row">
                    <div class="form-group col-12">
                        <div class="form-check">
                            <input class="form-check-input" type="checkbox" v-model="currentRoomItem.active">
                            <label class="form-check-label" for="activeCheckBox">
                                Active
                            </label>
                        </div>
                    </div>
                    <div class="form-group col-12">
                        <label>Name</label>
                        <input type="text" class="form-control" placeholder="Item name" v-model="currentRoomItem.name">
                    </div>
                    <div class="form-group col-12">
                        <label>Type</label>
                        <input type="text" class="form-control" placeholder="Item type" v-model="currentRoomItem.type">
                    </div>
                    <template v-if="activeView === 'create'">
                        <div class="form-group col-12">
                            <label>Project</label>
                            <select class="form-control" v-model="currentRoomItem.project_id">
                                <option v-for="(project, index) in projects" :key="project" :value="index">{{ project }}</option>
                            </select>
                        </div>
                    </template>
                </div>
                <div>
                    <div class="btn btn-success mr-3" v-on:click="save()">
                        <template v-if="issetRoomItem">
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
    </div>
</template>

<script>
import ProjectService from '@/services/admin/project.service'
import RoomItemService from '../../services/admin/roomItem.service'
import Pagination from '@/components/Pagination.vue'

export default {
    components: {
        Pagination,
    },
    name: 'admin-room-items',
    data() {
        return {
            roomItems: Array,
            pagination: Array,
            activeView: this.propActiveView,
            roomItemId: this.propRoomItemId,
            currentRoomItem: {},
            projects: [],
        }
    },
    props: {
        propActiveView: {
            type: String,
            default: 'list',
            required: false,
        },
        propRoomItemId: {
            type: String,
            default: '',
            required: false,
        },
    },
    computed: {
        issetRoomItem() {
            return typeof this.currentRoomItem.id !== 'undefined' && this.currentRoomItem.id > 0
        }
    },
    methods: {
        changePage(page) {
            if (this.pagination.currentPage === page) {
                return
            }
            this.$router.push({ name: 'room-items', query: { page: page } })
        },
        goToList() {
            this.$router.push({ path: this.$router.options.history.state.back })
        },
        edit(id) {
            this.$router.push({ name: 'edit-room-item', params: { propRoomItemId: id } })
        },
        getTemplates(id) {
            this.$router.push('/admin/room-item-templates/template/' + id)
        },
        create() {
            this.$router.push({ name: 'create-room-item' })
        },
        getRoomItem(id) {
            let self = this

            RoomItemService.get(id).then(
                (response) => {
                    self.currentRoomItem = response.data
                    self.activeView = 'edit'
                },
                (error) => {
                    console.log(error)
                },
            )
        },
        getList() {
            let self = this

            RoomItemService.all(this.$route.query?.page).then(
                (response) => {
                    self.roomItems = response.data.items
                    self.pagination = response.data.pagination
                    self.activeView = 'list'
                },
                (error) => {
                    console.log(error)
                },
            )
        },
        save() {
            let self = this

            RoomItemService.save(this.currentRoomItem).then(
                () => {
                    self.goToList()
                },
                (error) => {
                    console.log(error)
                },
            )
        },
        async updatePage() {
            if (this.activeView === 'list') {
                this.getList()
            }
            if (this.activeView === 'edit') {
                this.getRoomItem(this.roomItemId)
            }
            if (this.activeView === 'create') {
                await this.getAllowProjects()
                this.currentRoomItem = {
                    id: 0,
                    active: true,
                    name: '',
                    type: '',
                    project_id: 0,
                }
            }
        },
        async getAllowProjects() {
            let self = this
            await ProjectService.allowProjectList().then(
                (response) => {
                    self.projects = response.data
                },
                (error) => {
                    console.log(error)
                },
            )
        },
    },
    mounted() {
        this.updatePage()
    },
}
</script>
