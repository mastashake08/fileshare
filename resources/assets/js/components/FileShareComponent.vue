<template>
  <transition name="fade">
    <div class="container" v-if="ready">
        <div class="row">
            <div class="col-md-8 col-md-offset-2">
                <div class="panel panel-default">
                    <div class="panel-heading">Send File</div>

                    <div class="panel-body">
                        Your channel ID is {{peer.id}}
                        <input class="form-control" v-model="remote_channel_id" placeholder="Remote Channel ID">

                        <input    v-on:change="updateFile($event)" type="file">
                        <br>
                         <button class="btn btn-default" v-on:click="uploadFile($event)">Send File</button>

                    </div>
                </div>
            </div>
        </div>
    </div>
  </transition>
</template>

<script>
    export default {
        mounted() {
            console.log('Component mounted.')
        },
        data() {
          return {
            peer: {},
            conn : {},
            file: {},
            remote_channel_id: '',
            files: [],
            ready: false,
            blob: null
          }
        },
        methods: {
          updateFile: function(event){
            this.file = event.target.files[0];
            this.files = event.target.files;
          },
          uploadFile: function(event){
            var file = this.file;
            var files = this.files;

            console.log(file);
            this.conn = this.peer.connect(this.remote_channel_id);
            var conn = this.conn;
            this.conn.on('open', function(){

              var blob = new Blob(files, {type: file.type});

              conn.send({
                  file: blob,
                  filename: file.name,
                  filetype: file.type
              });
            });




          },
          downloadFile: function(data){
            var blob = new Blob([data.file], {type: data.filetype});
            this.blob = URL.createObjectURL(blob);
          }
        },
        created(){
          this.peer = new Peer({host:'fileshare.jyroneparker.com',port:9000});
          this.ready = true;
          var that = this;
          this.peer.on('connection', function(conn) {

            console.log(conn);
            that.conn = conn;
            conn.on('open', function() {
              // Receive messages
              conn.on('data', function(data){
                console.log(data);
                var blob = new Blob(data.file, {type: data.filetype});
                that.blob = URL.createObjectURL(blob);
              });
            });
          });
        }
    }
</script>
