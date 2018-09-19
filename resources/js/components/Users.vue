<template>
    <div class="container">
        <div class="row">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal">Add new <i class="fas fa-user-plus ml-2"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tbody><tr>
                    <th>ID</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="user in users" :key="user.id">
                    <td>{{ user.id }}</td>
                    <td>{{ user.name | upText }}</td>
                    <td>{{ user.email }}</td>
                    <td>{{ user.type | upText }}</td>
                    <td>{{ user.created_at | myDate }}</td>
                    <td>
                        <a href="#" @click="editModal(user)">
                            <i class="fa fa-edit blue"></i>
                        </a>
                        |
                        <a @click="deleteUser(user.id)" href="#">
                            <i class="fa fa-trash red"></i>
                        </a>
                    </td>
                  </tr>
                </tbody></table>
              </div>
              <!-- /.card-body -->
            </div>
            <!-- /.card -->
          </div>
        </div>
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="addNewTitle" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" v-show="!editmodal" id="addNewTitle">Add New</h5>
                    <h5 class="modal-title" v-show="editmodal" id="addNewTitle">Update User's Info</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <form @submit.prevent="editmodal ? updateUser() : createUser()">
                <div class="modal-body">

                        <div class="form-group">
                            <label>Username</label>
                            <input v-model="form.name" type="text" name="name" 
                                placeholder="Name"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                            <has-error :form="form" field="name"></has-error>
                        </div>

                        <div class="form-group">
                            <label>Email</label>
                            <input v-model="form.email" type="email" name="email" 
                                placeholder="Email address"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                            <has-error :form="form" field="email"></has-error>
                        </div>

                        <div class="form-group">
                            <label>Bio</label>
                            <textarea v-model="form.bio" name="bio" id="bio"
                                placeholder="Short bio for user (Optional)"
                                class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                            <has-error :form="form" field="bio"></has-error>
                        </div>

                        <div class="form-group">
                            <label>Type</label>
                            <select name="type" id="type" v-model="form.type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">Standard User</option>
                                <option value="author">Author</option>
                            </select>
                        </div>

                        <div v-show="!editmodal" class="form-group">
                            <label>password</label>
                            <input v-model="form.password" type="password" name="password" id="password"
                            class="form-control" :class="{'is-invalid': form.errors.has('password')}">
                            <has-error :form="form" field="password"></has-error>
                        </div>

                        <div v-show="!editmodal" class="form-group">
                            <label>Re-password</label>
                            <input v-model="form.password_confirm" type="password" name="password_confirm" id="password_confirm"
                            class="form-control" :class="{'is-invalid': form.errors.has('password_confirm')}">
                            <has-error :form="form" field="password_confirm"></has-error>
                        </div>
                    
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                    <button v-show="editmodal" type="submit" class="btn btn-success">Update</button>
                    <button v-show="!editmodal" type="submit" class="btn btn-primary">Create</button>
                </div>
                </form>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        data(){
            return {
                editmodal: false,
                users : {},
                form: new Form({
                    id:'',
                    name : '',
                    email: '',
                    password: '',
                    password_confirm: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },
        methods: {
            updateUser(){
                this.$Progress.start()
                this.form.put('api/user/'+this.form.id)
                .then(()=>{
                    $('#addNew').modal('hide')
                    swal(
                        'Updated!',
                        'Information has been updated.',
                        'success'
                    )
                    this.$Progress.finish()
                    Fire.$emit('AfterCreated')
                })
                .catch(()=>{
                    this.$Progress.fail()
                })
            },
            editModal(user){
                this.editmodal = true
                this.form.reset()
                $('#addNew').modal('show')
                this.form.fill(user)
            },
            newModal(){
                this.editmodal =false
                this.form.reset()
                $('#addNew').modal('show')
            },
            deleteUser(id){
                swal({
                    title: 'Are you sure?',
                    text: "You won't be able to revert this!",
                    type: 'warning',
                    showCancelButton: true,
                    confirmButtonColor: '#3085d6',
                    cancelButtonColor: '#d33',
                    confirmButtonText: 'Yes, delete it!'
                }).then((result) => {
                    // Send request to the server
                    if(result.value){
                        this.form.delete('api/user/'+id).then(()=>{
                            this.$Progress.start()
                            swal(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                            )
                            Fire.$emit('AfterCreated')
                            this.$Progress.finish()
                        }).catch(()=>{
                            swal("Failed!","There was something wronge.","warning")
                        })
                    }
                })
            },
            loadUsers(){
                axios.get("api/user").then(({ data }) => (this.users = data.data));
            },
            createUser(){
                
                this.form.post('api/user')
                .then(() => {
                    this.$Progress.start()
                    Fire.$emit('AfterCreated')
                    $('#addNew').modal('hide')
                    toast({
                        type: 'success',
                        title: 'User created in successfully'
                    })
                    // this.loadUsers()
                    this.$Progress.finish()
                })
                .catch(() => {
                    this.$Progress.fail()
                })
                
            }
        },
        beforeCreate() {
            this.$Progress.start()
        },
        created() {
            this.loadUsers()
            Fire.$on('AfterCreated',() => {
                this.loadUsers()
            })
            // setInterval(()=>this.loadUsers(),3000)
        },
        mounted() {
            this.$Progress.finish()
        },
    }
</script>
