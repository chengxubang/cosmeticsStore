jsp+servlet实战编写购物商城
=


程序有问题，找[程序帮](http://ll032.cn/HZ6vHa)：QQ1022287044


项目介绍
----
> 本项目使用jsp+servlet+mysql架构搭建美妆购物商城，主要分为用户端和商城端，用户端主要有首页展示，商品信息、购物车、订单和个人中心几个大的模块，每个模块都包含主要的电商功能；商户端主要是对商品的增删改查，对商品销量的统计，订单管理以及公告管理几个核心板块。



开发环境：
-----
1. jdk 8
2. intellij idea
3. tomcat 8
4. mysql 5.7

所用技术：
-----
1. jsp+servlet
2. js+ajax
3. layui
4. jdbc+C3P0

运行效果
----

- 注册 

![注册](/image/注册.png)
- 用户信息 

![用户信息](/image/用户信息.png)
- 首页

![首页](/image/首页.png)
- 商品列表

![商品列表](/image/商品列表.png)
- 订单确认

![订单确认](/image/订单确认.png)
- 后端-销售榜单

![后端-销售榜单](/image/后端-销售榜单.png)


重要代码:
-----
1. 登录，权限角色控制：
```diff
public void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    // 1.获取登录页面输入的用户名与密码
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    // 2.调用service完成登录操作。
    UserService service = new UserService();
    try {
        User user = service.login(username, password);
        // 3.登录成功，将用户存储到session中.
        request.getSession().setAttribute("user", user);
        // 获取用户的角色，其中用户的角色分普通用户和超级用户两种
        String role = user.getRole();
        // 如果是超级用户，就进入到后台管理系统；否则进入我的账户页面
        if ("超级用户".equals(role)) {
            response.sendRedirect(request.getContextPath() + "/admin/login/home.jsp");
            return;
        } else {
            response.sendRedirect(request.getContextPath() + "/client/myAccount.jsp");
            return;
        }
    } catch (LoginException e) {
        // 如果出现问题，将错误信息存储到request范围，并跳转回登录页面显示错误信息
        e.printStackTrace();
        request.setAttribute("register_message", e.getMessage());
        request.getRequestDispatcher("/client/login.jsp").forward(request, response);
        return;
    }
}
// 退出登录，销毁用户缓存信息
public void doPost(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
    // 获取session对象.
    HttpSession session = request.getSession();		
     // 销毁session
    session.invalidate();
     // flag标识
    String flag = request.getParameter("flag");     
      // 重定向到首页
    if (flag == null || flag.trim().isEmpty()) {
        response.sendRedirect(request.getContextPath() + "/index.jsp");
    }
}
```

2. 购物车逻辑
```diff
// 后端数据组装
public void doPost(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
    // 1.获取美妆id
    String id = request.getParameter("id");
    // 2.根据id条件查询具体美妆参数信息
    ProductService service = new ProductService();
    try {
        Product p = service.findProductById(id);
        //3.将商品添加到购物车
        //3.1获得session对象
        HttpSession session = request.getSession();
        //3.2从session中获取购物车对象
        Map<Product, Integer> cart = (Map<Product, Integer>)session.getAttribute("cart");
        //3.3如果购物车为null,说明没有商品存储在购物车中，创建出购物车
        if (cart == null) {
            cart = new HashMap<Product, Integer>();
        }
        //3.4向购物车中添加商品
        Integer count = cart.put(p, 1);
        //3.5如果商品数量不为空，则商品数量+1，否则添加新的商品信息
        if (count != null) {
            cart.put(p, count + 1);
        }			
        session.setAttribute("cart", cart);
        response.sendRedirect(request.getContextPath() + "/client/cart.jsp");
        return;
    } catch (FindProductByIdException e) {
        e.printStackTrace();
    }
}
   
//jsp 页面数据渲染
<c:forEach items="${cart}" var="entry" varStatus="vs">
    <table width="100%" border="0" cellspacing="0">
        <tr>
            <td width="10%">${vs.count}</td>
            <td width="30%">${entry.key.name }</td>
            <td width="10%">${entry.key.price }</td>
            <td width="20%">
                <!-- 减少商品数量 -->
                <input type="button" value='-' style="width:20px"
                       onclick="changeProductNum('${entry.value-1}','${entry.key.pnum}','${entry.key.id}')">
                 <!-- 商品数量显示 -->
                <input name="text" type="text" value="${entry.value}" style="width:40px;text-align:center" />
                <!-- 增加商品数量 -->
                <input type="button" value='+' style="width:20px"
                       onclick="changeProductNum('${entry.value+1}','${entry.key.pnum}','${entry.key.id}')">
            </td>
            <td width="10%">${entry.key.pnum}</td>
            <td width="10%">${entry.key.price*entry.value}</td>
            <td width="10%">
                <!-- 删除商品 -->
                <a href="${pageContext.request.contextPath}/changeCart?id=${entry.key.id}&count=0"
                style="color:#FF0000; font-weight:bold" onclick="javascript:return cart_del()">X</a>
            </td>
        </tr>
    </table>
    <c:set value="${total+entry.key.price*entry.value}" var="total" />
</c:forEach>

```
 
3. 文字验证码生成
![文字验证码生成](文字验证.png)

```
public void doGet(HttpServletRequest request, HttpServletResponse response)throws ServletException, IOException {
    // 禁止缓存
    // response.setHeader("Cache-Control", "no-cache");
    // response.setHeader("Pragma", "no-cache");
    // response.setDateHeader("Expires", -1);
    int width = 180;
    int height = 30;
    // 步骤一 绘制一张内存中图片
    BufferedImage bufferedImage = new BufferedImage(width, height,
            BufferedImage.TYPE_INT_RGB);
    // 步骤二 图片绘制背景颜色 ---通过绘图对象
    Graphics graphics = bufferedImage.getGraphics();// 得到画图对象 --- 画笔
    // 绘制任何图形之前 都必须指定一个颜色
    graphics.setColor(getRandColor(200, 250));
    graphics.fillRect(0, 0, width, height);
    // 步骤三 绘制边框
    graphics.setColor(Color.WHITE);
    graphics.drawRect(0, 0, width - 1, height - 1);
    // 步骤四 四个随机数字
    Graphics2D graphics2d = (Graphics2D) graphics;
    // 设置输出字体
    graphics2d.setFont(new Font("宋体", Font.BOLD, 18));
    Random random = new Random();// 生成随机数
    int index = random.nextInt(words.size());
    String word = words.get(index-1);// 获得成语
    // 定义x坐标
    int x = 10;
    for (int i = 0; i < word.length(); i++) {
        // 随机颜色
        graphics2d.setColor(new Color(20 + random.nextInt(110), 20 + random
                .nextInt(110), 20 + random.nextInt(110)));
        // 旋转 -30 --- 30度
        int jiaodu = random.nextInt(60) - 30;
        // 换算弧度
        double theta = jiaodu * Math.PI / 180;
        // 获得字母数字
        char c = word.charAt(i);
        // 将c 输出到图片
        graphics2d.rotate(theta, x, 20);
        graphics2d.drawString(String.valueOf(c), x, 20);
        graphics2d.rotate(-theta, x, 20);
        x += 40;
    }
    // 将验证码内容保存session
    request.getSession().setAttribute("checkcode_session", word);
    // 步骤五 绘制干扰线
    graphics.setColor(getRandColor(160, 200));
    int x1;
    int x2;
    int y1;
    int y2;
    for (int i = 0; i < 30; i++) {
        x1 = random.nextInt(width);
        x2 = random.nextInt(12);
        y1 = random.nextInt(height);
        y2 = random.nextInt(12);
        graphics.drawLine(x1, y1, x1 + x2, x2 + y2);
    }
    // 将上面图片输出到浏览器 ImageIO
    graphics.dispose();// 释放资源
    ImageIO.write(bufferedImage, "jpg", response.getOutputStream());
}
```

项目总结
----
通过项目能学习一些java的基本数据渲染和读取，前后端数据传递除开request外也可以用session进行缓存交互，但是有点浪费缓存资源，不必要长久缓存数据可以及时清空
缓存，原生jsp+servlet组合框架也有很多不足，后期会改版成ssh或者ssm或者springboot版本的电商平台

