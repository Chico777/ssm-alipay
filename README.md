

#### 1、项目架构
![这里写图片描述](https://img-blog.csdn.net/20180620155335825?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

- 项目架构：spring+springmvc+mybatis
- 数据库：mysql
- 部署环境：tomcat9.0
- 开发环境：jdk9、idea
- 支付：支付宝、微信


整合到ssm一样，我们需要像沙箱测试环境一样，需要修改**支付的配置信息**
![这里写图片描述](https://img-blog.csdn.net/20180620170214281?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

#### 2、数据库代码

主要包括以下的数据库表：

- user：用户表
- order：支付产生的订单
- flow：流水账
- product：商品表：用于模拟购买商品。
```
drop table if exists user;

/*==============================================================*/
/* Table: user                                                  */
/*==============================================================*/
create table user
(
   id                   varchar(20) not null,
   username             varchar(128),
   sex                  varchar(20),
   primary key (id)
);

alter table user comment '用户表';


CREATE TABLE `flow` (
  `id` varchar(20) NOT NULL,
  `flow_num` varchar(20) DEFAULT NULL COMMENT '流水号',
  `order_num` varchar(20) DEFAULT NULL COMMENT '订单号',
  `product_id` varchar(20) DEFAULT NULL COMMENT '产品主键ID',
  `paid_amount` varchar(11) DEFAULT NULL COMMENT '支付金额',
  `paid_method` int(11) DEFAULT NULL COMMENT '支付方式\r\n            1：支付宝\r\n            2：微信',
  `buy_counts` int(11) DEFAULT NULL COMMENT '购买个数',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='流水表';


CREATE TABLE `orders` (
  `id` varchar(20) NOT NULL,
  `order_num` varchar(20) DEFAULT NULL COMMENT '订单号',
  `order_status` varchar(20) DEFAULT NULL COMMENT '订单状态\r\n            10：待付款\r\n            20：已付款',
  `order_amount` varchar(11) DEFAULT NULL COMMENT '订单金额',
  `paid_amount` varchar(11) DEFAULT NULL COMMENT '实际支付金额',
  `product_id` varchar(20) DEFAULT NULL COMMENT '产品表外键ID',
  `buy_counts` int(11) DEFAULT NULL COMMENT '产品购买的个数',
  `create_time` datetime DEFAULT NULL COMMENT '订单创建时间',
  `paid_time` datetime DEFAULT NULL COMMENT '支付时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单表';


CREATE TABLE `product` (
  `id` varchar(20) NOT NULL,
  `name` varchar(20) DEFAULT NULL COMMENT '产品名称',
  `price` varchar(11) DEFAULT NULL COMMENT '价格',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='产品表 ';


```

#### 3、支付宝支付controller（支付流程）

**支付流程图**
![这里写图片描述](https://img-blog.csdn.net/20180620165147752?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

首先，启动项目后，输入http://localhost:8080/,会进入到商品页面，如下：
![这里写图片描述](https://img-blog.csdn.net/20180620160715724?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

下面是页面代码

**商品页面（products.jsp）**
![这里写图片描述](https://img-blog.csdn.net/20180620160814279?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<%@ taglib uri="http://java.sun.com/jsp/jstl/functions" prefix="fn" %>  
<script src="<%=request.getContextPath() %>/static/js/jquery.min.js" type="text/javascript"></script>

<html>

    <head>
        
    </head>
    
    <body>
        
        <table>
        	<tr>
        		<td>
        			产品编号
        		</td>
        		<td>
        			产品名称
        		</td>
        		<td>
        			产品价格
        		</td>
        		<td>
        			操作
        		</td>
        	</tr>
	        <c:forEach items="${pList }" var="p">
	        	<tr>
	        		<td>
	        			${p.id }
	        		</td>
	        		<td>
	        			${p.name }
	        		</td>
	        		<td>
	        			${p.price }
	        		</td>
	        		<td>
	        			<a href="<%=request.getContextPath() %>/alipay/goConfirm.action?productId=${p.id }">购买</a>
	        		</td>
	        	</tr>
	        	
	        </c:forEach>
        </table>
        
        <input type="hidden" id="hdnContextPath" name="hdnContextPath" value="<%=request.getContextPath() %>"/>
    </body>
    
</html>


<script type="text/javascript">

	$(document).ready(function() {
		
		var hdnContextPath = $("#hdnContextPath").val();
		
		
	});
	

</script>


```

点击上面的**购买**，进入到**订单页面**

![这里写图片描述](https://img-blog.csdn.net/20180620161120463?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


填写**个数**，然后点击**生成订单**，调用如下代码

![这里写图片描述](https://img-blog.csdn.net/20180620161348895?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
根据`SID`（生成id的工具）等信息生成订单，保存到数据库。

进入到**选择支付页面**

![这里写图片描述](https://img-blog.csdn.net/20180620161452619?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

调用了如下代码：

![这里写图片描述](https://img-blog.csdn.net/20180620161534326?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


然后，我们选择**支付宝支付**，进入到了我们支付的页面了，大功告成！
![这里写图片描述](https://img-blog.csdn.net/201806201616503?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpaGFpMTIzNDU=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

调用了如下代码：

```
/**
	 *
	 * @Title: AlipayController.java
	 * @Package com.sihai.controller
	 * @Description: 前往支付宝第三方网关进行支付
	 * Copyright: Copyright (c) 2017
	 * Company:FURUIBOKE.SCIENCE.AND.TECHNOLOGY
	 *
	 * @author sihai
	 * @date 2017年8月23日 下午8:50:43
	 * @version V1.0
	 */
	@RequestMapping(value = "/goAlipay", produces = "text/html; charset=UTF-8")
	@ResponseBody
	public String goAlipay(String orderId, HttpServletRequest request, HttpServletRequest response) throws Exception {

		Orders order = orderService.getOrderById(orderId);

		Product product = productService.getProductById(order.getProductId());

		//获得初始化的AlipayClient
		AlipayClient alipayClient = new DefaultAlipayClient(AlipayConfig.gatewayUrl, AlipayConfig.app_id, AlipayConfig.merchant_private_key, "json", AlipayConfig.charset, AlipayConfig.alipay_public_key, AlipayConfig.sign_type);

		//设置请求参数
		AlipayTradePagePayRequest alipayRequest = new AlipayTradePagePayRequest();
		alipayRequest.setReturnUrl(AlipayConfig.return_url);
		alipayRequest.setNotifyUrl(AlipayConfig.notify_url);

		//商户订单号，商户网站订单系统中唯一订单号，必填
		String out_trade_no = orderId;
		//付款金额，必填
		String total_amount = order.getOrderAmount();
		//订单名称，必填
		String subject = product.getName();
		//商品描述，可空
		String body = "用户订购商品个数：" + order.getBuyCounts();

		// 该笔订单允许的最晚付款时间，逾期将关闭交易。取值范围：1m～15d。m-分钟，h-小时，d-天，1c-当天（1c-当天的情况下，无论交易何时创建，都在0点关闭）。 该参数数值不接受小数点， 如 1.5h，可转换为 90m。
    	String timeout_express = "1c";

		alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\","
				+ "\"total_amount\":\""+ total_amount +"\","
				+ "\"subject\":\""+ subject +"\","
				+ "\"body\":\""+ body +"\","
				+ "\"timeout_express\":\""+ timeout_express +"\","
				+ "\"product_code\":\"FAST_INSTANT_TRADE_PAY\"}");

		//请求
		String result = alipayClient.pageExecute(alipayRequest).getBody();

		return result;
	}
```







