<template>
    <section class="section">
          <h1 class="section-header">
            <div>Dashboard</div>
          </h1>
          <div class="row">
            <div class="col-lg-3 col-md-6 col-12">
              <div class="card card-sm-3">
                <div class="card-icon bg-primary">
                  <i class="ion ion-person"></i>
                </div>
                <div class="card-wrap">
                  <div class="card-header">
                    <h4>Total Member</h4>
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
                  <i class="ion ion-ios-paper-outline"></i>
                </div>
                <div class="card-wrap">
                  <div class="card-header">
                    <h4>Jumlah Buku</h4>
                  </div>
                  <div class="card-body">
                    {{brows}}
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
                    <h4>Total Buku</h4>
                  </div>
                  <div class="card-body">
                    {{sum}}
                  </div>
                </div>
              </div>
            </div>
            <div class="col-lg-3 col-md-6 col-12">
              <div class="card card-sm-3">
                <div class="card-icon bg-success">
                  <i class="ion ion-record"></i>
                </div>
                <div class="card-wrap">
                  <div class="card-header">
                    <h4>Jumlah Buku Yang Dipinjam</h4>
                  </div>
                  <div class="card-body">
                    {{sumdipinjam}}
                  </div>
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
      name: "",
      email: "",
      password: "",
      action: "",
      message: "",
      currentPage: 1,
      rows: 0,
      brows: 0,
      perPage: 2,
      key: "",
      user: [],
      sum: "",
      sumdipinjam: "",
      fields: ["id", "name", "email", "Aksi"],
    }
  },

  methods: {
    getData : function(){
      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show("loadingToast");
      axios.get(base_url + "/user/" + this.perPage + "/" + offset, conf)
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
  getBook : function(){

      let conf = { headers: { "Authorization" : 'Bearer ' + this.key } };
      let offset = (this.currentPage - 1) * this.perPage;
      this.$bvToast.show("loadingToast");
      axios.get(base_url + "/book/" + this.perPage + "/" + offset, conf)
      .then(response => {
        if(response.data.status == 1){
          this.$bvToast.hide("loadingToast");
          this.brows = response.data.count;
          this.sum = response.data.sum;
          this.sumdipinjam = response.data.sumdipinjam;
        } else {
          window.location = "../login.html";
        }
        
      })
      .catch(error => {
        console.log(error);
      });
  }

  },
  mounted(){
    this.key = this.$cookies.get("Authorization");
    this.getData();
    this.getBook();

  }
}
</script>