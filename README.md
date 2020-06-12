<p align="center">
  <img width="320" src="http://bgk.nmm80.com/admin_xpx/images/login.png">
</p>

简体中文 | [English](./README.md) | 

## 简介 作者-刘玉全
[driver_salary_online](https://panjiachen.github.io/vue-element-admin) 是一个可前端可后台分离的demo，
它基于 [vue](https://cn.vuejs.org/) 和 [element-ui](https://element.eleme.cn/2.13/#/zh-CN)实现。
重新封装了利用vue混入（mixin）封装全局参数以及函数 [xVue](./js/XVue.js)，当前小胖熊后台vue+element-ui开发方式较多，但当前自带后台会有全局样式影响组件样式
所以重新封装header ，方便后台开发，也便于前端接收开发，前后台分离式开发

## 目录
driver_salary_online
- css
    - list.less    引入[less](http://lesscss.cn/)可动态控制样式（包含了一些基础样式）
- js 
    -   [api](./js/api.js) 封装接口 多种请求方式
    -   [request](./js/request.js) 重新封装axios （对于200、 500做特殊处理）引入Qs解决post请求传输数据问题
    -   [XVue](./js/XVue.js) 重新封装[vue](https://cn.vuejs.org/)
   
- [list](./list.html)  示例页面

- [header](./header.htm) 主要引入文件不可缺少  重新封装页面header便于前端直接开发，做到完全前后台分离开发
                        （引入js文件下所有js，css下所有less、css）

## 后台开发使用   
[php]    
    
        原先方式渲染页面
        $ur_here = '司机申诉审核详情';        
        $url = 'driver_salary_online/driver_appeal_exa_view.html';
        $smarty->assign('ur_here', $ur_here);
        
        
 [html]   
  
     引入新的header
      {include file="./header.htm"}
           <div id='app></div>
        <script>
          let opts = {
              el: '#app',
              data() {
                  return {},
                  watch:{},
                  created(){},
                  methods : {
                      test()
                      {
                          console.log(_that, '使用全局_that需要在created之后')
                      }
                  }
              }
          }
         const _that = new  XVue(opts) // 实例vue
        </script>
        
        
## 前端开发使用
[html]

    直接引入--需要注意路径
    <title>网站标题</title>
    <meta name="robots" content="noindex, nofollow">
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <script src="https://unpkg.com/axios/dist/axios.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/vue"></script>
    <!-- 引入qs 序列化post参数-->
    <script src="https://cdn.bootcss.com/qs/6.7.0/qs.min.js"></script>
    <!-- 引入组件库 -->
    <script src="https://unpkg.com/element-ui/lib/index.js"></script>
    <link rel="stylesheet/less" type="text/css" href="./css/list.less" />
    <link rel="stylesheet" href="https://unpkg.com/element-ui/lib/theme-chalk/index.css">
    <script src="//cdnjs.cloudflare.com/ajax/libs/less.js/3.11.1/less.min.js" ></script><!-- 引入样式 -->
    <!-- 二次封装的vue 包含 $post请求和 $get请求 -->
    <script src="./js/XVue.js"></script>
    <script src="./js/request.js"></script>
    <script src="./js/api.js"></script>
    <style> 
        [v-cloak]{
            display: none;
        }
    </style>

[request详解]

    /* 封装axios */
    const url = document.location.toString().split('//');
    const host = url[0] + '//' + url[1].substr(0, url[1].indexOf('/'));
    const service = axios.create({
        baseURL: host, // 根据不同需求做出改动
        timeout: 10000 // 超时控制
    })
    axios.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded'
    service.interceptors.request.use(
        config => {
            config.data  = config.data || {}
            // post setting
            if( config.method == 'post') {
                config.data = Qs.stringify(config.data) // post 特殊处理
            }
            return config
        },
        error => {
            console.log(error) // for debug
            return Promise.reject(error)
        }
    )
    // response interceptor
    service.interceptors.response.use(
        /**
         * Determine the request status by custom code
         * Here is just an example
         * You can also judge the status by HTTP Status Code
         */
        response => {
        
            const res = response.data
            // 判断相应参数 做不同操作
            if (res.code === 200 || res.code === '200' || res.code === 100) {
                if (res.message || res.msg) {
                    x_vue.$message({
                        message: res.message || res.msg,
                        type: 'success'
                    })
                }
            } else if(res.code == 500 ){
                x_vue.$message({
                    message: res.message || res.msg || '请求失败',
                    type: 'error'
                })
            } else  {
                x_vue.$message({
                    message: res.message || res.msg || '请求失败',
                    type: 'error'
                })
                return Promise.reject(new Error({message: res.message || 'Error', data:[]}))
            }
            return res
        },
        error => {
            console.log('err' + error) // for debug
            x_vue.$message({
                message: error.message,
                type: 'error'
            })
            return Promise.reject(error)
        }
    )
    
    /*通用 get 请求*/
    const $GET = (url, params) => {
        return service({
            method: 'get',
            url: url,
            params: params
        })
    }
    /*通用 post 请求*/
    const $POST = (url, data) => {
        return service({
            method: 'post',
            url: url,
            data: data
        })
    }

[XVue代码详解]
    
    // 全局混入对象
    const  mixin_obj = {
        data() {
            return {
                layout:  'total, sizes, prev, pager, next, jumper' ,
                pageSizes: [10, 20, 40, 50 ,100, 200],
                pickerOptions: {
                    disabledDate(time) {
                        return time.getTime() > Date.now();
                    },
                    shortcuts: [{
                        text: '今天',
                        onClick(picker) {
                            picker.$emit('pick', new Date());
                        }
                    }, {
                        text: '昨天',
                        onClick(picker) {
                            const date = new Date();
                            date.setTime(date.getTime() - 3600 * 1000 * 24);
                            picker.$emit('pick', date);
                        }
                    }, {
                        text: '一周前',
                        onClick(picker) {
                            const date = new Date();
                            date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
                            picker.$emit('pick', date);
                        }
                    }]
                }
            }
        },
        created: function() {
        },
        /*公用函数*/
        methods : {
              // 获取连接参数 返回对象数组
             parseQuery(option){
                const res = {};
                const query = (location.href.split("?")[1] || "")
                    .trim()
                    .replace(/^(\?|#|&)/, "");
                if (!query) {
                    return res;
                }
                query.split("&").forEach(param => {
                    const parts = param.replace(/\+/g, " ").split("=");
                    const key = decodeURIComponent(parts.shift());
                    const val = parts.length > 0 ? decodeURIComponent(parts.join("=")) : null;
                    if (res[key] === undefined) {
                        res[key] = val;
                    } else if (Array.isArray(res[key])) {
                        res[key].push(val);
                    } else {
                        res[key] = [res[key], val];
                    }
                });
                return res;
            },
            //对象判断
            emptyObject(obj) {
                if(typeof obj == "undefined" || obj == null || obj == ""|| obj == {}){
                    return true;
                }else{
                    return false;
                }
            },
            //数组 和字符串判断 不包含 0的判断
            empty(data){
                return (data === "" || data === undefined || data === null ||data.length == 0 ) ? true : false;
            }
        },
        /*公用过滤函数 请参考vue文档*/
        filter: {  
        }
    }
    // 混入
    let  x_vue  = {}
    function  XVue(obj){
        // obj.mixins =[ mixin_obj]
        obj.mixins = [mixin_obj]
        Vue.config.debug = true;
        const vue = new Vue(obj);
        vue.delimiters = ['${','}'];
        x_vue  = vue
        return vue
    }


