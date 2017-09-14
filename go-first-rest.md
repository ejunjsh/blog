---
title: 用go实现一个简单的restful接口
date: 2017-07-18 01:01:18
tags: [go,restful]
categories: [go]
---
# 前言
go的标准库`http`已经封装好很多接口，可以很简单实现一个web服务器。
````go
// 定义 handler
func HelloServer(w http.ResponseWriter, req *http.Request) {
    io.WriteString(w, "hello, world!\n")
}

func main() {
    //绑定pattern和handler
    http.HandleFunc("/hello", HelloServer)
    err := http.ListenAndServe(":12345", nil)
    if err != nil {
        log.Fatal("ListenAndServe: ", err)
    }
}
````
基于上面例子可以封装一个restful接口，不是难事。
<!-- more -->
# 实现
从上面例子可以看到，一个url pattern对应一个handler，即对应一个处理，就可以处理http请求了，所以下面的实现是基于对这两个东西的封装开始
## 封装一个restful app 结构
````go
type App struct {
    //一个map，key是pattern，value是handler
	handlers map[string]func(r *HttpRequest,w HttpResponse)error
    //pattern数组，用来保证加入pattern的顺序，因为上面的map是无顺序的
	patterns []string
    //一个map，key是pattern，value是http method
	methods map[string]string
    //用来实现在url path取出参数的。
	regexps map[string]*regexp.Regexp
	pathparamanmes map[string][]string
    //用来处理异常的handler
	errHandler func( err error, r *HttpRequest,w HttpResponse)
}
````
初始化函数
````go
func NewApp() *App {
	return &App{
		handlers: make(map[string]func(r *HttpRequest,w HttpResponse)error),
		patterns:make([]string,0),
		methods:make(map[string]string),
		regexps:make(map[string]*regexp.Regexp),
		pathparamanmes:make(map[string][]string),
        //一个默认的异常处理，直接返回异常内容
		errHandler: func(err error, r *HttpRequest, w HttpResponse) {
			w.Write( []byte(err.Error()))
		},
	}
}
````
## 映射绑定
````go
func(a *App) handle(method string,pattern string, handler func(r *HttpRequest,w HttpResponse) error){
    //绑定pattern和handler
	a.handlers[pattern]=handler
    //绑定pattern和method
	a.methods[pattern]=method
    //绑定pattern 正则，用来匹配url pattern,和获取url path 参数
	a.regexps[pattern],a.pathparamanmes[pattern]=convertPatterntoRegex(pattern)
	for _,s:=range a.patterns{
		if s==pattern{
			return
		}
	}
    //加入数组，方便用此数组确定顺序
	a.patterns=append(a.patterns,pattern)
}

//绑定GET
func (a *App) Get(pattern string, handler func(r *HttpRequest,w HttpResponse)error)  {
	a.handle("GET",pattern,handler)
}
//绑定POST
func (a *App) Post(pattern string, handler func(r *HttpRequest,w HttpResponse)error)  {
	a.handle("POST",pattern,handler)
}
//绑定DELETE
func (a *App) Delete(pattern string, handler func(r *HttpRequest,w HttpResponse)error)  {
	a.handle("DELETE",pattern,handler)
}
//绑定PUT
func (a *App) Put(pattern string, handler func(r *HttpRequest,w HttpResponse) error)  {
	a.handle("PUT",pattern,handler)
}

func (a *App) Error(handler func(err error,r *HttpRequest,w HttpResponse))  {
	a.errHandler=handler
}
````
有了Restful接口的四个方法映射绑定，剩下的就要请求能进到来，所以接下来要写个入口才行。

## 编写http入口
````go
//http 入口
func(a *App) Run(address string) error{
	fmt.Printf("Server listens on %s",address)
	err:=http.ListenAndServe(address,&hodler{app:a})
	if err!=nil{
		return err
	}
	return nil
}

//https 入口
func(a *App) RunTls(address string,cert string,key string) error{
	fmt.Printf("Server listens on %s",address)
	err:=http.ListenAndServeTLS(address,cert,key,&hodler{app:a})
	if err!=nil{
		return err
	}
	return nil
} 
````
入口函数主要调用`http`库来启动http服务，然后把请求处理函数作为`ListenAndServe`第二个参数传入。这里由`holder`来实现这个处理函数。
````go
func (h *hodler) ServeHTTP(w http.ResponseWriter, r *http.Request){
	//封装一下，附加更多功能
    request:= newHttpRequest(r)
	response:=newHttpResponse(w)
	//捕获panic，并让errhandler处理返回。
    defer func() {
		if err:=recover();err!=nil{
			if e,ok:=err.(error);ok{
				h.app.errHandler(InternalError{e,""},request,response)
			}
			if e,ok:=err.(string);ok{
				h.app.errHandler(InternalError{nil,e},request,response)
			}
		}
	}()
    //根据pattern的添加顺序，循环判断
   for _,p:=range h.app.patterns{
       if reg,ok:= h.app.regexps[p];ok{
           //匹配method
		   if method,ok:=h.app.methods[p];ok&&r.Method==method{
              //匹配pattern
			   if reg.Match([]byte(r.URL.Path)) {
                   //抽取url path parameters
				   matchers:=reg.FindSubmatch([]byte(r.URL.Path))
				   pathParamMap:=make(map[string]string)
				   if len(matchers)>1{
                       if pathParamNames,ok:=h.app.pathparamanmes[p];ok{
						   for i:=1;i<len(matchers);i++{
							   pathParamMap[pathParamNames[i]]=string(matchers[i])
						   }
					   }
				   }
                   //PathParams是封装后的request独有的属性
				   request.PathParams=pathParamMap
				   if handler,ok:=h.app.handlers[p];ok{
                       //执行handler
					   err:=handler(request,response)
					   if err!=nil{
                           //执行errhandler
						   h.app.errHandler(err,request,response)
					   }
					   return
				   }
			   }
		   }
	   }
   }
   //执行no found errhandler
	h.app.errHandler(NoFoundError{},request,response)
}
````
基本一个请求的流程如下：
requset->ServeHTTP()->匹配url pattern->匹配method->匹配到你的handler->执行你的handler->你的handler返回结果

## 返回结果
由于返回结果可以有很多，所以封装了`http`库的`http.ResponseWriter`来实现`WriteString,WriteJson,WriteXml,WriteFile`等方法。
````go
//封装request，附件一个PathParams来保存url path parameters.
type HttpRequest struct {
	*http.Request
	PathParams map[string] string
}

type HttpResponse struct {
	http.ResponseWriter
}
//用来返回字符
func (response *HttpResponse) WriteString(str string) error {
	...
}
//返回JSON
func (response *HttpResponse) WriteJson(jsonObj interface{}) error {
	...
}
//返回XML
func (response *HttpResponse) WriteXml(xmlObj interface{}) error {
	...
}
//返回文件
func (response *HttpResponse) WriteFile(filepath string) error {
    ...
}
//返回一个模板html
func (response *HttpResponse) WriteTemplates(data interface{},tplPath ...string) error  {
	...
}
````
## 例子
````go
func main(){
    //new 一个restful接口
	app:=gorest.NewApp()
    //绑定
	app.Get("/json", func(r *gorest.HttpRequest, w gorest.HttpResponse) error {
		a:= struct {
			Abc string `json:"abc"`
			Cba string `json:"cba"`
		}{"123","321"}
        //返回json作为结果
		return w.WriteJson(a)
	})
	app.Error(func(err error, r *gorest.HttpRequest, w gorest.HttpResponse){
		if e,ok:=err.(gorest.NoFoundError);ok {
			w.Write([]byte(e.Error()))
		}
		if e,ok:=err.(gorest.InternalError);ok {
			w.Write([]byte(e.Error()))
		}
	})

    //启动
	app.Run(":8081")
}
````
收工😄
# 总结
go的标准库封装了很多了，所以实现这个其实还是比较轻松的😄
详细代码见https://github.com/ejunjsh/gorest
