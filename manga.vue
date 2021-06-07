<template>
    <div>
        <ul>
           <li v-for="pic in pics" :style="{backgroundImage: 'url(' + require('../../images/'+pic)+')'}"></li>
        </ul>
    </div>
</template>

<script>
    export default {
      data() {
        return {
          message: "Hello Vue!",
          pics: [],
          aspectRatio: 0.5,
          threshold: 768
        }
      },
      methods:{
          setStyle(){
              let thisObj=this
              this.ul.style.left='0px';
              //ulの幅を画面幅によって変える
              this.setUlWidth();
              //liの高さをアスペクト比に応じて変える
              this.setLiHeight();
              
              this.setZIcon();
              
          },
          setZIcon(){
              if(this.isSmallWindow()){
                  this.div.style.backgroundImage='none';
              }else{
                  this.div.style.backgroundImage='url('+require('./z.svg')+')';
              }
          },
          pittari(){
            this.ul.style.transition = 'left 1s ease';
            let page =(Math.floor(-this.ulLeft()/this.lis[0].clientWidth))
            if(this.direction === 'left'){
                this.ul.style.left = -(page+1) * this.lis[0].clientWidth + 'px';   
            }else if(this.direction === 'right'){
                this.ul.style.left = -(page) * this.lis[0].clientWidth + 'px';
            }
            this.ulNotOver();
            let thisObj = this;
            setTimeout(function(){thisObj.ul.style.transition= 'left 0s ease';},100);
          },
          
          
          
          moveEnd(e){
              e.preventDefault();
              this.pittari();
              this.nowMove = false;
          },
          movePics(e){
              e.preventDefault();
              if(this.nowMove === true){
                 let move_length = 0;
                 if(e.type==='mousemove'){
                     move_length = e.movementX;
                     this.ul.style.left= this.ulLeft() + move_length +'px';
                 }else if(e.type==='touchmove'){
                     move_length = -(this.startX - e.targetTouches[0].clientX);
                     this.ul.style.left = this.startLeft + move_length + 'px';
                 }
                 //右に移動したか左に移動し方を検知する
                 move_length < 0 ? this.direction = 'left' : this.direction = 'right';
                
                 this.ulNotOver();
                 
              }
          },
          
          ulNotOver(){
             if(this.ulLeft()>0){
                 this.ul.style.left='0px';
             }else if(this.ulLeft() < -this.ul.clientWidth * (this.lis.length-1)/this.lis.length){
                 this.ul.style.left=-this.ul.clientWidth * (this.lis.length-1)/this.lis.length+'px';
             }
          },
          
          moveStart(e){
              e.preventDefault();
              if(this.isSmallWindow()){
                  this.nowMove = true;
                  this.startLeft=this.ulLeft();
                  if(e.type==='mousedown'){
                      this.startX = e.clientX;
                  }else if(e.type==='touchstart'){
                      this.startX = e.targetTouches[0].clientX
                  }
              }
          },
          
          ulLeft(){
             return  this.ul.style.left==='' ? 0 : Number(this.ul.style.left.replace('px',''));
          },
          
          
          //liの高さをアスペクト比に応じて変える
          setLiHeight(){
              let thisObj = this;
              //liの幅を取得
              this.li_width = this.lis[0].clientWidth;
              //liの高さをアスペクト比によって変更する
              Object.keys(this.lis).forEach(function(key){
                  thisObj.lis[key].style.height=thisObj.aspectRatio * thisObj.li_width + 'px';
                  thisObj.lis[key].style.marginBottom='20px';
              })
          },
          
          //ulの幅を設定する
          setUlWidth(){
              if(this.isSmallWindow()){
                  this.ul.style.width = this.div.clientWidth * this.lis.length + 'px';
              }else{
                  this.ul.style.width = '100%';
              }
          },
          //画面幅が小さいかどうかをチェック
          isSmallWindow(){
              return window.innerWidth < this.threshold;
          }
          
      },
      mounted(){
          //現在移動中かどうかのフラグ
          this.nowMove = false;
          //マウントした要素のpics属性から画像名一覧を取得する
          this.pics = JSON.parse(this.$el.parentNode.attributes.pics.nodeValue)
          //アスペクト比を取得
          this.aspectRatio = Number(this.$el.parentNode.attributes.aspectRatio.nodeValue)
          //マウントした要素を取得
          this.element = this.$el.parentElement;
          this.div = this.element.getElementsByTagName('div')[0];
          this.ul = this.div.getElementsByTagName('ul')[0];
          this.lis = this.ul.getElementsByTagName('li');
          //描画が終わったらスタイルを設定
          this.$nextTick(() => {
            this.setStyle();
        　})
        　//リサイズしたらスタイルを設定
          window.addEventListener('resize', this.setStyle,false)
          
          this.div.addEventListener('mousedown',this.moveStart,false);
          this.div.addEventListener('touchstart',this.moveStart,false);
          
          this.div.addEventListener('mousemove',this.movePics,false);
          this.div.addEventListener('touchmove',this.movePics,false);
          
          this.div.addEventListener('mouseup',this.moveEnd,false);
          this.div.addEventListener('touchend',this.moveEnd,false);
      }
    }
</script>

<style scoped>
    div{
        overflow:hidden;
        background-repeat:no-repeat;
        background-position:center;
        background-size: 200px 200px;
        
    }
    ul{
        padding:0;
        overflow:hidden;
        position:relative;
        left:0px;
    }
    li{
        box-sizing:border-box;
        border:solid 0px blue;
        background-size:contain;
        background-repeat:no-repeat;
        background-position: center;
        list-style:none;
    }

    
@media screen and (min-width:768px){
    ul{
        display:block;
        overflow:hidden;
    }
    li{
        width: 48%;
    }
    
    li:nth-child(2n+1){
        float:left;
        clear:both;
    }
    li:nth-child(2n){
        float:right;
    }
}
    

@media screen and (max-width:768px){
    ul{
        margin:0 auto;
        display:flex;
    }
    li{
        width:100%;
        height:200px;
        float:none;
    }
    
    
    
}
</style>