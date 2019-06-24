<template>
  <div class="container">
    <div class="row mt-5">
      <div class="col-12">
        <div class="card">
          <div class="card-header">
            <h3 class="card-title">Users List</h3>

            <div class="card-tools">
              <div class="input-group input-group-sm" style="width: 150px;">
                <button class="btn btn-success" @click="openNewUserModal">
                  <i class="fa fa-user-plus"></i>
                </button>
              </div>
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
                  <th>Modify</th>
                </tr>
                <tr v-for="user in users" :key="user.id">
                  <td>{{user.id}}</td>
                  <td>{{user.name}}</td>
                  <td>{{user.email}}</td>
                  <td>
                    <span class="tag tag-success">{{user.type | upText}}</span>
                  </td>
                  <td>{{user.created_at | readableDate}}</td>
                  <td>
                    <button class="btn btn-success btn-small" @click="openEditUserModal(user)">
                      <i class="fa fa-pen"></i>
                    </button>
                    <button @click="deleteUser(user.id)" class="btn btn-danger btn-small">
                      <i class="fa fa-trash"></i>
                    </button>
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
    <div
      class="modal fade"
      id="userModal"
      tabindex="-1"
      role="dialog"
      aria-labelledby="userModalTitle"
      aria-hidden="true"
    >
      <div class="modal-dialog modal-dialog-centered" role="document">
        <div class="modal-content">
          <div class="modal-header">
            <h5 v-show="editMode" class="modal-title" id="userModalTitle">Update User Info</h5>
            <h5 v-show="!editMode" class="modal-title" id="userModalTitle">Add New</h5>
            <button type="button" class="close" data-dismiss="modal" aria-label="Close">
              <span aria-hidden="true">&times;</span>
            </button>
          </div>
          <form @submit.prevent="editMode? updateUser() : createUser()" id="userForm">
            <div class="modal-body">
              <div class="form-group">
                <input
                  v-model="form.name"
                  type="text"
                  name="name"
                  placeholder="Your name"
                  class="form-control"
                  :class="{ 'is-invalid': form.errors.has('name') }"
                >
                <has-error :form="form" field="name"></has-error>
              </div>

              <div class="form-group">
                <input
                  v-model="form.email"
                  type="email"
                  name="email"
                  placeholder="Email address"
                  class="form-control"
                  :class="{ 'is-invalid': form.errors.has('email') }"
                >
                <has-error :form="form" field="email"></has-error>
              </div>

              <div class="form-group">
                <input
                  v-model="form.password"
                  type="password"
                  name="password"
                  placeholder="Password"
                  class="form-control"
                  :class="{ 'is-invalid': form.errors.has('password') }"
                >
                <has-error :form="form" field="password"></has-error>
              </div>

              <div class="form-group">
                <select
                  v-model="form.type"
                  name="type"
                  class="form-control"
                  :class="{ 'is-invalid': form.errors.has('type') }"
                >
                  <option value selected>Select a user</option>
                  <option value="admin">Admin</option>
                  <option value="user">User</option>
                  <option value="author">Author</option>
                </select>
                <has-error :form="form" field="type"></has-error>
              </div>

              <div class="form-group">
                <textarea
                  v-model="form.bio"
                  name="bio"
                  placeholder="Bio"
                  class="form-control"
                  :class="{ 'is-invalid': form.errors.has('bio') }"
                ></textarea>
                <has-error :form="form" field="bio"></has-error>
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
  data() {
    return {
      users: {},
      editMode: false,
      form: new Form({
        id: "",
        name: "",
        email: "",
        password: "",
        bio: "",
        type: "",
        photo: ""
      })
    };
  },

  methods: {
    openNewUserModal() {
      this.editMode = false;
      this.form.reset();
      $("#userModal").modal("show");
    },
    openEditUserModal(user) {
      this.editMode = true;
      this.form.reset();
      $("#userModal").modal("show");
      this.form.fill(user);
    },
    createUser() {
      this.$Progress.start();
      this.form
        .post("api/user")
        .then(() => {
          Fire.$emit("loadUsers");
          $("#userModal").modal("hide");
          toast.fire({
            type: "success",
            title: "User Created successfully"
          });
          this.$Progress.finish();
        })
        .catch(() => {
          toast.fire({
            type: "error",
            title: "An error occured"
          });
          this.$Progress.fail();
        });
    },
    updateUser() {
      this.$Progress.start();
      this.form
        .put("api/user/" + this.form.id)
        .then(() => {
          Fire.$emit("loadUsers");
          $("#userModal").modal("hide");
          toast.fire({
            type: "success",
            title: "User Updated successfully"
          });
          this.$Progress.finish();
        })
        .catch(() => {
          toast.fire({
            type: "error",
            title: "An error occured"
          });
          this.$Progress.fail();
        });
    },
    loadUsers() {
      axios.get("api/user").then(({ data }) => {
        //console.log(data);
        this.users = data.data;
      });
    },

    deleteUser(id) {
      swal
        .fire({
          title: "Are you sure?",
          text: "You won't be able to revert this!",
          type: "warning",
          showCancelButton: true,
          confirmButtonColor: "#3085d6",
          cancelButtonColor: "#d33",
          confirmButtonText: "Yes, delete it!"
        })
        .then(result => {
          if (result.value) {
            //server call
            this.form
              .delete("api/user/" + id)
              .then(() => {
                swal.fire("Deleted!", "User has been deleted.", "success");
                Fire.$emit("loadUsers");
              })
              .catch(() => {
                swal.fire(
                  "Opps!",
                  "There was some error deleting the user",
                  "error"
                );
              });
          }
        });
    }
  },
  created() {
    this.loadUsers();
    Fire.$on("loadUsers", () => {
      console.log("new user created");
      this.loadUsers();
    });
  }
};
</script>
