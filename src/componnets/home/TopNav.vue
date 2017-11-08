<template>
	<div>
		<el-col :span="4" style="font-size:20px;">
			<!--<img src="../assets/logo4.png" class="logo">-->
			<router-link to="/main" class="project_name">{{ProjectName}}</router-link>
		</el-col>
		<el-col :span="16">
			<el-menu theme="dark" default-active="1" class="el-menu-demo" mode="horizontal"  id="topNavBox" unique-opened router>
				<template v-for="(item,index) in menuList">
					<li class="top-ment-item el-menu-item" :index="index+''"  v-on:click="topNavClick(index+'')">{{item.name}}</li>
				</template>
			</el-menu>
		</el-col>
		<el-col :span="4" class="rightbar">
			<el-dropdown trigger="click">
				<span class="el-dropdown-link" style="color:#c0ccda;cursor: pointer;"><img src="../../assets/user.png" class="head"> {{name}}</span>
				<el-dropdown-menu slot="dropdown">
					<el-dropdown-item  @click.native="logout">退出登录</el-dropdown-item>
				</el-dropdown-menu>
			</el-dropdown>
		</el-col>
	</div>
</template>

<script>
    import { mapState } from 'vuex'
    import user from '../../api/user'
    export default {
        data() {
            return {
                name:'',
                menuList:' ',
                path:'',
                indexpath:'',
                topName:'' //顶部一级菜单名称
            }
        },
        computed: {
                ...mapState([
                    'ProjectName'
                ])
        },
        methods: {
            topNavClick(a) {
                //this.$bus.$emit('checkLeftNav', a);
                this.$bus.$emit('checkLeftNav',a,'',this.menuList[a].list);
            },
            //获取一级菜单索引
            getTopNavIndex(name,arr){
                var index = null;
                for(var i = 0; i<arr.length; i++){
                    if(arr[i].name ==name){
                        index = i;
                        break;
                    }
                }
                return index;
            },
            //退出登录
            logout() {
                var _this = this;
                this.$confirm('确认退出吗?', '提示', {
                    //type: 'warning'
                }).then(() => {
                    user.delInfo();
                    _this.$router.replace('/login');
                }).catch(() => {

                });
            },
            getMenu(){
                var _this = this;
                var userID = _this.$Utils.getCookie("UserLoginID");
                if(userID){
                    _this.path = _this.$route.path; //获取当前路由路径
                    if(_this.path.lastIndexOf('/') > 1){
                        _this.path = _this.path.slice(0,_this.path.lastIndexOf('/'));
                        if(_this.path.lastIndexOf('/') > 1){
                            _this.path = _this.path.slice(0,_this.path.lastIndexOf('/'));
                        }
                    }
                    // console.log(222)
                    //console.warn(this.$route.matched)
                    _this.indexpath = _this.$route.matched[0].meta.index||0; //一级菜单索引，发现不正确
                    _this.topName = _this.$route.matched[0].name; //一级菜单名称
                    // console.log(_this.topName)
                    //console.log(_this.$route.path);
                    // console.log(_this.$route.matched[0].meta.index);
                    user.getUserMenuList(userID).then(res => {
                        if(res&&res.length>0){
                        for(var i = 0; i < res.length; i++){
                            if( res[i].name=="风控"){
                                _this.menuList=res[i].list;
                                var index = _this.getTopNavIndex(_this.topName,_this.menuList); //获取一级菜单索引
                                if(_this.path =='/main'){
                                    _this.$bus.$emit('checkLeftNav',0,'',_this.menuList[0].list);
                                }else{
                                    _this.$bus.$emit('checkLeftNav',index,_this.path,_this.menuList[index].list);
                                }
                                //_this.$bus.$emit('checkLeftNav',_this.indexpath,_this.path,_this.menuList[_this.indexpath].list);
                                var data = res[i].list,NavData=[];
                                if(data.length>0){
                                    for(var i in data){
                                        if(data[i].list.length>0){
                                            for(var j in data[i].list){
                                                if(data[i].list[j].list.length>0){
                                                    for(var k in data[i].list[j].list){
                                                        NavData.push(data[i].list[j].list[k].url);//遍历三级菜单获得路由路径
                                                    }
                                                }
                                            }
                                        }
                                    }
                                }
                                _this.$Utils.setCookie("UserPermissions",NavData);//登记用户权限
                            }
                        }
                    }
                }).catch(function(error){
                        _this.$Error(_this,error);
                    })
                }
            },
            findNav(obj,arr){
                var boole = false;
                for(var i=0;i<arr.length;i++) {
                    if(arr[i]==obj||obj.indexOf(arr[i])>-1){
                        boole = true;
                    }
                }
                return boole;
            }
        },
        mounted() {
            var UserName=this.$Utils.getCookie("UserName");
            this.name = UserName;
            this.getMenu();
        }
    }
</script>

<style scoped>
	#topNavBox{background: url(../../../static/images/header_bg.jpg) no-repeat center top!important; color: #fff!important;}
	#topNavBox .top-ment-item.el-menu-item{color: #fff;}
	#topNavBox .top-ment-item.el-menu-item:hover{background-color: #4db3ff!important;}
	.panel-top .rightbar{  text-align: right;  padding-right: 35px;  }
	.panel-top .rightbar .head{  width: 40px;  height: 40px;  border-radius: 20px;  margin: 10px 0px 10px 10px;  float: right;  }
	.tip-logout i {  cursor: pointer;  }
	.top-ment-item{ font-size: 16px;}
	.project_name{ color: #fff; padding-left: 20px; text-decoration:none; width: 210px; display:inline-block;}
</style>