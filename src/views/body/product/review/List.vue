<template>
    <div class="main container one-column content" ref="scrollContainer">
        <div class="col-main">
            <div class="std">
                <div class="review_lists">
                    <div class="review_list_product" style="width:100%">
                        <div class="row">
                            <div class="col-20">
                                <router-link :to="'/catalog/product/'+product.product_id"  >
                                    <img :src="product.imgUrl">
                                </router-link>
                                
                            </div>
                            <div class="col-80">
                                <router-link :to="'/catalog/product/'+product.product_id"  class="product_name">
                                    {{product.name}}
                                </router-link>
                                
                                
                                <div class="product_info review_add_price">
                                    <div class="price_info" v-if="product.price_info && product.price_info.special_price">
                                        <div class="special_price special_active">
                                            {{product.price_info.special_price.symbol}}
                                            {{product.price_info.special_price.value}}
                                        </div>
                                        <div  class="price special_active">
                                            {{product.price_info.price.symbol}}
                                            {{product.price_info.price.value}}
                                        </div>
                                        <div class="clear"></div>
                                    </div>
                                    
                                    <div class="price_info" v-else-if="product.price_info && product.price_info.price" >
                                        <div class="price no-special">
                                            {{product.price_info.price.symbol}}
                                            {{product.price_info.price.value}}
                                        </div>
                                    </div>
                                </div>
                            </div>
                        </div>
                        
                        <div class="review_cart">
                            <div style="margin:20px 0 0">
                                <div class="rbc_cold">
                                    <span>
                                        <span class="average_rating">Average rating :</span>
                                        <span :class="'review_star review_star_' + reviw_rate_star_average" style="font-weight:bold;" itemprop="average"></span>  
                                        
                                        (<span itemprop="count">{{review_count}} reviews</span>)
                                        
                                        
                                    </span>
                                </div>					
                                
                                <div class="content-block">
                                    <div class="row">
                                        <div class="col-50">
                                            <router-link :to="'/catalog/product/'+product.product_id"   class="submitbutton button  button-fill button-success">
                                                <span><span>Add To Cart</span></span> 
                                            </router-link>
                                        </div>
                                        <div class="col-50">
                                            <router-link style="margin-left:10px" :to="'/product/review/add/'+product.product_id"   class="submitbutton button  button-fill button-danger">
                                                <span><span>Add Review</span></span> 
                                            </router-link>
                                        </div>
                                    </div>
                                </div>
                                
                            </div>
                        </div>
                        <div class="clear"></div>
                    </div>
                    <div class="product-reviews"> 
                        <div class="clear"></div>
                        <div class="review_title">
                            <a external href="javascript:void(0)">Product Review</a>
                        </div>
                        
                        <div v-if="reviewList.length > 0" class="product-Reviews_top">
                            <template  v-for="(item_one ,t_index) in reviewList">     
                                <div class="card">
                                    <div class="fec-card-header">
                                        {{item_one.summary}}
                                    </div>
                                    <div class="fec-card-content">
                                        <div class="fec-card-content-inner">
                                            <div class="review-content">
                                                {{item_one.review_content}}
                                            </div>
                                            <div class="moderation">
                                                
                                                <div v-if="item_one.status == 10 ">
                                                    Your Review is awaiting moderation...
                                                </div>
                                                
                                                <div v-if="item_one.status == 2 ">
                                                    Your Review is refused...
                                                </div>
                                            </div>
                                            <div class="review_list_remark">
                                                <p>By{{item_one.name}}</p>
                                                <span>{{item_one.review_date}}</span>
                                            </div>
                                        </div>
                                    </div>
                                    <div class="fec-card-footer">
                                        <a href="#" :class=" 'review_star review_star_'+ item_one.rate_star" onclick="javascript:return false;"></a>
                                    </div>
                                </div>
                            </template>
                            <!-- 加载提示符 -->
                            <mugen-scroll :handler="fetchReview" :should-handle="!loading" scroll-container="scrollContainer">
                                <div style="display:none;"  class="infinite-scroll-preloader">
                                    <div class="preloader"></div>
                                </div>
                            </mugen-scroll>
                        </div>
                        
                    </div>
                </div>
            </div>
        </div>
    </div>
   
</template>
<script>
var root = process.env.API_ROOT;
import MugenScroll from 'vue-mugen-scroll'
export default {
    data () {
        return {
            pageInitUrl: root + '/catalog/reviewproduct/lists' ,
            errormsg:'',
            reviewList:[],
            product:{},
            review_count:0,
            reviw_rate_star_average:0,
            count:0,
            loading: false ,
            correctmsg:''
        }
    },
    created: function(){
        this.pageInit();
    },
    components:{
        'mugen-scroll': MugenScroll
    },
    methods: {
        pageInit: function(){
            var product_id = this.$route.params.product_id;
            var self = this;
            self.errormsg = '';
            self.correctmsg = '';
            $.showIndicator();
            $.ajax({
                url: self.pageInitUrl,
                async: true,
                timeout: 120000,
                type: 'get',
                headers: self.getRequestHeader(),
                data:{ 
                    product_id:product_id,
                },
                success:function(reponseData, textStatus,request){
                    if(reponseData.code == 200){
                        self.reviewList = reponseData.data.reviewList;
                        self.product = reponseData.data.product;
                        self.review_count = reponseData.data.review_count;
                        self.reviw_rate_star_average = reponseData.data.reviw_rate_star_average;
                        self.count = 1;
                        console.log('get editAccount info success');
                        self.saveReponseHeader(request); 
                    }
                    $.hideIndicator();
                },
                error:function(){
                    $.toast("system error");
                    $.hideIndicator();
                    console.log('get address list page init error');
                }
            });
            
        },
        fetchReview: function(){
            var product_id = this.$route.params.product_id;
            var self = this;
            if(self.count >=1){
                self.loading = true;
                self.errormsg = '';
                self.correctmsg = '';
                $.showIndicator();
                $.ajax({
                    url: self.pageInitUrl,
                    async: true,
                    timeout: 120000,
                    type: 'get',
                    headers: self.getRequestHeader(),
                    data:{ 
                        product_id:product_id,
                        p: self.count+1
                    },
                    success:function(reponseData, textStatus,request){
                        if(reponseData.code == 200){
                            var reviewList = reponseData.data.reviewList;
                            if(reviewList.length > 0){
                                for(var x in reviewList){
                                    self.reviewList.push(reviewList[x]);
                                }
                                self.loading = false;
                                console.log('fetch review loading false');
                                $.init();
                                self.count++;
                            }
                            self.saveReponseHeader(request); 
                        }
                        $.hideIndicator();
                    },
                    error:function(){
                        $.toast("system error");
                        $.hideIndicator();
                        console.log('get address list page init error');
                    }
                });
            }
        }
    }
}
</script>