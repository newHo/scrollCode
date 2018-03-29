<template>
  <div class="Home">
   	<div class="content">
   		 <ul class="leftList">
  	    	<li v-for="(singList,index) in leftList" :class="{'active':index == activedIndex}" @click="changeIndex(index)">
            <p>{{singList.name}}</p>
            <span class="totalNumIcon" v-if="singList.totalCountOff">{{singList.totalCount}}</span>
          </li>
  	    </ul>
          <div class="rightLists" @scroll="fingerScroll($event)" ref="rightLists">
              <div class="sectionList" v-for="(singList,index) in leftList" ref="lists">
                <dl>{{singList.name}}</dl> 
                <dt v-for="(item,singleIndex) in singList.foods" class="smallList">
                    <p>{{item.name}}</p>
                    <p>{{item.description}}</p>
                    <div class="icons">
                      <i class="iconfont icon-jianhao" v-if="item.off" @click="subtract(index,singleIndex)"></i>
                      <span v-if="item.count>0">{{item.count}}</span>
                      <i class="iconfont icon-jiahao" @click="increase(index,singleIndex)"></i>
                    </div>
                </dt>
              </div>  
          </div>
   	</div>
   
  </div>
</template>

<script>
import leftList from '../data/foodsJson.js'
export default {

  name: 'Home',
  data () {
    return {
      leftList,
      activedIndex:0, //被选中下标
      listHeight:[],   //列表高度集合数组
      fingerTimer:null,
      btnTimer:null
    }
  },
  methods:{
    //点击按钮改变下标，执行动画
    changeIndex:function(index){
      //当前点击
      var nextIndex = index;
      //点击上一次
      var prevIndex = this.activedIndex;
      //选中下标
      this.activedIndex = index;
      //右侧对应被选中的列表项
      let sectionList = this.$refs.lists[index];
      //右侧容器节点
      let rightLists = this.$refs.rightLists; 
       var _this = this;
       if(nextIndex > prevIndex){
            this.tapBtnScroll(rightLists,sectionList,15);

       }else{
            this.tapBtnScroll(rightLists,sectionList,-15);   
       }
    },
    //滑动动画封装
    tapBtnScroll(parentNode,selectedList,num){
        let _this = this;
        clearInterval(this.btnTimer);
        this.btnTimer = setInterval(function(){
            parentNode.scrollTop += num;
            if(num > 0){
              if(parentNode.scrollTop + parentNode.offsetTop >= selectedList.offsetTop || parentNode.scrollTop + parentNode.clientHeight >= selectedList.offsetTop){
                clearInterval(_this.btnTimer);
                parentNode.scrollTop = selectedList.offsetTop;
              }
            }else{
              if(parentNode.scrollTop + parentNode.offsetTop <= selectedList.offsetTop){
                clearInterval(_this.btnTimer);
                parentNode.scrollTop = selectedList.offsetTop;
              }
            }

        },1)
    },
    //手指滑动联动按钮
    fingerScroll(event){
        this.listHeightAll;
        //滚动距离
        let scrollValue = event.srcElement.scrollTop;
        var _this = this;
        clearTimeout(this.fingerTimer);
        this.fingerTimer = setTimeout(function(){
            let newScrollValue = event.srcElement.scrollTop;
            if(newScrollValue == scrollValue){
              let length = _this.listHeight.length;
              for(let i = 0;i<length-1;i++){
                let height1 = _this.listHeight[i];
                let height2 = _this.listHeight[i+1];
                if(!height2 || (scrollValue + 2 >= height1 && scrollValue < height2)){
                  _this.activedIndex = i;
                  clearTimeout(this.fingerTimer);
                }
             }
            }
        },10)
    },
    //单个商品的增加
    increase(index,singleIndex){
      this.leftList[index].foods[singleIndex].count += 1;
      this.leftList[index].foods[singleIndex].off = true;
      this.common(index);
    },
    //单个商品的减少
    subtract(index,singleIndex){
      this.leftList[index].foods[singleIndex].count -= 1;
      if(this.leftList[index].foods[singleIndex].count == 0){
        this.leftList[index].foods[singleIndex].off = false; 
      }
      this.common(index);
    },
    //一类商品的总数量
    common(index){
      let num = 0;
      let lists = this.leftList[index];
      let listFood = lists.foods;
      listFood.forEach((item) => {
        num += item.count;
      })
      lists.totalCount = num;
      if(lists.totalCount > 0){
          lists.totalCountOff = true;
      }else{
          lists.totalCountOff = false;
      }
    }
  },
  //获取食品分类的高度范围数组
  computed:{
    listHeightAll(){
      //右侧列表dom对象集合
      let foodList = this.$refs.lists;
      let height = 0;
      this.listHeight.push(height);
      for(let i = 0;i < foodList.length;i++){
          let itemList = foodList[i];
          height += itemList.offsetHeight;
          this.listHeight.push(height);
      }
    }
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  @import url("//at.alicdn.com/t/font_508700_ay0shufmrilwstt9.css");

  .content{
    display: flex;
  }
  .leftList{
    width:30%;
    margin-right: 0.25rem;
  }
  .leftList li{
  	height:2rem;
  	line-height: 2rem;
    text-align: center;
    margin:.25rem;
    position: relative;
  	border:1px solid #ccc;
  }
  .active{
    background-color: #e8e8e8;
  }

  .rightLists{
    flex:1;
    height:100vh;
    overflow-y: scroll;
  }
  .sectionList{
    font-size: .8rem;
  }
  .sectionList dl{
    height:2rem;
    line-height: 2rem;
    background-color: skyblue;
  }
  .smallList{
    height:4rem;
    padding: 0.5rem 0 0 0.5rem;
    border:1px solid #f7f7f7;
    position: relative;
  }

  i{
    font-size: 1rem;
    color:#17abe3;
  }
  .icons{
    position: absolute;
    bottom: 5px;
    right:5px;
  }
  .totalNumIcon{
    display: inline-block;
    width:1rem;
    height:1rem;
    line-height: 1rem;
    position: absolute;
    right:1px;
    top:0;
    border-radius: 50%;
    background-color: lightblue;
  }
</style>
