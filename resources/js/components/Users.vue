<template>
    <div class="container">
        <div class="row mt-5">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users Table</h3>

                <div class="card-tools">
                  <button class="btn btn-success" @click="newModal()">Add New <i class="fas fa-user-plus faw"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                    <thead>
                        <tr>
                            <th>ID</th>
                            <th>Name</th>
                            <th>Email</th>
                            <th>Type</th>
                            <th>Registered at</th>
                            <th>Modify</th>
                        </tr>
                    </thead>

                    <tbody>
                        <tr v-for="user in users" :key="user.id">
                            <td>{{ user.id }}</td>
                            <td>{{ user.name }}</td>
                            <td>{{ user.email }}</td>
                            <td>{{ user.type | upText }}</td>
                            <td>{{ user.created_at | myDate }}</td>
                            <td>
                            <a href="" @click.prevent="editModal(user)">
                                <i class="fa fa-edit blue"></i>
                            </a>

                            <a href="" @click.prevent="deleteUser(user.id)">
                                <i class="fa fa-trash red"></i>
                            </a>
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

        <!-- Modal -->
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 class="modal-title" id="exampleModalLabel">{{ editMode ? 'Update User\'s Info' : 'Add New User' }}</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>
              <form @submit.prevent="editMode ? updateUser() : createUser()">

              <div class="modal-body">

                <div class="form-group">
                    <input v-model="form.name" type="text" name="name"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('name') }" placeholder="Name">
                    <has-error :form="form" field="name"></has-error>
                </div>
 
                <div class="form-group">
                    <input v-model="form.email" type="email" name="email"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }" 
                        placeholder="Email">
                    <has-error :form="form" field="email"></has-error>
                </div>

                <div class="form-group">
                    <textarea v-model="form.bio" type="bio" name="bio"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }" 
                        placeholder="Short bio for user (Optional)">
                    </textarea>
                    <has-error :form="form" field="bio"></has-error>
                </div>

                 <div class="form-group">
                   <select  name="type" v-model="form.type" id="type" class="form-control" :class="{'is-invalid' : form.errors.has('type')}">
                       <option value="">Select User Role</option>
                       <option value="admin">Admin</option>
                       <option value="user">Standard User</option>
                       <option value="author">Author</option>
                   </select>
                   <has-error :form="form" field="type"></has-error>
                </div>

                <div class="form-group">
                    <input v-model="form.password" type="password" name="password" id="password"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }" 
                        placeholder="Password"
                    >
                    <has-error :form="form" field="password"></has-error>
                </div>
              </div>
              <div class="modal-footer">
                <button type="button" class="btn btn-danger" data-dismiss="modal">Close</button>
                <button v-show="!editMode" type="submit" class="btn btn-primary">Create</button>
                <button v-show="editMode" type="submit" class="btn btn-success">Update</button>
              </div>

              </form>

            </div>
          </div>
        </div>
    </div>


</template>

<script>
    export default {
        data() {
            return {
                editMode : false,
                users : {},
                form: new Form({
                    id : '',
                    name : '',
                    email : '',
                    password : '',
                    type : '',
                    bio : '',
                    photo : ''
                })
            }
        },
        methods: {
            updateUser(){
               this.$Progress.start();
               this.form.put('api/user/' + this.form.id)
               .then( () => {
                    $('#addNew').modal('hide');
                    swal.fire(
                        'Updated!',
                        'User has been updated.',
                        'success'
                    );
                    Fire.$emit('AfterCreate');
                    this.$Progress.finish();
               })
               .catch( () => {
                    this.$Progress.fail();
               });
            },
            editModal(user){
                this.editMode = true;
                this.form.clear();
                this.form.fill(user);
                $('#addNew').modal('show');
            },
            newModal(){
                this.editMode = false;
                this.form.reset();
                $('#addNew').modal('show');
            },
            loadUsers(){
                axios.get("api/user").then(({data}) => (this.users = data.data));
            },
            createUser(){
                this.$Progress.start();
                this.form
                .post('api/user')
                .then( () => {
                    Fire.$emit('AfterCreate');
                    $('#addNew').modal('hide');
                    Toast.fire({
                        type: 'success',
                        title: 'User Created in successfully'
                    });
                    this.$Progress.finish(); 
                })
                .catch( () => {
                    this.Progress.fail()
                });  
            },
            deleteUser(id){
                swal.fire({
                    title : 'Are you sure?',
                    text : "You won't be able to revert this!",
                    type : "warning",
                    showCancelButton : true,
                    confirmationButtonColor : '#3085d6',
                    cancelButtonColor : '#d33',
                    confirmButtonText : "Yes, delete id!"
                }).then( (result) => {
                    if(result.value){
                        this.form.delete('api/user/' + id).then( () => {
                            Fire.$emit('AfterCreate');
                            swal.fire(
                                'Deleted!',
                                'User has been deleted.',
                                'success'
                            )
                        }).catch( () => {
                             swal.fire(
                                'Failed!',
                                'There was something wrong.',
                                'warning'
                            )
                        }); 
                    }
                });
            }
        },
        mounted() {
            console.log('Component mounted.')
        },
        created(){
            this.loadUsers();
            Fire.$on('AfterCreate', () => {
                this.loadUsers();
            });
            //setInterval( () => this.loadUsers(), 3000);
        }
    }
</script>
