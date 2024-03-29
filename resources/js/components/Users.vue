<template>
    <div class="container">
        <div class="row justify-content-center" v-if="!$gate.isAdminORAuthor()">
            <div class="col-md-8 mt-3">
                <not-found></not-found>
            </div>
        </div>

        <div class="row mt-4" v-if="$gate.isAdminORAuthor()">
          <div class="col-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User Profiles</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal()">Add New <i class="fas fa-user-plus fa-fw"></i></button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover text-nowrap">
                  <thead>
                    <tr>
                      <th>ID</th>
                      <th>Name</th>
                      <th>Email</th>
                      <th>Type</th>
                      <th>Registered At</th>
                      <th>Modify</th>
                    </tr>
                  </thead>
                  <tbody>
                    <tr v-for="user in users" :key="user.id">
                      <td>{{user.id}}</td>
                      <td>{{user.name}}</td>
                      <td>{{user.email}}</td>
                      <td>{{user.type | upText}}</td>
                      <td>{{user.created_at | trimDateTime}}</td>
                      <td>
                            <a href="#" @click="editModal(user)">
                                <i class="fa fa-edit blue"></i>
                            </a>
                          /
                            <a href="#" @click="deleteUser(user.id)">
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
        <div class="modal fade" id="addNewModal" tabindex="-1" role="dialog" aria-labelledby="exampleModalCenterTitle" aria-hidden="true">
          <div class="modal-dialog modal-dialog-centered" role="document">
            <div class="modal-content">
              <div class="modal-header">
                <h5 v-show="!editMode" class="modal-title" id="addNewModalTitle">Add New</h5>
                <h5 v-show="editMode" class="modal-title" id="updateModalTitle">Update Users's Info</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                  <span aria-hidden="true">&times;</span>
                </button>
              </div>

              <form @submit.prevent=" editMode ? updateUser() : createUser()">
              <div class="modal-body">
                    <div class="form-group">
                        <input v-model="form.name" type="text" name="name"
                            placeholder="Name"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                        <has-error :form="form" field="name"></has-error>
                    </div>

                    <div class="form-group">
                        <input v-model="form.email" type="email" name="email"
                            placeholder="Email"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                        <has-error :form="form" field="email"></has-error>
                    </div>

                    <div class="form-group">
                        <textarea v-model="form.bio" name="bio" id="bio"
                            placeholder="short bio for the user(optional)"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                        <has-error :form="form" field="bio"></has-error>
                    </div>

                    <div class="form-group">
                        <select v-model="form.type" type="text" id="type"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                                <option value="">Select User Role</option>
                                <option value="admin">Admin</option>
                                <option value="user">User</option>
                                <option value="author">Author</option>
                        </select>
                        <has-error :form="form" field="type"></has-error>
                    </div>

                    <div class="form-group">
                        <input v-model="form.password" type="password" name="password" id="password"
                            placeholder="Password"
                            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
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
        data(){
            return{
                editMode: false,
                users: {},
                form: new Form({
                    id: '',
                    name: '',
                    email: '',
                    password: '',
                    type: '',
                    bio: '',
                    photo: ''
                })
            }
        },

        methods:{
            updateUser(){
                this.$Progress.start();
                this.form.put('api/user/'+this.form.id)
                .then(()=>{
                    Toast.fire({
                        icon: 'success',
                        title: 'User Updated successfully'
                    });
                    $('#addNewModal').modal('hide');
                    this.$Progress.finish();
                    Fire.$emit('AfterCreate');
                })
                .catch((e)=>{
                    this.$Progress.fail();
                })

            },

            editModal(user){
                this.editMode = true;
                this.form.reset();
                $('#addNewModal').modal('show');
                this.form.fill(user);
            },

            newModal(){
                this.editMode = false;
                this.form.reset();
                $('#addNewModal').modal('show');
            },

            loadUsers(){
                if(this.$gate.isAdminORAuthor()){
                    axios.get("api/user").then(({data}) => (this.users = data.data));
                }
            },

            createUser(id){
                this.$Progress.start();
                this.form.post('api/user')
                .then( () =>{
                    Fire.$emit('AfterCreate');
                    $('#addNewModal').modal('hide')

                    Toast.fire({
                        icon: 'success',
                        title: 'User Created successfully'
                    });

                    this.$Progress.finish();
                })
                .catch((e)=>{
                    this.$Progress.fail();
                })
            },

            deleteUser(id){
                Swal.fire({
                title: 'Are you sure?',
                text: "You won't be able to revert this!",
                icon: 'warning',
                showCancelButton: true,
                confirmButtonColor: '#3085d6',
                cancelButtonColor: '#d33',
                confirmButtonText: 'Yes, delete it!'
                }).
                then((result) => {
                    if (result.isConfirmed){
                        //send resuls to the server
                        this.form.delete('api/user/'+id)
                        .then(()=>{
                                Swal.fire(
                                  'Deleted!',
                                  'Your file has been deleted.',
                                  'success'
                                );
                                Fire.$emit('AfterCreate');
                        })
                        .catch((e)=>{
                            Swal("Failed!","There was something wrong.","warning")
                        })
                    }
            })
            },

            //New methods go here
        },

        mounted() {
            this.loadUsers();
            Fire.$on('AfterCreate',()=>{
                this.loadUsers();
            });
        }
    }
</script>
