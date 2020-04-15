<template>
    <div class="container">
        <div class="row mt-5">
            <div class="col-md-12">
                <div class="card">
                    <div class="card-header">
                        <h3 class="card-title">Management User</h3>
                        <div class="card-tools">
                            <button class="btn btn-success" @click="newModal">Add New <i
                                class="fas fa-user-plus fa-fw"></i></button>
                        </div>
                    </div>
                    <!-- /.card-header -->
                    <div class="card-body table-responsive p-0">
                        <table class="table table-hover">
                            <tbody>
                            <tr>
                                <th>ID</th>
                                <th>Name</th>
                                <th>Email</th>
                                <th>Type</th>
                                <th>Registered At</th>
                                <th>Action</th>
                            </tr>
                            <tr v-for="(user, index) in users" :key="user.id">
                                <td>{{index + 1}}</td>
                                <td>{{user.name}}</td>
                                <td>{{user.email}}</td>
                                <td>{{user.type | upText}}</td>
                                <td>{{user.created_at | formatDate}}</td>
                                <td>
                                    <a href="#" @click="formEdit(user)"><i class="fa fa-edit blue"></i></a>
                                    /
                                    <a href="#" @click="deleteUser(user.id)"><i class="fa fa-trash red"></i></a>
                                </td>
                            </tr>
                            </tbody>
                        </table>
                    </div>
                    <!-- /.card-body -->
                </div>
                <!-- /.card -->
            </div>
        </div>
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewLabel"
             aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title" v-show="!editMode">Add New User</h5>
                        <h5 class="modal-title" v-show="editMode">Update User's Info</h5>
                        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                            <span aria-hidden="true">&times;</span>
                        </button>
                    </div>
                    <form @submit.prevent="editMode ? updateUser() : createUser()">
                        <div class="modal-body">
                            <div class="form-group">
                                <input v-model="form.name" type="text" name="name"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('name') }"
                                       placeholder="Input Name ...">
                                <has-error :form="form" field="name"></has-error>
                            </div>
                            <div class="form-group">
                                <input v-model="form.email" type="email" name="email"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('email') }"
                                       placeholder="Input Email Address ...">
                                <has-error :form="form" field="email"></has-error>
                            </div>
                            <div class="form-group">
                            <textarea v-model="form.bio" id="bio" name="bio"
                                      class="form-control" :class="{ 'is-invalid': form.errors.has('name') }"
                                      placeholder="Short bio for user (Optional)">
                            </textarea>
                                <has-error :form="form" field="bio"></has-error>
                            </div>
                            <div class="form-group">
                                <select name="type" v-model="form.type" id="type" class="form-control"
                                        :class="{ 'is-invalid': form.errors.has('type') }">
                                    <option value="" selected disabled>Select User Role</option>
                                    <option value="admin">Admin</option>
                                    <option value="user">Standard User</option>
                                    <option value="author">Author</option>
                                </select>
                                <has-error :form="form" field="type"></has-error>
                            </div>
                            <div class="form-group">
                                <input v-model="form.password" type="password" name="password" id="password"
                                       class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                                <has-error :form="form" field="password"></has-error>
                            </div>
                        </div>
                        <div class="modal-footer">
                            <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                            <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
                            <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Users",
        data() {
            return {
                users: [],
                editMode: false,
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: '',
                })
            }
        },
        computed: {},
        methods: {
            newModal() {
                this.editMode = false;
                this.form.reset();
                $('#addNew').modal('show')
            },
            loadUsers() {
                axios.get('api/user')
                    .then(({data}) => (this.users = data.data))
            },
            formEdit(user) {
                this.editMode = true;
                this.form.reset();
                $('#addNew').modal('show');
                this.form.fill(user)
            },
            createUser() {
                this.$Progress.start();
                this.form.post('/api/user')
                    .then(() => {
                        Fire.$emit('AfterCreate');
                        $('#addNew').modal('hide');
                        toast({
                            type: 'success',
                            title: 'User Created in successfully'
                        });
                        this.$Progress.finish();
                    }).catch(() => {

                });
            },
            updateUser() {
                this.$Progress.start();
                this.form.put('/api/user/' + this.form.id)
                    .then(() => {
                        $('#addNew').modal('hide');
                        swal(
                            'Updated!',
                            'Information has been updated.',
                            'success'
                        );
                        this.$Progress.finish();
                        Fire.$emit('AfterCreate');
                    }).catch(() => {
                    this.$Progress.fail();
                });
            },
            deleteUser(id) {
                swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    if (result.value) {
                        this.form.delete('api/user/' + id)
                            .then(() => {
                                swal(
                                    'Deleted!',
                                    'Your file has been deleted.',
                                    'success'
                                );
                                Fire.$emit('AfterCreate');
                            }).catch(() => {
                            swal("Failed!", "There was something wrong.", "warning");
                        })
                    }
                })
            }
        },
        created() {
            this.loadUsers();
            Fire.$on('AfterCreate', () => {
                this.loadUsers();
            });
        }
    }
</script>

<style scoped>

</style>
