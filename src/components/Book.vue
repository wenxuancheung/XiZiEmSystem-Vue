<template>
	<div>
		<a @click="routerBack" class="backLast"></a>
		<div class="time" :class="{timeRed : isRed}">
			<h2 :class="{timeDis : isDis}">剩余点餐时间</h2><br />
			<span >{{book_hour}}{{book_min}}</span>
		</div>
		<div class="book clearfix">
			<div id="lunch"><p>午餐</p></div>
			<div id="menu" class="clearfix">
				<form action="" id="Book_form">
					<ul v-for="(item) in food_list">
						<li ><input type="radio" name="foods" :value="item" v-model="item_food">{{item}}</li>
					</ul>
				</form>
				<input type="button"  id="booking" value="下单" class="btn" :disabled="btnDis" @click="bookFoods"/>
			    <input type="button"  id="delete"  value="删除" class="btn" :disabled="btnDis" @click="deleteFoods"/>
			</div>
		</div>
		<div class="show">
			<div id="showMenu">
				<p>下单列表</p>	
			</div>
			<div id="showItems">
				<ul v-for="food_name in food_items">
					<li>{{food_name}}</li>
				</ul>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: 'Book',
	data () {
	    return {
	    	food_list : [],
	    	user_time : [],
	    	my_hour : "",
	    	my_date : "",
	    	my_minutes : "",
	    	book_hour : "",
	    	book_min : "",
	    	isRed : false,
	    	isDis : true,
	    	btnDis : false,
	    	item_food : "",
	    	food_items :[],
	    	show_food : "",
	    	food_name : "",
	    	i : 0
	     // msg: 'Welcome to Your Vue.js App'
	    }
	},
	create : function(){
	},
	beforeMounted : function(){
		
	},
	mounted : function(){
		//计时器
		this.getTime()
		this.getFood()
	},
	methods : {
		routerBack : function(){
			this.$router.go(-1)
		},
		getTime : function(){
			let _this = this
			let my_date
			let my_hour
			let my_minutes
			let year
			let month
			let day
			let i
			let hour_time
			let min_time
			let time
			this.$http.get('static/index.json').then(function(result){
				_this.user_time = result.body.times
				//循环获取当前时间并显示倒计时
				setInterval(function(){
					my_date = new Date()
					my_hour = my_date.getHours()
					my_minutes = my_date.getMinutes()
					year = my_date.getFullYear()
					month = my_date.getMonth()+1
					day = my_date.getDate()
					if(my_hour < _this.user_time[0].hour && my_minutes < _this.user_time[0].min){
					time = new Date(year+'/'+month+'/'+day+ ' '+_this.user_time[0].hour+':'+_this.user_time[0].min+':00');
					}else{
						day += 1
						time = new Date(year+'/'+month+'/'+day+ ' '+_this.user_time[0].hour+':'+_this.user_time[0].min+':00');
					}
					i = time.getTime() - my_date.getTime()
					hour_time =(i-i%(1000*60*60))/(60*1000*60)%24
					min_time = (i-i%(1000*60))/(60*1000)%60
					if(hour_time > _this.user_time[0].hour){
						_this.book_hour = "已过点餐时间"
						_this.isRed = true
					}else if(hour_time ==_this.user_time[0].hour && min_time >=_this.user_time[0].min){
						_this.book_hour ="已过点餐时间"
						_this.isRed = true
					}else{
						_this.book_hour = hour_time+"时"
						_this.book_min = min_time+"分"
						_this.isDis = false
					}
					if(my_hour > _this.user_time[0].hour){
						_this.btnDis = true
					}else if(my_hour == _this.user_time[0].hour && my_minutes > _this.user_time[0].min-1){
						_this.btnDis = true
					}
				},1000)
			})

		},
		getFood : function(){
			var _this = this
			let new_food = []
			let x
			let y = 0
			let rand
			let rand2
			this.$http.get('static/index.json').then(function(result){
				_this.food_list = result.body.foods
				new_food = _this.food_list
				x = new_food.length-1;
				rand = parseInt(Math.random() * (x - y + 1) + y)
				rand2 = parseInt(Math.random() * (x - y + 1) + y)
				if(rand==rand2 && rand==y){
					rand2 = rand + 1;
				}
				else if(rand==rand2 && rand==x){
					rand2 = rand - 1;
				}
				else if(rand==rand2){
					rand = rand - 1;
				}
				_this.food_list = [new_food[rand],new_food[rand2]]
			})
		},
		bookFoods : function(){
			let _this = this
			let food 	//保存食物
			let name  	//保存人名
			let time   	//保存时间
			let my_date = new Date();
			let my_hour = my_date.getHours();	    //获取当前小时数用于判断
			let my_minutes = my_date.getMinutes(); //获取当前分钟数用于判断
			let name_num
			let random
			let confirm_msg
			let x
			let y = 0
			let rand
			this.$http.get('static/index.json').then(function(result){
				food = result.body.foods
				name = result.body.names
				time = result.body.times
				if(my_hour > time){
					_this.btnDis = true
				}else if(my_hour == time[0].hour && my_minutes > time[0].min-1){
					_this.btnDis = true
				}
				if(_this.btnDis == true){
					alert("已过点餐时间")
				}else{
					//点击下单
					name_num = name.length-1
					random = parseInt(Math.random() * name_num)
					if(this.item_food === ""){
						alert("你没有选择菜品")
					}else{
						confirm_msg = confirm("你确定要点"+this.item_food+"吗?")
						if(confirm_msg){
							x = name.length-1;
							rand = parseInt(Math.random() * (x - y + 1) + y)
							this.show_food = name[rand]+" "+this.item_food
							this.food_items[this.i] = this.show_food
							this.i++
						}
					}
				}
			})
		},
		deleteFoods : function(){
			//获取点击delete时的时间
			let my_date = new Date()
			let my_hour = my_date.getHours()	    //获取当前小时数用于判断
			let my_minutes = my_date.getMinutes() //获取当前分钟数用于判断
			if(my_hour > this.user_time){
				this.btnDis = true
			}else if(my_hour == this.user_time && my_minutes > this.user_time-1){
				this.btnDis = true
			}
			if(this.btnDis == true){
				alert("点餐时间已过，你不可以删除订单")
			}else{
				//点击删除订单
				if(this.food_items == ""){
				    alert("没有可删除的订单")
				}
				else{
				    //原生confirm
					var confirm_msg = confirm("你确定要删除吗?")
					if(confirm_msg){
						var index = this.food_items.indexOf(this.show_food)
						this.food_items.splice(index,1)
					}		
				}
			}	
		}

	}
}
</script>

<style scoped>
/*
 * 点餐时间
 */
.time{
	text-align: center;
	margin : 1rem auto 0;
	font : bold;
	height: 9rem;
	width: 80vw;
	border : solid 1px;
}
.time span{
	font-size: 4rem;
}
.time h2{
	margin-top: 2rem;
}
.timeRed {
	background-color: red;
	font-size: 4rem
}
.timeDis {
	display: none
}
/*
 * 选项
 */
.book{
	margin: 2rem auto 0;
	height: 25rem;
	width: 90vw;
	border: solid 0.1rem;
}
#lunch{
	width: 100%;
	height: 25%;
	background-color: rgb(65,199,219);
}
#lunch p{
	line-height: 6rem;
	text-indent: 0.5em;
	font-size: 1.6rem;
}
#menu{
	text-align: center;
	margin-top: 2rem;
	font-size: 2.5rem;
}
.btn{
	width: 35%;
	height: 5rem;
	border-radius: 0.8rem;
	font-size: 1.6rem;
}
#booking{
	margin :4.5rem 0 0px 0.8rem ;
	float: left;
}
#delete{
	margin :4.5rem 0.8rem 0px 0px ;
	float: right;
	color:red;
	border-color: red;
}
.btnDis{
	disabled : true;
}
#menu li{
	margin-top: 2rem
}

/*
 * 下单列表
 */
.show{
	
	margin: 2rem auto 0;
	height: 20rem;
	width: 90vw;
	border: solid 1px;
}
#showMenu{
	width: 100%;
	height: 5rem;
	background-color: rgb(65,199,219);
}
#showMenu p{
	font-size: 1.6rem;
	line-height: 5rem;
	text-indent: 0.5em;
}
#showItems{
    height: 65%;
	overflow: auto;
	margin-top: 1.5rem;
	font-size: 1.6rem;
	text-align: center;
}
#showItems li{
	margin-top :1rem;
}
</style>