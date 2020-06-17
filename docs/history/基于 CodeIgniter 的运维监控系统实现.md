> CodeIgniter，后简称CI，是一个小巧而性能出众的PHP Web框架，典型的MVC架构，逻辑结构清晰、文档完整，有强大的类库支持，便于快速开发。

<br />

# 1. 前言

本项目是某银行数据中心运维监控系统，由于受生产环境限制，故只能因地制宜，部署于Linux系统，服务器Apache2，数据库为Oracle，选择完全能满足需求的CodeIgniter框架。



 ## 1.1 功能需求分析

- 用户登录功能
- 权限分组管理
- 功能菜单管理
- 系统各项指标监控
- 主机资源查询
- 设备维保信息管理



## 1.2 总体设计

### 总体架构设计

采用传统的三层架构设计，自下而上依次为持久层、业务逻辑层和表示层。

![clip_image002](http://47.94.165.69:3031/upload/aaa/a21d07e67b9e7.png)  

- **持久层**，采用Oracle数据库存储数据，通过轮询实时监控并获取各系统服务器的资源使用情况，将数据保存在数据库。

- **业务逻辑层**，该系统的核心层，主要由部署于apache2的CI组成，包括数据模型创建，控制逻辑的设计，数据可视化的实现等。

- **表示层**，通过浏览器向用户展示系统门户，完成系统与用户的各种交互功能。

  ​

### 功能划分及处理流程

+ 登录

  接收POST请求，验证数据库的用户名与密码，保存在session里，然后执行后面操作。很常规，毋庸赘言。

+ 权限分组管理

  系统的功能菜单与用户组的对应关系，保存在数据库中，可以进行CRUD操作。用户登陆时会验证用户权限，确定所属分组，然后获取相应的功能菜单。权限之外的操作将会被拒绝。

+ 功能菜单管理

  系统功能菜单保存在数据库，用户可以依据自身权限获取相应的菜单。超级用户可以对用户菜单进行CRUD等操作。

+ 系统各项指标监控

  用户登陆后，依其权限获取已保存在数据库里的相应系统指标，CPU使用率、内存使用率、数据库使用情况等。

+ 主机资源查询

  用户可查询各系统的主机资源，包括主机名，主机用途，IP，操作系统等。

+ 设备维保信息管理

  包括维保信息的CRUD操作，并能EXCEL批量导入维保信息。



### 系统功能全景图

![img](http://47.94.165.69:3031/upload/aaa/91f7e9d521562.png)  

​

## 1.3 项目文件结构

![dcys](http://47.94.165.69:3031/upload/aaa/b8270c574b3bd.PNG)  



## 1.4 系统的数据流程

![appflowchart](http://47.94.165.69:3031/upload/aaa/9ff3074c412ab.png)  

1. index.php 文件作为前端控制器，初始化运行 CodeIgniter 所需的基本资源；
2. Router 检查 HTTP 请求，以确定如何处理该请求；
3. 如果存在缓存文件，将直接输出到浏览器，不用走下面正常的系统流程；
4. 在加载应用程序控制器之前，对 HTTP 请求以及任何用户提交的数据进行安全检查；
5. 控制器加载模型、核心类库、辅助函数以及其他所有处理请求所需的资源；
6. 最后一步，渲染视图并发送至浏览器，如果开启了缓存，视图被会先缓存起来用于 后续的请求。


<br />



# 2. 核心架构及实现

CI 的开发基于 MVC（模型 - 视图 -控制器）设计模式。将应用程序的逻辑层和表现层分离开来。

- **模型** 代表你的数据结构。通常来说，模型类将包含帮助你对数据库进行增删改查的方法。

- **视图** 是要展现给用户的信息。一个视图通常就是一个网页，但是在 CI 中， 一个视图也可以是一部分页面（例如页头、页尾），它也可以是一个 RSS 页面， 或其他任何类型的页面。

- **控制器** 是模型、视图以及其他任何处理 HTTP 请求所必须的资源之间的中介，并生成网页。

  ​

## 2.1 Model层

- **BASE_Model**，由于所有的数据模型都有对数据库的CRUD操作，这些操作仅仅在表名、查询条件、查询字段等参数方面有差别，因此创建一个BASE_Model作为父辈模型，供其它模型继承。

```
class BASE_Model extends CI_Model{
  	protected $table_name = '';
  	
	function __construct(){
		$this->table_name = $this->table_name;
		parent::__construct();
	}

	/**
	 * 执行sql查询
	 * @param $where 		查询条件[例`name`='$name']
	 * @param $data 		需要查询的字段值[例`name`,`gender`,`birthday`]
	 * @param $limit 		返回结果范围[例：10或10,10 默认为空]
	 * @param $order 		排序方式	[默认按数据库默认方式排序]
	 * @param $group 		分组方式	[默认为空]
	 * @param $key          返回数组按键名排序
	 * @return array		查询结果集数组
	 */
	final public function select($where = '', $data = '*', $limit = '', $order = '', $group = '', $key='') {
      // ...
	}
	
	/**
	 * 执行插入操作insert
	 * @param $data 
	 */
	final public function insert($data){
		return $this->db->insert($this->table_name,$data);
 	}

	/**
	 * 批量insert
	 * @param $data 批量数组
	 */
	final public function insert_batch($data){
		return $this->db->insert_batch($this->table_name,$data);
	}
	/* 
	... 
	*/
}
```

- **用户模型**，Oracle数据库中的用户表如下所列。

```
CREATE TABLE `dc_sys_member` (
  `user_id` mediumint(8) unsigned NOT NULL AUTO_INCREMENT,
  `username` char(100) NOT NULL,
  `password` char(32) NOT NULL,
  `group_id` tinyint(3) unsigned DEFAULT '0',
  `avatar` varchar(50) DEFAULT NULL,
  `last_login_ip` char(15) DEFAULT NULL,
  `last_login_time` datetime DEFAULT NULL,
  `encrypt` varchar(50) DEFAULT NULL,
  `fullname` varchar(50) DEFAULT NULL,
  `qq` varchar(50) DEFAULT NULL,
  `created` datetime DEFAULT NULL,
  `modified` datetime DEFAULT NULL,
  `mobile` varchar(50) DEFAULT NULL,
  `sex` varchar(2) DEFAULT '0',
  PRIMARY KEY (`user_id`),
  UNIQUE KEY `username` (`username`(15)),
  KEY `groupID` (`group_id`)
) ENGINE=MyISAM AUTO_INCREMENT=5 DEFAULT CHARSET=utf8;
```

​	根据用户表创建数据模型。

```
class Member_model extends BASE_Model{
  public function __construct() {
    $this->table_name = 'DC_SYS_MEMBER';
    parent::__construct();
  }
  /* 
  ... 
  */
}
```

- **用户组模型**，给用户按权限角色分组，用户组表的字段如下。

```
CREATE TABLE `dc_sys_member_role` (
  `role_id` int(10) unsigned NOT NULL AUTO_INCREMENT COMMENT '组ID',
  `role_name` varchar(45) NOT NULL DEFAULT '' COMMENT '组名',
  `type_id` int(10) unsigned NOT NULL DEFAULT '0' COMMENT '保留',
  `description` varchar(200) DEFAULT NULL COMMENT '描述',
  PRIMARY KEY (`role_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8 ROW_FORMAT=FIXED;
```

​	创建用户组模型。

```
class Member_role_model extends BASE_Model{
  public function __construct() {
    $this->table_name = 'DC_SYS_MEMBER_ROLE';
    parent::__construct();
  }
  /* 
  ... 
  */
}
```

- **功能菜单模型**，菜单表的字段如下。字段中，CI 的 URI 路由由`controller/folder/method`来唯一确定；每条菜单entry的`arr_parentid`字段，从根开始依次保存其所有祖先的`menu_id`。

```
CREATE TABLE `dc_sys_module_menu` (
  `menu_id` smallint(6) unsigned NOT NULL AUTO_INCREMENT,
  `menu_name` char(40) NOT NULL DEFAULT '',
  `parent_id` smallint(6) NOT NULL DEFAULT '0',
  `controller` varchar(50) DEFAULT NULL,
  `folder` varchar(50) DEFAULT NULL,
  `arr_parentid` varchar(250) DEFAULT NULL,
  `arr_childid` varchar(250) DEFAULT NULL,
  `is_parent` tinyint(1) DEFAULT '0',
  PRIMARY KEY (`menu_id`),
  KEY `parent_id` (`parent_id`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8
```

​	创建功能菜单模型。

```
class Member_role_model extends BASE_Model{
  public function __construct() {
    $this->table_name = 'DC_SYS_MEMBER_ROLE';
    parent::__construct();
  }
  /* 
  ... 
  */
}
```

- **用户组-菜单关系模型**，用户组-菜单关系表的字段如下。

```
CREATE TABLE `dc_sys_member_role_priv` (
  `role_id` tinyint(3) unsigned NOT NULL DEFAULT '0',
  `folder` varchar(50) NOT NULL DEFAULT '',
  `controller` varchar(50) NOT NULL DEFAULT '',
  `method` varchar(50) NOT NULL DEFAULT '',
  `data` varchar(50) NOT NULL DEFAULT '',
  `priv_id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `menu_id` int(11) DEFAULT '0',
  PRIMARY KEY (`priv_id`),
  KEY `role_id` (`role_id`,`folder`,`controller`,`method`)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

​	创建用户组-菜单关系模型。

```
class Member_role_priv_model extends BASE_Model{
  public function __construct() {
    $this->table_name = 'DC_SYS_MEMBER_ROLE_PRIV';
    parent::__construct();
  }
  /* 
  ... 
  */
}
```

- **其他功能模型**，其他功能于系统的主要框架关联性不大，可按需扩展，不再赘述。



## 2.2 Controller层

* **BASE_Controller**，创建一个父辈控制器，包含一些所有功能模块都共有的`method`，例如通过session查看登陆状态，检查用户权限，获取功能菜单，显示全局提示，生成视图等。

```
class BASE_Controller extends CI_Controller{

  function __construct(){
    parent::__construct();
    $this->check_member();
    $this->check_priv();
  }
  
  /*
   * 判断用户是否已经登陆
   */
   protected function check_member() {
   	$_datainfo = $this->Member_model->get_one(array('USER_ID'=>$this->user_id,'USERNAME'=>$this->user_name));
	if(!($this->page_data['folder_name']=='adminpanel'&&$this->page_data['controller_name']=='manage'&&$this->page_data['method_name']=='login')&&!$_datainfo){
    	redirect(site_url('adminpanel/manage/login'));
		exit(0);
	}
   }

  /*
   * 检查用户权限
   */
  protected function check_priv(){
    $r =$this->Member_role_priv_model->get_one(array('METHOD'=>$this->page_data['method_name'],'CONTROLLER'=>$this->page_data['controller_name'] ,'FOLDER'=>$this->page_data['folder_name'],'ROLE_ID'=>$this->group_id ));
    if(!$r) exit("您没有权限操作该项");
  }
  /* 
  ... 
  */
  /**
   * view 自动模板调用
   */
  protected function view($view_file,$sub_page_data=NULL){
    $this->load->view("header");
    $this->load->view($view_file, $sub_page_data);
    $this->load->view("footer");
  }
}
```

* **其他控制器**，其他控制器皆继承自`BASE_Controller`，可按照不同的功能需求创建，每一个`method`，对应一个菜单entry，其中包含`controller/folder/method`的唯一路由。

  ​

## 2.3 View层

一个视图基本上就是一个网页或网页的一部分，根据路由调用Controller里的对应method，会加载view模板，并将从服务器端得到的数据渲染在该模板内。通常会对一个网页进行切割，方便共用Header，Footer，Sidebar，Breadcrumb，Nav等 。

![dcsys-1](http://47.94.165.69:3031/upload/aaa/a65cc5a169fe2.PNG) 

![dcys-2](http://47.94.165.69:3031/upload/aaa/c48defd3083b5.PNG) 

![dcys-3](http://47.94.165.69:3031/upload/aaa/83132a41aa451.999Z%20copy) 

![dcys-4](http://47.94.165.69:3031/upload/aaa/f780a55ca6578.PNG) 

![dcys-4](http://47.94.165.69:3031/upload/aaa/482632cc54c84.741Z%20copy) 

## 2.4 Libraries 类库

自定的功能函数库，方便在某些methods里调用。例如Menu_tree，作用是根据用户权限，获取用户的功能菜单树。

```
Class Menu_tree {

  //把tree展开成一维数组便于view，见 generate_treeview_arr
  public $treeview_arr = array();
  /**
  *根据节点从root开始建造目录树
  * $result 必须 menu_id ASC
  */
  public function build($result){
    $menu=array();
    foreach ($result as $r) {
      $menu_id = $r["MENU_ID"];
      $path = explode(",",$r["ARR_PARENTID"]);
      $obj_r = (object)null;
      $obj_r->item = $r;
      if($r["IS_PARENT"]) $obj_r->children = array();
      $tmp=&$menu;
      for($i=1;$i<count($path);$i++){
        $tmp=&$tmp[$path[$i]]->children;
      }
      $tmp[$menu_id] = $obj_r;
    }
    return $menu;
  }

 /**
 *  把tree展开成一维数组便于view
 *  展开之后 保存在 treeview_arr
 */
  public function generate_treeview_arr($menu_tree){
    foreach ($menu_tree as $value) {
      $obj=(object)null;
      $nbsp="";
      for ($i=1; $i<count(explode(",",$value->item["ARR_PARENTID"])); $i++) 
      	$nbsp.= "&nbsp;<i class='fa fa-i-cursor'>&nbsp;</i><i class='fa fa-ellipsis-h'></i>";
      $obj->nbsp=$nbsp."&nbsp;";
      if(property_exists((object)$value->item,"CHECKED")) 
      	$obj->checked = $value->item["CHECKED"];
      $obj->val = $value->item;
      $this->treeview_arr[] = $obj;
      if(property_exists($value,"children")){
        $this->generate_treeview_arr($value->children);
      }
    }
  }
}

```

<br />



## 2.5 Helpers 辅助函数

包含了自定义的全局functions，例如下面两个。

```
/**
* 安全过滤函数
*/
if(! function_exists('safe_replace')){
  function _safe_replace($string) {
    $string = str_replace('%20','',$string);
    $string = str_replace('%27','',$string);
    $string = str_replace('%2527','',$string);
    $string = str_replace('*','',$string);
    $string = str_replace('"','&quot;',$string);
    $string = str_replace("'",'',$string);
    $string = str_replace('"','',$string);
    $string = str_replace(';','',$string);
    $string = str_replace('<','&lt;',$string);
    $string = str_replace('>','&gt;',$string);
    $string = str_replace("{",'',$string);
    $string = str_replace('}','',$string);
    return $string;
  }
  
  function safe_replace($string) {
    if(!is_array($string)) return _safe_replace($string);
    foreach($string as $key => $val) $string[$key] = _safe_replace($val);
    return $string;
  }
}

/**
* 判断是不是ajax请求
*/
if(!function_exists('is_ajax_request')){
  function is_ajax_request(){
    return isset($_SERVER['HTTP_X_REQUESTED_WITH']) && $_SERVER['HTTP_X_REQUESTED_WITH'] ='xmlhttprequest';
  }
}

```

<br />



# 3. 仍需改进之处 

* 主页需要重新设计
* 新增和编辑功能菜单可以做得更细致、更灵活
* 系统指标和主机资源的查询SQL优化
* Web前端的图形化展示可以更丰富更友好






