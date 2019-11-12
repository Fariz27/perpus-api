<template>
    <section class="section">
          <h1 class="section-header">
            <div>Data Buku</div>
          </h1>
          <div class="row">
            <div class="col-lg-3 col-md-6 col-12">
              <div class="card card-sm-3">
                <div class="card-icon bg-primary">
                  <i class="ion ion-ios-paper-outline"></i>
                </div>
                <div class="card-wrap">
                  <div class="card-header">
                    <h4>Total Buku</h4>
                  </div>
                  <div class="card-body">
                    {{rows}}
                  </div>
                </div>
              </div>
            </div>
            <div class="col-lg-3 col-md-6 col-12">
              <div class="card card-sm-3">
                <div class="card-icon bg-danger">
                  <i class="ion ion-person"></i>
                </div>
                <div class="card-wrap">
                  <div class="card-header">
                    <h4>Buku Dipinjam</h4>
                  </div>
                  <div class="card-body">
                    {{sumdipinjam}}
                  </div>
                </div>
              </div>
            </div>
            <div class="col-lg-3 col-md-6 col-12">
              <div class="card card-sm-3">
                <div class="card-icon bg-warning">
                  <i class="ion ion-paper-airplane"></i>
                </div>
                <div class="card-wrap">
                  <div class="card-header">
                    <h4>Total Stock</h4>
                  </div>
                  <div class="card-body">
                    {{sum}}
                  </div>
                </div>
              </div>
            </div>                
          </div>
          <!-- ---------------------------------------------------------------  -->
          <h1 class="section-header">
            <div>Data Buku</div>
          </h1>
          <div class="section-body">
            <div class="row">
              <div class="col-12 col-md-12 col-lg-12">
                <div class="card">
                  <div class="card-header">
                    <h4>Tambah Buku</h4>
                    <b-button variant="primary" v-on:click="Add" v-b-modal.modal_book pill>
                        Tambah Buku
                    </b-button>
                  </div>
                  <b-card
                    class="mt-2">
                      
                      <b-form-input v-model="search" placeholder="Pencarian..." class="mb-3" v-on:keyup.enter="searching"></b-form-input>
                      <b-table striped hover :items="book" :fields="fields">
                        <template v-slot:cell(Aksi)="data">
                          <b-button variant="info" size="sm" v-on:click="Edit(data.item)" v-b-modal.modal_book pill>
                              Ubah
                          </b-button>
                          <b-button variant="danger" size="sm" v-on:click="Drop(data.item.id)" pill>
                              Hapus
                          </b-button>
                          <b-button variant="primary" size="sm" v-on:click="Pinjam(data.item)" v-b-modal.modal_pinjam pill>
                              Pinjam
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
                        id="modal_book"
                        title="Form Book"
                        header-bg-variant="light"
                        border-variant="light"
                        hide-footer>
                        
                        <b-container fluid>
                          <form v-on:submit="Save">
                            Nama Buku
                            <b-form-input v-model="title" class="mb-2" required></b-form-input>
                            Deskripsi
                            <b-form-input v-model="description" class="mb-2" required></b-form-input>
                            Stock
                            <b-form-input type="number" v-model="stock" class="mb-2" required></b-form-input>
                            <b-button variant="primary" pill class="pull-right btn-sm my-3" type="submit">
                                Simpan
                            </b-button>
                          </form>
                        </b-container>

                        <template v-slot:modal-footer>
                          
                        </template>
                      </b-modal>

                      <b-modal
                        id="modal_pinjam"
                        title="Form Pinjam"
                        header-bg-variant="light"
                        border-variant="light"
                        hide-footer>
                        
                        <b-container fluid>
                          <form v-on:submit="Save">
                            <h2>Pinjam {{title}}</h2>
                            Nama Peminjan <br>
                            <select v-model="peminjam">
                              <option v-for="users in user" :value="users['id']" v-bind:key="users">{{users.name}}</option>
                            </select>
                            <br>
                            Jumlah pinjam
                            <b-form-input type="number" v-model="jumlah_pinjam" class="mb-2" required></b-form-input>
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
      npeminjams: [],
      tanggal_kembali: "",
      peminjam: "",
      jumlah_pinjam: "",
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
      perPage: 5,
      key: "",
      book: [],
      peminj: [],
      user: [],
      sum: "",
      sumdipinjam: "",
      // total: "",
      fields: ["id", "title", "stock", "description", "dipinjam", "Aksi"],
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
          this.book = response.data.book;
          this.rows = response.data.count;
          this.sum = response.data.sum;
          this.sumdipinjam = response.data.sumdipinjam;
          // this.total = response.data.stock_sum;
        } else {
          window.location = "../login.html";
        }
        
      })
      .catch(error => {
        console.log(error);
      });
    },

    getUser : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show("loadingToast");
      axios.get(base_url + "/user/" + this.perPage + "/" + offset, conf)
      .then(response => {
        if(response.data.status == 1){
          this.$bvToast.hide("loadingToast");
          this.user = response.data.user;
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
      axios.post(base_url + "/book/" + this.perPage + "/" + offset, form, conf)
      .then(response => {
        if(response.data.status == 1){
          this.$bvToast.hide("loadingToast");
          this.book = response.data.book;
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
      this.title = "";
      this.description = "";
      this.stock = "",
      this.dipinjam = ""; 
    },

    Edit : function(item){
      this.action = "pinjam";
      this.id = item.id;
      this.title = item.title;
      this.description = item.description;
      this.stock = item.stock;
      this.dipinjam = "";    
    },

    Pinjam : function(item){
      this.getUser();
      this.action = "pinjam";
      this.id = item.id;
      this.title = item.title;
      this.description = item.description;
      this.stock = item.stock;
      this.dipinjam = "";
    },

    Save : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      this.$bvToast.show("loadingToast");
      this.$bvModal.hide("modal_book");
      let form = new FormData();
      //form.append("action", this.action);
      form.append("id", this.id);
      form.append("title", this.title);
      form.append("description", this.description);
      form.append("stock", this.stock);
      form.append("dipinjam", this.dipinjam);
      form.append("peminjam", this.peminjam);
      form.append("jumlah_pinjam", this.jumlah_pinjam);

      if(this.action === "insert"){
        axios.post(base_url + "/book/register", form, conf)
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
      } if(this.action === "pinjam"){
        axios.post(base_url + "/pinjam/" + this.id, form, conf)
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
       else {
        axios.post(base_url + "/book/ubah", form, conf)
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
        axios.delete(base_url + "/book/" + id, conf)
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