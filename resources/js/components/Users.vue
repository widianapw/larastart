<template>
    <div class="container">
        <div class="row mt-5" v-if="$gate.isAdminorAuthor()">
          <div class="col-md-12">
            <div class="card">
              <div class="card-header">
                <h3 class="card-title">User Table</h3>

                <div class="card-tools">
                  <button class="btn btn-success" data-toggle="modal" data-target="#addNew" @click="newModal()">Add New
                      <i class="fas fa-user-plus fa-fw"></i>
                  </button>
                </div>
              </div>
              <!-- /.card-header -->
              <div class="card-body table-responsive p-0">
                <table class="table table-hover">
                  <tr>
                    <th>No</th>
                    <th>Name</th>
                    <th>Email</th>
                    <th>Type</th>
                    <th>Registered At</th>
                    <th>Modify</th>
                  </tr>
                  <tr v-for="(user,index) in users.data" :key="user.id">
                    <td>{{index+1}}</td>
                    <td>{{user.name}}</td>
                    <td>{{user.email}}</td>
                    <td><span class="tag tag-success">{{user.type | upCase}}</span></td>
                    <td>{{user.created_at | myDate}}</td>
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
                </table>
              </div>
              <!-- /.card-body -->
              <div class="card-footer">
                <pagination :data="users" @pagination-change-page="getResults">

                </pagination>
              </div>
            </div>
            <!-- /.card -->
          </div>
        </div>
        <div v-if="!$gate.isAdminorAuthor()"> 
          <not-found></not-found>
        </div>
        <div class="modal fade" id="addNew" tabindex="-1" role="dialog" aria-labelledby="exampleModalLabel" aria-hidden="true">
            <div class="modal-dialog modal-dialog-centered" role="document">
                <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title" v-show="!editmode" id="exampleModalLabel">Add New User</h5>
                    <h5 class="modal-title" v-show="editmode" id="exampleModalLabel">Edit User Data</h5>
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                    </button>
                </div>
                <form @submit.prevent="editmode ? updateUser() : createUser()">
                <div class="modal-body">
                  
                  <div class="form-group">
                    <label>Name</label>
                    <input v-model="form.name" type="text" name="name"
                      class="form-control" :class="{ 'is-invalid': form.errors.has('name') }">
                    <has-error :form="form" field="name"></has-error>
                  </div>
                  
                    <div class="form-group">
                      <label>Email</label>
                      <input v-model="form.email" type="email" name="email"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('email') }">
                      <has-error :form="form" field="email"></has-error>
                    </div>
                  
                    <div class="form-group">
                      <label>Bio</label>
                      <textarea v-model="form.bio" name="bio"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('bio') }"></textarea>
                      <has-error :form="form" field="bio"></has-error>
                    </div>
                  
                    <div class="form-group">
                      <label>Type</label>
                      <select name="type" v-model="form.type" id="type" class="form-control" :class="{ 'is-invalid': form.errors.has('type') }">
                        <option value="">Select User Role</option>
                        <option value="admin">Admin</option>
                        <option value="user">User</option>
                        <option value="author">Author</option>
                      </select>
                      <has-error :form="form" field="type"></has-error>
                    </div>

                    <div class="form-group">
                      <label>Password</label>
                      <input v-model="form.password" type="password" name="password"
                        class="form-control" :class="{ 'is-invalid': form.errors.has('password') }">
                      <has-error :form="form" field="password"></has-error>
                    </div>
                  

                </div>
                
                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-dismiss="modal">Cancel</button>
                    <button type="submit" v-show="editmode" class="btn btn-success">Edit</button>
                    <button type="submit" v-show="!editmode" class="btn btn-primary">Create</button>
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
          editmode:false,
          users:{}
          ,
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
      methods:{
        
        getResults(page = 1) {
          axios.get('api/user?page=' + page)
            .then(response => {
              this.users = response.data;
            });
        }
        ,
        updateUser(){
          this.$Progress.start();
          this.form.put('api/user/'+this.form.id)
          .then(()=>{
            Fire.$emit ('AfterCreate');
            $('#addNew').modal('hide');
            swal.fire(
              'Updated!',
              'Data Has Been Updated.',
              'success'
            )
            this.$Progress.finish();
          })
          .catch(()=>{
            this.$Progress.fail();
          });
        }
        ,
        editModal(user){
          this.editmode = true,
          this.form.reset();
          $('#addNew').modal('show');
          this.form.fill(user);
        }
        ,
        newModal(){
          this.editmode = false,
          this.form.reset();
          $('#addNew').modal('show');
        }
        ,
        deleteUser(id){
          swal.fire({
            title: 'Are you sure?',
            text: "You won't be able to revert this!",
            type: 'warning',
            showCancelButton: true,
            confirmButtonColor: '#3085d6',
            cancelButtonColor: '#d33',
            confirmButtonText: 'Yes, delete it!'
          }).then((result) => {
            this.form.delete('api/user/'+id).then(()=>{
              if (result.value) {
                Fire.$emit ('AfterCreate');
                swal.fire(
                  'Deleted!',
                  'Your file has been deleted.',
                  'success'
                )
              }
            }).catch(()=>{
              swal.fire(
                'Failed!',
                'There was Something Wrong.',
                'warning'
              )
            });
            
          })
        }
        ,
        loadUsers(){
          if(this.$gate.isAdminorAuthor()){
            axios.get("api/user").then(({data}) =>(this.users = data));
          }
        }
        ,
        createUser(){
          this.$Progress.start();
          this.form.post('api/user')
          .then(() => {
            Fire.$emit ('AfterCreate');
            $('#addNew').modal('hide');
            toast.fire({
              type: 'success',
              title: 'Signed in successfully'
            })
            this.$Progress.finish();
          })
          .catch(() => {

          });
        }
      },
      created() {
        Fire.$on('searching',()=>{
          let query = this.$parent.search;
          axios.get('api/findUser?q=' + query)
          .then((data)=>{
            this.users = data.data;
          })
          .catch(()=>{

          })
        })
          this.loadUsers();
          Fire.$on('AfterCreate',() =>{
            this.loadUsers();
          });
          // setInterval(() => this.loadUsers(), 3000);
      }
    }
</script>
