<!--  -->
<template>
  <div class='warpper'>
    <hello-world :img="img" @cancle="cancel" @sure="sure" :width="width" :height="heigth" />
  </div>
</template>

<script>
import HelloWorld from './HelloWorld'
export default {
  name:'test',
  data () {
    return {
       img:require("../assets/test.jpg"), 
       width:350,
       heigth:350,
    }
  },
  

  components: {
      HelloWorld,
  },
  mounted:function(){},

  methods: {
      cancel(){
          console.log("点击了取消");
      },
      sure(img){
          console.log(img);
      }
  }
}

</script>
<style lang='css' scoped>
</style>