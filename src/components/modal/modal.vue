<template>
  <div class="modal-mask" v-show="show" transition="modal" @click="show = false">

      <div class="modal-container" @click="preventClick($event)" :style="containerStyle">
        <article>

          <!--<div class="user_info_ctn">-->

          <div id="user_info" v-if="image.sourceOwner">
            <div id="wrapper">
            <div id="user_avatar">
              <img :src='image.sourceOwner.profile_picture'>
            </div>
            <div id="user_info_right">
              <div id="user_name">
                <a :href="determineOriginal(image)" target="_blank">{{image.sourceOwner.username}}</a>
              </div>

              <div class="time_location">
                <div class="location" v-if="image.location"><i class="fa fa-map-marker" aria-hidden="true" style="margin-right:0.5em"></i>{{image.location.address}}</div>
                <div class="image_time"><i class="fa fa-clock-o" aria-hidden="true" style="margin-right:0.2em"></i>{{getShotTime(image.shotTime)}}</div>
              </div>

            </div>
              </div>
          </div>

          <div id="user_info" v-else>
            <div id="wrapper">
              <div id="user_avatar">
                <img :src='image.owner.avatar'>
              </div>
              <div id="user_info_right">
                <div id="user_name">
                  <a :href="determineOriginal(image)" target="_blank">{{image.owner.name}}</a>
                </div>
                <div class="time_location">
                  <div class="location" v-if="image.location"><i class="fa fa-map-marker" aria-hidden="true" style="margin-right:0.5em"></i>{{image.location.address}}</div>
                  <div class="image_time"><i class="fa fa-clock-o" aria-hidden="true" style="margin-right:0.2em"></i>{{getShotTime(image.shotTime)}}</div>
                </div>

              </div>
            </div>
          </div>



            <!--</div>-->



          <div id="image_info_left">
            <div id="image_description">
              {{image.description}}
            </div>
            <div id="image_tags">
              <ul class="tag_list">
                <li class="one_tag" v-for="tag in image.tags"><a v-link="{path: userType=='user' ? `/appcontent/search/${tag.text}`:`/search/${tag.text}`}" target="_blank">{{'#' + tag.text}}</a></li>
              </ul>
            </div>
          </div>

          <div>
            <div id="rtn-image" v-bind:style="{paddingBottom: heightPerc}">
              <img :src="image.retina.url" :style="imgStyle">
            </div>
            <div id="temp"></div>
          </div>

        </article>
  </div>
</template>

<script>
  require('./modal.scss')

    import dataService from '../../services/dataservices'
    export default{
      props: {
        show: {
          type: Boolean,
          required: true,
          twoWay: true
        },
        image:{
          type: Object
        },
        width:{
          required: true
        },
        height:{
          required: true
        }
      },

      ready(){
      },
        data(){
            return{
              heightPerc: 0,
              containerStyle:{
                maxWidth: '935px'
              },
              imgStyle:{
                width: '600px'
              },
              userType:dataService.getUserType()
            }
        },
        components:{
        },

        watch: {
          'width': function(val, oldVal){
            let width = val;
            let height = this.height;
            let perc = 600 * this.height / this.width / 600 * 100;
            var defaultImgWidth = 600;
            var defaultContWidth = 935;
            while(defaultImgWidth * perc / 100 > window.innerHeight - 50){
              defaultImgWidth = defaultImgWidth - 40;
              defaultContWidth = defaultContWidth - 40;
              perc = defaultImgWidth * this.height / this.width / defaultImgWidth * 100;
            }
            this.containerStyle.maxWidth = defaultContWidth.toString() + 'px';
            this.imgStyle.width = defaultImgWidth.toString() + 'px';
            perc = perc.toString() + "%";
            this.heightPerc = perc;
          },
          'show':function(val, oldVal){
            if (val){
              $('html').css('overflow-y','hidden');
            }else{
              $('html').css('overflow-y','auto');
            }
          }
        },
        methods:{
          preventClick: function(e){
            e.stopPropagation();
          },
          determineOriginal: function (image) {

            if (image && image.sourceOwner) {
              switch (image.sourceType) {
                case'SM':
                  return '#/user/' + image.owner.id;
                  break;
                case 'IN':
                  if (image.sourceOwner != null)
                    return 'https://instagram.com/' + image.sourceOwner.username;
                  break;
                case 'WB':
                  return 'https://weibo.com/' + image.sourceOwner.id;
                  break;
                case 'TW':
                  return 'https://twitter.com/' + image.sourceOwner.username;
                  break;
                case 'FL':
                  return 'https://www.flickr.com/photos/' + image.sourceOwner.id;
                  break;
                case 'PX':
                  return 'https://500px.com/' + image.sourceOwner.username.replace(/ /g, '');
                  break;
                case 'TU':
                  return 'https://' + image.sourceOwner.id;
                  break;
              }
            }
          },

          getShotTime: function (shotTime) {
            var second = (Date.now() - shotTime) * 0.001;
            var minutes = second / 60;
            var hour = minutes / 60;
            if (hour < 1) {
              return Math.floor(minutes) + ' mins ago';
            }
            var day = hour / 24;
            if (day < 1) {
              return Math.floor(hour) + ' hours ago';
            }
            var week = day / 7;
            if (week < 1) {
              return Math.floor(day) + ' days ago';
            }
            return Math.floor(week) + ' weeks ago';
          }
        }
    }
</script>
