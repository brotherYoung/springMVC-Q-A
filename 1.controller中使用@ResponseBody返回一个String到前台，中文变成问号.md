#controller中使用@ResponseBody返回一个String到前台，中文变成问号。

首先我们来看一段代码:

    $('#query').click(function() {
    	var topic = $('#topic').val();
        $.ajax({
        	  type: "POST",
            url: "findNewsByTopic.do?topic=" + topic,
            dataType: "json",
            success: function(data) {
            	alert(data);
            	$.each(data, function(index, object) {
            		$('.table').append('<tr>' + '<td>'+object.authorname+'</td>' + '</tr>');
                });
            	alert("ok"); 
            },
            error: function() {
            	alert("error");
            }
        });
    });
前台通过jquery ajax向后台发送一个请求。


    @RequestMapping(value = "/findNewsByTopic.do")
  	public @ResponseBody String findNewsByTopic(@RequestParam String topic, HttpServletResponse response, HttpServletRequest request) {
  		List<News> list = new ArrayList<News>();
  		list = newsDao.findNewsByTopic(topic);
  		System.out.println(JSON.toJSONString(list));
  		return JSON.toJSONString(list);
  	}
后台controller接收前台传来的请求，调用dao层查询数据库，将查出来的List转为json（利用alibaba的fastjson进行转换），然后返回给前台
ajax接收，@ResponseBody该注解用于将Controller的方法返回的对象，通过适当的HttpMessageConverter转换为指定格式后，写入到Response
对象的body数据区。

结果呢，前台收到的json格式的数据中，中文全部变成了“？”问号。这里向大家提个醒，发现中文变问号，都是因为编码ISO系列导致的！

原因如下：这可以说是spring mvc的一个bug，spring MVC有一系列HttpMessageConverter去处理用@ResponseBody注解的返回值，
如返回list则使用MappingJacksonHttpMessageConverter，返回string，则使用StringHttpMessageConverter，这个convert使用的
是字符集是iso-8859-1,而且是final的。

解决办法：网上有很多中办法，修改StringHttpMessageConverter的默认字符集，不用@ResponseBody注解，将返回值设为void，用下面一段代码向前台
返回json。
      
      response.setContentType("application/json; charset=UTF-8");
      response.getWriter().print(jsonStr);
    
**看这里**，最终网上找到一位大神的解决办法：

    StringHttpMessageConverter默认iso-8859-1编码，但是会根据请求头信息指定的编码格式来转换，所以只需要在ajax请求的时候指定
    头信息Accept属性
    $.ajax({
    ...
    headers: { 
    Accept : "text/plain; charset=utf-8",
    }
    即可
  
问题完美解决！！！！！！！！

最终代码如下：（后台controller不用变）

    $('#query').click(function() {
    	var topic = $('#topic').val();
        $.ajax({
        	type: "POST",
            url: "findNewsByTopic.do?topic=" + topic,
            dataType: "json",
            headers: { 
            	Accept : "text/plain; charset=utf-8",
            	},
            success: function(data) {
            	alert(data);
            	$.each(data, function(index, object) {
            		$('.table').append('<tr>' + '<td>'+object.authorname+'</td>' + '</tr>');
                });
            	alert("ok"); 
            },
            error: function() {
            	alert("error");
            }
        });
    });
