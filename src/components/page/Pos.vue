<template>
  <div class="pos">
    <el-row>
      <el-col :span="7" class="pos-order" id="order-list">
        <el-tabs type="card">
          <el-tab-pane label="点餐">
            <el-table border style="width:100%" :data="tableData">
              <el-table-column label="商品名称" prop="goodsName"></el-table-column>
              <el-table-column label="数量" prop="count" width="50"></el-table-column>
              <el-table-column label="金额" prop="price" width="70"></el-table-column>
              <el-table-column label="操作" width="100" fixed="right">
                <template v-slot="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totalDiv">
              <small> 数量</small>：{{totalCount}}    <small>金额</small> ：{{totalMoney}} 元
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">
            挂单
          </el-tab-pane>
          <el-tab-pane label="外卖">
            外卖
          </el-tab-pane>
        </el-tabs>
        <div class="div-button">
          <el-button type="warning">挂单</el-button>
          <el-button type="danger" @click="delAllGoods">删除</el-button>
          <el-button type="success" @click="checkout">结账</el-button>
        </div>
      </el-col>
      <el-col :span="17">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="goods in oftenGoods" :key="goods.goodsId" @click="addOrderList(goods)">
                <span>{{goods.goodsName}}</span>
                <span class="o-price">￥{{goods.price}}</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="goods-type">
          <el-tabs type="card">
            <el-tab-pane label="汉堡">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type0Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                    <span class="foodImg"><img alt="图片" :src="goods.goodsImg" width="100%"/></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="小食">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type1Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                    <span class="foodImg"><img alt="图片" :src="goods.goodsImg" width="100%"/></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="饮料">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type2Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                    <span class="foodImg"><img alt="图片" :src="goods.goodsImg" width="100%"/></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
            <el-tab-pane label="套餐">
              <div>
                <ul class="cookList">
                  <li v-for="goods in type3Goods" :key="goods.goodsId" @click="addOrderList(goods)">
                    <span class="foodImg"><img alt="图片" :src="goods.goodsImg" width="100%"/></span>
                    <span class="foodName">{{goods.goodsName}}</span>
                    <span class="foodPrice">{{goods.price}}元</span>
                  </li>
                </ul>
              </div>
            </el-tab-pane>
          </el-tabs>
        </div>
      </el-col>
      
    </el-row>

    
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'pos',
  data(){
    return {
        tableData:[],
        oftenGoods:[],
        type0Goods:[],
        type1Goods:[],
        type2Goods:[],
        type3Goods:[],
        totalMoney:0,
        totalCount:0
    }
  },
  created:function(){
    //获取常用商品
    axios.get('https://yapi.baidu.com/mock/67005/kabuto/kabuto91')
    .then(res=>{
      console.log(res);
      this.oftenGoods = res.data.oftenGoods;
    })
    .catch(err=>{
      console.log(err);
      alert('网络错误，不能访问');
    })

    //获取分类商品数据
    axios.get('https://yapi.baidu.com/mock/67005/kabuto/kabuto91')
    .then(res=>{
      // console.log(res.data.typeGoods);
      this.type0Goods = res.data.typeGoods[0];
      this.type1Goods = res.data.typeGoods[1];
      this.type2Goods = res.data.typeGoods[2];
      this.type3Goods = res.data.typeGoods[3];
      
    })
    .catch(err=>{
      console.log(err);
      alert('网络错误，不能访问');
    })
  },
  mounted:function(){
    var orderHeight = document.body.clientHeight;
    console.log(orderHeight);
    document.getElementById('order-list').style.height = orderHeight + 'px';
  },
  methods:{
    addOrderList(goods){

      this.totalMoney = 0;
      this.totalCount = 0;
      //商品是否已经存在于订单列表中
      let isHave = false;
      for(let i=0;i<this.tableData.length;i++){
        if(this.tableData[i].goodsId==goods.goodsId){
          isHave = true;
        }
      }

      //根据判断的值编写业务逻辑
      if(isHave){
        //改变列表中商品数量
        let arr = this.tableData.filter(o=>o.goodsId == goods.goodsId);
        arr[0].count++;
      }else{
        let newGoods = {goodsId:goods.goodsId,goodsName:goods.goodsName,price:goods.price,count:1};
        this.tableData.push(newGoods);
      }
      this.getAllMoney();

    },
    //删除单个商品
    delSingleGoods(goods){
      console.log(goods);
      this.tableData = this.tableData.filter(o=>o.goodsId != goods.goodsId);
      this.getAllMoney();
    },
    delAllGoods(){
      this.tableData = [];
      this.totalMoney = 0;
      this.totalCount = 0;
    },
    //模拟结账
    checkout(){
      if(this.totalCount != 0){
        this.tableData = [];
        this.totalMoney = 0;
        this.totalCount = 0;
        this.$message({
          message:'结账成功，感谢你又为店里除了一份力！',
          type:'success'
        })
      }else{
        this.$message.error('不能空结，老板了解你急切的心情')
      }
    },
    //汇总数量和金额
    getAllMoney(){
      this.totalCount = 0;
      this.totalMoney = 0;
      if(this.tableData){
        //计算汇总金额数量
        this.tableData.forEach((element)=>{
          this.totalCount+=element.count;
          this.totalMoney=this.totalMoney+(element.price*element.count);
        });
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

.pos-order{
  background-color: #F9FAFC;
  border-right: 1px solid #C0CCDA;
}
.div-button{
  margin-top: 20px;
}
.title{
  height: 20px;
  border-bottom: 1px solid #D3DCE6;
  background-color: #F9FAFC;
  padding: 10px;
}
.often-goods-list ul li{
  list-style: none;
  float: left;
  border: 1px solid #E5E9F2;
  padding: 10px;
  margin: 5px;
  background-color: #fff;
}
.o-price{
  color: #58B7FF;
}
.goods-type{
  clear: both;
}
.cookList li{
  list-style: none;
  width: 23%;
  border: 1px solid #E5E9F2;
  height: auto;
  overflow: hidden;
  background-color: #fff;
  padding: 2px;
  float: left;
  margin: 2px;
  cursor: pointer;
}
.cookList li span{
  display: block;
  float: left;
}
.foodImg{
  width: 40%;
}
.foodName{
  font-size: 16px;
  padding-left: 10px;
  color: brown;
}
.foodPrice{
  font-size: 16px;
  padding-left: 10px;
  padding-top: 10px;
}
.totalDiv{
  background-color: #fff;
  padding: 10px;
  border-bottom: 1px solid #D3dce6;
}
</style>
