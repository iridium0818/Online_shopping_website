# 仿天猫商城  
迷你天猫商城是一个基于SSM框架的综合性B2C电商平台，需求设计主要参考天猫商城的购物流程：用户从注册开始，到完成登录，浏览商品，加入购物车，进行下单，确认收货，评价等一系列操作。
作为模拟天猫商城系统的核心组成部分之一，采用SSM框架的天猫数据管理后台包含商品管理，订单管理，类别管理，用户管理和交易额统计等模块，实现了对整个商城的一站式管理和维护。


# 部署方式
1.项目使用IntelliJ IDEA开发。  
2.项目数据库为MySQL，请在resources下的sql文件夹中下载sql，并导入到数据库中。  
3.使用IDEA打开项目后，在maven面板刷新项目，下载依赖包。  
4.在IDEA中启动springboot项目即可（run方式或debug方式都行）。  
5.账户名和密码详见resources下的sql文件夹中的sql文件（sql运行之后在admin表里有账户名和密码）。

注意事项：后台管理界面的订单图表没有数据为正常现象，该图表显示的为近7天的交易额。

---后台界面(部分)--- 访问地址 http://43.139.213.186:8082/admin/login  （账户名和密码在admin表里，如用户名：MRJIANG，密码：123456）

---前台界面(部分)--- 访问地址 http://43.139.213.186:8082/   (账户名和密码在user表里，如用户名：a120，密码：cc123.0)

---

# TmallDemo 项目调试与维护报告
**日期**： 2026年1月2日

## 1. 问题概述
本次调试主要解决了后台管理系统无法正常加载（500错误）、页面跳转失效（404错误）的问题。

## 2. 详细调试过程

### 2.1 后端：后台首页 500 Internal Server Error
*   **现象**：管理员登录成功后，访问 `/admin` 或 `/admin/home` 时，服务器返回 500 错误。
*   **排查**：
    *   查看服务器日志，捕获到 `org.apache.ibatis.binding.BindingException`。
    *   错误信息提示 `Invalid bound statement (not found): com.xq.tmall.dao.ProductMapper.selectTitle`。
*   **根因分析**：
    *   MyBatis 的 Mapper 接口（Java文件）中定义了 `selectTitle` 和 `getTotalByDate` 方法。
    *   对应的 XML 映射文件（`ProductMapper.xml` 和 `ProductOrderMapper.xml`）中缺失了这两个方法的 SQL 实现 (`<select>` 标签)。
*   **解决方案**：
    *   在 `src/main/resources/mapper/ProductMapper.xml` 中补全了 `selectTitle` 的 SQL 查询。
    *   在 `ProductOrderMapper.xml` 中补全了 `getTotalByDate` 的 SQL 查询（用于统计图表数据）。
    *   执行 `mvn clean package` 重新编译项目。

### 2.2 前端：管理页跳转 404 Not Found
*   **现象**：在后台管理界面点击“产品管理”、“分类管理”等菜单时，AJAX 请求返回 404。
*   **排查**：
    *   浏览器控制台显示请求路径错误，例如请求了 `/tmall/admin/product`，但服务器实际部署在根路径 `/` 下。
    *   检查 `base.js` 中的 `getPage` 函数，发现 URL 拼接逻辑依赖 `contextPath` 变量，导致路径解析不一致。
*   **解决方案**：
    *   修改 `base.js`，将 AJAX 请求的基础路径修正为绝对路径 `/admin/`，移除对 `contextPath` 的依赖。
    *   修改 `header.jsp`，在引入 JS 文件时添加时间戳参数（`?v=<%=System.currentTimeMillis()%>`），强制浏览器清除缓存，确保新逻辑生效。

## 3. 结论
经过修复，后台管理系统的首页统计图表可正常加载，所有子模块（分类、产品、用户、订单）的 AJAX 跳转均恢复正常。代码已同步至仓库，运行稳定。
