<template>
    <section class="section">
          <h1 class="section-header">
            <div>Data user</div>
          </h1>
          <div class="section-body">
            <div class="row">
              <div class="col-12 col-md-12 col-lg-12">
                <div class="card">
                  <div class="card-header">
                    <h4>Tambah user</h4>
                    <b-button variant="primary" v-on:click="Add" v-b-modal.modal_user pill>
                        Tambah user
                    </b-button>
                  </div>
                  <b-card
                    class="mt-2">
                      
                      <b-form-input v-model="search" placeholder="Pencarian..." class="mb-3" v-on:keyup.enter="searching"></b-form-input>
                      <b-table striped hover :items="user" :fields="fields">
                        <template v-slot:cell(Aksi)="data">
                          <b-button variant="info" size="sm" v-on:click="Edit(data.item)" v-b-modal.modal_user pill>
                              Ubah
                          </b-button>
                          <b-button variant="danger" size="sm" v-on:click="Drop(data.item.id)" pill>
                              Hapus
                          </b-button>
                        </template>
                      </b-table>
                      <b-pagination
                        v-model="currentPage"
                        :per-page="perPage"
                        :total-rows="rows"
                        align="center"
                        v-on:input="pagination">
                      </b-pagination>

                      <b-toast id="loadingToast" title="Processing Data" no-auto-hide>
                        <b-spinner label="Spinning" variant="success"></b-spinner>
                        <strong class="text-success">Loading...</strong>
                      </b-toast>

                      <!-- toast untuk tampilan message box -->
                      <b-toast id="message" title="Message">
                        <strong class="text-success">{{ message }}</strong>
                      </b-toast>

                      <b-modal
                        id="modal_user"
                        title="Form User"
                        header-bg-variant="light"
                        border-variant="light"
                        hide-footer>
                        
                        <b-container fluid>
                          <form v-on:submit="Save">
                            Nama User
                            <b-form-input v-model="name" class="mb-2" required></b-form-input>
                            Email
                            <b-form-input v-model="email" class="mb-2" required></b-form-input>
                            Password
                            <b-form-input type="password" v-model="password" class="mb-2" required></b-form-input>
                            <b-button variant="primary" pill class="pull-right btn-sm my-3" type="submit">
                                Simpan
                            </b-button>
                          </form>
                        </b-container>

                        <template v-slot:modal-footer>
                          
                        </template>
                      </b-modal>

                  </b-card>
                </div>
              </div>
            </div>
          </div>
        </section>
</template>
<script>
module.exports = {
  data : function(){
    return {
      search: "",
      id: "",
      title: "",
      description: "",
      stock: "",
      dipinjam: "",
      action: "",
      message: "",
      currentPage: 1,
      rows: 0,
      perPage: 2,
      key: "",
      user: [],
      buku: [],
      fields: ["id", "title", "description", "stock", "dipinjam", "Aksi"],
    }
  },

  methods: {
    getData : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show("loadingToast");
      axios.get(base_url + "/book/" + this.perPage + "/" + offset, conf)
      .then(response => {
        if(response.data.status == 1){
          this.$bvToast.hide("loadingToast");
          this.user = response.data.user;
          this.rows = response.data.count;
        } else {
          window.location = "../login.html";
        }
        
      })
      .catch(error => {
        console.log(error);
      });
    },

    searching : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show("loadingToast");
      let form = new FormData();
      form.append("find", this.search);
      axios.post(base_url + "/user/" + this.perPage + "/" + offset, form, conf)
      .then(response => {
        if(response.data.status == 1){
          this.$bvToast.hide("loadingToast");
          this.user = response.data.user;
          this.rows = response.data.count;
        } else {
          window.location = "../login.html";
        }
      })
      .catch(error => {
          console.log(error);
      });
    },

    pagination : function(){
      if(this.search == ""){
        this.getData();
      } else {
        this.searching();
      }
    },

    Add : function(){
      this.action = "insert";
      this.name = "";
      this.email = "";
      this.password = ""; 
    },

    Edit : function(item){
      this.action = "update";
      this.id = item.id;
      this.name = item.name;
      this.email = item.email;
    },

    Save : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      this.$bvToast.show("loadingToast");
      this.$bvModal.hide("modal_user");
      let form = new FormData();
      //form.append("action", this.action);
      form.append("id", this.id);
      form.append("name", this.name);
      form.append("email", this.email);
      form.append("password", this.password);

      if(this.action === "insert"){
        axios.post(base_url + "/register", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          if(this.search == ""){
            this.getData();
          } else {
            this.searching();
          }
          this.message = response.data.message;
          this.$bvToast.show("message");
        })
        .catch(error => {
          console.log(error);
        });
      } else {
        axios.post(base_url + "/user/ubah", form, conf)
        .then(response => {
          this.$bvToast.hide("loadingToast");
          if(this.search == ""){
            this.getData();
          } else {
            this.searching();
          }
          this.message = response.data.message;
          this.$bvToast.show("message");
        })
        .catch(error => {
          console.log(error);
        });
      }
    },

    Drop : function(id){
      let conf = { headers: { "Authorization" : "Bearer " + this.key} };
      if(confirm('Apakah anda yakin ingin menghapus data ini?')){
        this.$bvToast.show("loadingToast");
        axios.delete(base_url + "/user/delete/" + id, conf)
        .then(response => {
          if(response.data.status == 1){
            this.getData();
            this.$bvToast.hide("loadingToast");
            this.message = response.data.message;
            this.$bvToast.show("message");
          } else {
            window.location = "../login.html";
          }
        })
        .catch(error => {
          console.log(error);
        });
      }
    },
  },
  mounted(){
    this.key = this.$cookies.get("Authorization");
    this.getData();
  }
}
</script>