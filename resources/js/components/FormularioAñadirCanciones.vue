<template>
  <div class="container-fluid mt-5 mb-5">
    <h3 class="mb-4">Administración de Canciones</h3>
    <div class="card card-default d-flex px-5 py-5">
          <div class="p-1">
              <div class="d-flex justify-content-between pb-2 mb-2">
                  <h5 class="card-title">Añadir Canciones </h5>
                  <div>
                      <router-link :to="{name: 'admincanciones'}" class="nav-item nav-link"><button class="fondo-color tamaño_session2">Volver</button></router-link>
                  </div>
              </div>
  
  
              <div v-if="strSuccess" class="alert alert-success alert-dismissible fade show" role="alert">
                  <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                  <strong>{{strSuccess}}</strong>
              </div>
  
  
              <div v-if="strError" class="alert alert-danger alert-dismissible fade show" role="alert">
                  <button type="button" class="btn-close" data-bs-dismiss="alert" aria-label="Close"></button>
                  <strong>{{strError}}</strong>
              </div>
  
  
  
  
              <form @submit.prevent="addCancion" enctype="multipart/form-data">
                  <div class="form-group mb-2 mt-4">
                      <label class="mb-2">Nombre</label><span class="text-danger"> *</span>
                      <input type="text" class="form-control mb-2" v-model="name" placeholder="Introduce el nombre de la canción">
                  </div>

                  <div class="form-group mb-2 mt-4">
                    <label class="mb-2" for="id_categoria_cancion" name="id_categoria_cancion">Categoria</label><span class="text-danger"> *</span>
                    <br>
                    <select class="form-control mb-2" name="id_categoria_cancion" v-model="id_categoria">
                        <option value="" selected> Seleccionar categoria</option>
                        <option></option>
                        <option value="1">Quevedo</option>
                        <option value="2">Bad Bunny</option>
                        <option value="3">Shakira</option>
                        <option value="4">Rosalía</option>
                        <option value="5">Eladio Carrión</option>
                        <option value="6">Karol G</option>
                    </select>
                  </div>
  
                  <div class="form-gorup mb-2 mt-4">
                      <label class="mb-2">Audio</label><span class="text-danger"> *</span>
                      <input type="file" class="form-control mb-2" v-on:change="onChangeAudio">
                  </div>
  
                  <div class="form-gorup mb-2 mt-4">
                      <label class="mb-2">Image</label><span class="text-danger"> *</span>
                      <input type="file" class="form-control mb-2" v-on:change="onChangeImg">
                  </div>
  
                <button type="submit" class="fondo-color tamaño_session2 mt-4">Añadir canción</button>
  
              </form>
  
          </div>
      </div>
  </div>

</template>

<script>
export default {
  data() {
      return {
          isLoggedin: false,
          user: null,
          name: '',
          audio: '',
          img: '',
          id_categoria: '',
          strSuccess: '',
          strError: '',
          imgPreview: null,
         
          audioPreview: null,
          
      }
  },
  created() {
        if(window.Laravel.isLoggedin){
            this.isLoggedin =true;
            this.user =window.Laravel.user;
        };
  },
  methods: {
      onChangeImg(e) {
          this.img = e.target.files[0];
          let reader = new FileReader();
          reader.addEventListener("load", function () {
              this.imgPreview = reader.result;
          }.bind(this), false);


          if (this.img) {
              if ( /\.(jpe?g|png|gif|webp)$/i.test( this.img.name ) ) {
                  reader.readAsDataURL( this.img );
              }
          }
      },
      onChangeAudio(e) {
          this.audio = e.target.files[0];
          let reader = new FileReader();
          reader.addEventListener("load", function () {
              this.audioPreview = reader.result;
          }.bind(this), false);


          if (this.audio) {
              if ( /\.(mp3)$/i.test( this.audio.name ) ) {
                  reader.readAsDataURL( this.audio );
              }
          }
      },
      /*Inicio*/
      addCancion(e) {
          this.$axios.get('/sanctum/csrf-cookie').then(response => {
              let existObj = this;
              const config = {
                  headers:{
                      'content-type': 'multipart/form-data'
                  }
              }

              const formData = new FormData();
              formData.append('name', this.name);
              formData.append('audio', this.audio);
              formData.append('file', this.img);
              formData.append('id_categoria_cancion', this.id_categoria);


              this.$axios.post('/api/admincanciones/add', formData, config)
                  .then(response => {
                      existObj.strError = "";
                      existObj.strSuccess = response.data.success;
                      }
                  )
                  .catch(function (error){
                      existObj.strError = error.response.data.message;
                      existObj.strSuccess = "";
                      }
                  );
          });
        }
      },
      beforeRouteEnter(to, from, next) {
  if (!window.Laravel.isLoggedin) {
    window.location.href = "/";
  } else {
    let canAdd = false;

    // Bucle para comprobar si existe el rol 'editar' 'eliminar' o 'añadir'
    for (let role of window.Laravel.user.roles) {
      if (role.rol === 'añadir') {
        canAdd = true;
      }

    }

    if (canAdd) {
      next();
    } else {
      next('/');
    }
  }
}
}


</script>