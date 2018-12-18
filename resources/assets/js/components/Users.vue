<template>
    <div class="container mt-5">
<div class="row">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">Users table</h3>

                <div class="card-tools">
                    <button class="btn btn-success" @click="newModal" >Add new</button>
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
                    <th>Regesterd at</th>
                    <th>Action</th>
                  </tr>

                  <tr v-for="user in users" :key="user.id">
                    <td>{{user.id}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td><span class="tag tag-success">{{user.type | upText}}</span></td>
                    <td>{{user.created_at | diffForHumanes}}</td>
                    <td>
                        <a href="#" @click="editModal(user)">
                            <i class="fa fa-edit"></i>
                        </a>
                        /
                         <a href="#" @click="deleteUser(user.id)">
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
        <!-- Modal -->
<div class="modal fade" id="newUser" tabindex="-1" role="dialog" aria-labelledby="newUserLabel" aria-hidden="true">
  <div class="modal-dialog modal-dialog-centered" role="document">
    <div class="modal-content">
      <div class="modal-header">
        <h5 v-show="!editMode" class="modal-title" id="newUserLabel">Add new user</h5>
        <h5 v-show="editMode" class="modal-title" id="newUserLabel">Update user info</h5>
        <button type="button" class="close" data-dismiss="modal" aria-label="Close">
          <span aria-hidden="true">&times;</span>
        </button>
      </div>

      <form @submit.prevent="editMode ? updateUser() : createUser()">
      <div class="modal-body">
        


    <div class="form-group">
        <input v-model="form.name" type="text" name="name" placeholder="Name"
            class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
        <has-error :form="form" field="name"></has-error>
    </div>

    <div class="form-group">
        <input v-model="form.email" type="email" name="email" placeholder="Email"
            class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
        <has-error :form="form" field="email"></has-error>
    </div>

    <div class="form-group">
        <select v-model="form.type" name="type"
            class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                <option value="">Select user role</option>
                <option value="admin">Admin</option>
                <option value="user">Standerd user</option>
                <option value="author">Author</option>
            
            
            </select>
        <has-error :form="form" field="type"></has-error>
    </div>

    <div class="form-group">
        <input v-model="form.password" type="password" name="password" placeholder="Password"
            class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
        <has-error :form="form" field="password"></has-error>
    </div>

        <div class="form-group">
        <textarea v-model="form.bio" name="bio" placeholder="Bio" cols="5" rows="5"
            class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
        <has-error :form="form" field="bio"></has-error>
    </div>


      </div>
      <div class="modal-footer">
        <button type="button" class="btn btn-secondary" data-dismiss="modal">Close</button>
        <button v-show="!editMode" type="submit" class="btn btn-success">Create</button>
        <button v-show="editMode" type="submit" class="btn btn-primary">Update</button>
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
                editMode:false,
                users:{},
                form: new Form({
                    id: '',
                    name : '',
                    email : '',
                    password : '',
                    type : '',
                    bio : '',
                    photo : '',
                })
            }
        },
        methods:{
            loadUsers(){
                axios.get('api/user').then(({data})=>(this.users=data.data));
            },
            newModal(){
                this.editMode=false;
                this.form.reset();
                $('#newUser').modal('show');
            },
            editModal(user){
                this.editMode=true;
                 this.form.reset();
                 this.form.fill(user);
                $('#newUser').modal('show');
            },
            createUser(){
                this.$Progress.start()
                this.form.post('api/user').then(()=>{
                     event.$emit('usersUpdated');
                     $('#newUser').modal('hide');
                        toast({
                        type: 'success',
                        title: 'User created successfully'
                        })
                this.$Progress.finish()
                }).catch(()=>{

                });

            },
            updateUser(){
                this.$Progress.start()
                this.form.put('api/user/'+this.form.id)
                    .then(()=>{
                        event.$emit('usersUpdated');
                        this.$Progress.finish();
                        $('#newUser').modal('hide');
                         toast({
                            type: 'success',
                            title: 'User updated successfully'
                              });
                        
                    })
                    .catch(()=>{
                        this.$Progress.fail();
                    })

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
                    if (result.value) {
                        this.form.delete('api/user/'+id)
                            .then(()=>{
                                swal(
                                'Deleted!',
                                'Your file has been deleted.',
                                'success'
                                )
                                event.$emit('usersUpdated');
                            })
                            .catch(()=>{
                                swal(
                                'Error!',
                                'There was something wrong.',
                                'danger'
                                )
                            })
                        swal(
                        'Deleted!',
                        'Your file has been deleted.',
                        'success'
                        )
                    }
                    })

            }
        },
        created() {
            this.loadUsers();
            event.$on('usersUpdated',()=>{
                this.loadUsers();
            })
        }
    }
</script>
