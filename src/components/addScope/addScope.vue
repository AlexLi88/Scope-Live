<template>
    <div id="addScope-bg" v-if="show" @click="show = false">
      <div id="addScope-container"  @click="preventClick($event)">
        <div id="create-form">
            <div id="addScope-header">Create a Scope</div>
            <div id="form-input">
              <div id="caption">
                <label>Scope Name</label>
                <input type="text" v-model="captionname" placeholder="Name your Scope" required>
              </div>


              <div id="hashtag">
                <label>Add Hashtags</label>
                <div id="add_tag_btn">
                  <button type="button" @click="addTag()"><i class="fa fa-plus" aria-hidden="true"></i></button>
                </div>


                <input type="text" v-model="newTag" placeholder="Insert A New Hashtag">

                <div id="tag_list"><div class="one_tag" v-for="tag in tagList">
                  <div class="chip">
                    {{tag}}
                    <i class="close material-icons" @click="tagList.splice(tagList.indexOf(tag),1)">close</i>
                  </div></div></div>

              </div>

              <div id="location">
                  <label>Scope Location</label>
                  <input type="text" v-model="address" placeholder="Add by city, street, etc.">
                  <div id="add_loc_btn">
                    <button type="button" @click="addLoc()"><i class="fa fa-map-pin" aria-hidden="true"></i></button>
                  </div>

                  <div id="del_loc_btn" v-show='scope.location'>
                    <button type="button" @click="delLoc()"><i class="fa fa-trash" aria-hiden="true"></i></button>
                  </div>
                <div id="map">
                  <map v-show="showMap" :scope.sync="scope"></map>
                </div>
              </div>

              <div id="description">
                  <label for="textarea1">Scope Description</label>
                  <input type="text" id="textarea1"  v-model="description" placeholder="What's your Scope about?">
              </div>

              <div class="switch" id="source">
                <label>Make a Private Scope</label><br>
                <div id="lever_container">
                <label>
                  Yes
                  <input type="checkbox" v-model="sourceType" checked="checked">
                  <span class="lever"></span>
                  No
                </label>
                  </div>
              </div>


              <div id="submit-but">
                <button class="waves-effect waves-light btn cancel" type="button" @click="show = false">Cancel</button><i class="fa fa-plus-circle" aria-hidden="true"></i><button class="waves-effect waves-light btn create" @click="submitForm()">Create Scope</button>
                </div>
            </div>
        </div>

      </div>
    </div>
</template>

<script>
    require('./addScope.scss')
    import dataService from '../../services/dataservices'
    export default{
        ready(){
          if (typeof this.dt != 'undefined'){
            var self = this;
            $.each(this.dt, function(index,value){
              self.tagList.push(value);
            });
          }

          this.scope = Object.assign({}, this.scope, { location:null})

        },
        data(){
            return{
                address:'',
                captionname: "",
                hashtag: "",
                newTag:"",
                scope:{},
                sourceType: "",
                tagList:[],
                description: "",
                scopeId: "",
                showMap: false
            }
        },
        watch:{

          'show':function(v,ov){
            $('html').css('overflow-y','hidden');
            if(v == false){
              $('html').css('overflow-y','auto');
              if(typeof this.dt == 'undefined'){
                this.tagList = [];
              }
              this.captionname = this.hashtag = this.description = this.address = '';
              this.scope = {};
              this.showMap = false;
            }
          },
          'dt': function(v,ov){
            if(this.dt){
              this.tagList = [];
              var self = this;
              $.each(this.dt, function(index,value){
                self.tagList.push(value);
              });
            }
          }
        },

        methods: {

          addTag:function(e){

            if(this.tagList.indexOf((this.newTag)) < 0){

              this.tagList.push(this.newTag);
//              console.log( this.tagList)
              this.newTag = '';
            }
          },

          addLoc:function(e){
            this.showMap = true;
            this.$broadcast('update-address', this.address);
          },
          delLoc:function(){
            this.showMap = false;
            this.scope.location = null;
            this.address = "";
          },
          preventClick: function(e){
            e.stopPropagation();
          },
          submitForm: function() {
            //console.log(this.scope)
            let sourceType = "ALL"
            if(!this.sourceType) {
              sourceType = "MEMBER";
            }
            if (!this.captionname.trim().length || !this.tagList.length) {

              !this.captionname.trim().length && toastr.error('Scope Must Have A Valid Caption');
              !this.tagList.length && toastr.error('Scope Must Have At Least One Tag');


            } else {
              var self = this;
              dataService.createScope('#' + this.tagList.join('#'), this.captionname, this.description, this.scope.location, sourceType).then((res)=> {
                if(res.status == 201)
              {
                if (res.data.result == "OK") {

                  toastr.success('Your Scope Has Been Created');
                  self.tagList = [];
                  self.$route.router.go({path: `/appcontent/s/${res.data.id}`});
                  this.show = false;

                  $('#sidemenu').trigger('::ScopeAdd',res.data.scope);

                } else if (res.data.result == "SCOPE_NAME_DUPLICATE") {
                  toastr.error('Scope Name is Duplicated');
                }
              }else
              {
                toastr.error('Unknwn Error,Please Try Later');
              }
            });
          }
         }
        },

        components:{
          map: require('../gMap/map.vue')
        },
        props: ['show', 'dt']
    }
</script>
