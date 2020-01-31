## SpringDatJPA关联映射操作

### 一对一的关联关系
需求： 用户与角色的一对一的关联关系
用户： 一方
角色： 一方

#### 创建一对一关联关系
```java
@Entity
@Table(name="t_users")
public class Users implements Serializable{

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)//strategy=GenerationType.IDENTITY 自增长
	@Column(name="userid")
	private Integer userid;
	
	@Column(name="username")
	private String username;
	
	@Column(name="userage")
	private Integer userage;
	
	@OneToOne(cascade=CascadeType.PERSIST)
	//@JoinColumn：就是维护一个外键
	@JoinColumn(name="roles_id")
	private Roles roles;
}
```

```java
@Entity
@Table(name="t_roles")
public class Roles {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	
	@Column(name="rolename")
	private String rolename;
	
	@OneToOne(mappedBy="roles")
	private Users users;	
}
```

cascade属性：  
* CascadeType.ALL         所有操作  
* CascadeType.PERSIST     保存  
* CascadeType.MERGE       修改  
* CascadeType.REMOVE      删除  


#### 一对一关联关系操作
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class OneToOneTest {

	@Autowired
	private UsersDao usersDao;
	
	/**
	 * 添加用户同时添加角色
	 */
	@Test
	public void test1(){
		//创建角色
		Roles roles = new Roles();
		roles.setRolename("管理员");
		
		//创建用户
		Users users = new Users();
		users.setUserage(30);
		users.setUsername("赵小刚");
		
		//建立关系
		users.setRoles(roles);
		roles.setUsers(users);
		
		//保存数据
		this.usersDao.save(users);
	}
	
	/**
	 * 根据用户ID查询用户，同时查询用户角色
	 */
	@Test
	public void test2(){
		Users users = this.usersDao.findOne(13);
		System.out.println("用户信息："+users);
		Roles roles = users.getRoles();
		System.out.println(roles);
	}
}
```

### 一对多的关联关系
需求： 从角色到用户的一对多的关联关系
角色： 一方
用户： 多方

#### 创建一对多关联关系
```java
@Entity
@Table(name="t_users")
public class Users implements Serializable{

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)//strategy=GenerationType.IDENTITY 自增长
	@Column(name="userid")
	private Integer userid;
	
	@Column(name="username")
	private String username;
	
	@Column(name="userage")
	private Integer userage;
	
	@ManyToOne(cascade=CascadeType.PERSIST)
	@JoinColumn(name="roles_id")
	private Roles roles;	
}

```

```java
@Entity
@Table(name="t_roles")
public class Roles {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	
	@Column(name="rolename")
	private String rolename;
	
	@OneToMany(mappedBy="roles")
	private Set<Users> users = new HashSet<>();	
}
```

#### 一对多关联关系操作
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class OneToManyTest {

	@Autowired
	private UsersDao usersDao;
	
	
	/**
	 * 添加用户同时添加角色
	 */
	@Test
	public void test1(){
		//创建角色
		Roles roles = new Roles();
		roles.setRolename("管理员");
		//创建用户
		Users users =new Users();
		users.setUserage(30);
		users.setUsername("小王");
		//建立关系
		roles.getUsers().add(users);
		users.setRoles(roles);
		//保存数据
		this.usersDao.save(users);
	}
	
	/**
	 * 根据用户ID查询用户信息，同时查询角色
	 */
	@Test
	public void test2(){
		Users users = this.usersDao.findOne(14);
		System.out.println("用户姓名："+users.getUsername());
		Roles roles = users.getRoles();
		System.out.println(roles);
	}
}
```

### 多对多的关联关系
需求： 一个角色可以拥有多个菜单， 一个菜单可以分配多个角色。 多对多的关联关系
角色： 多方
菜单： 多方

#### 创建多对多关联关系
```java
@Entity
@Table(name="t_roles")
public class Roles {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="roleid")
	private Integer roleid;
	
	@Column(name="rolename")
	private String rolename;
	
	@ManyToMany(cascade=CascadeType.PERSIST,fetch=FetchType.EAGER)
	//@JoinTable:配置中间表信息
	//joinColumns:建立当前表在中间表中的外键字段
	@JoinTable(name="t_roles_menus",joinColumns=@JoinColumn(name="role_id"),inverseJoinColumns=@JoinColumn(name="menu_id"))
	private Set<Menus> menus = new HashSet<>();
}
```

```java
@Entity
@Table(name="t_menus")
public class Menus {

	@Id
	@GeneratedValue(strategy=GenerationType.IDENTITY)
	@Column(name="menusid")
	private Integer menusid;
	
	@Column(name="menusname")
	private String menusname;
	
	@Column(name="menusurl")
	private String menusurl;
	
	@Column(name="fatherid")
	private Integer fatherid;

	@ManyToMany(mappedBy="menus")
	private Set<Roles> roles = new HashSet<>();
}
```

#### 多对多关联关系操作
```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration("classpath:applicationContext.xml")
public class ManyToManyTest {
	
	
	@Autowired
	private RolesDao rolesDao;
	
	/**
	 * 添加角色同时添加菜单
	 */
	@Test
	public void test1(){
		//创建角色对象
		Roles roles = new Roles();
		roles.setRolename("超级管理员");
		
		//创建菜单对象    XXX管理平台 --->用户管理
		Menus menus = new Menus();
		menus.setMenusname("XXX管理平台");
		menus.setFatherid(-1);
		menus.setMenusurl(null);
		
		//用户管理菜单
		Menus menus1 = new Menus();
		menus1.setMenusname("用户管理");
		menus1.setFatherid(1);
		menus1.setMenusurl(null);
		
		
		//建立关系
		roles.getMenus().add(menus);
		roles.getMenus().add(menus1);
		
		menus.getRoles().add(roles);
		menus1.getRoles().add(roles);
		
		
		//保存数据
		this.rolesDao.save(roles);
	}
	
	/**
	 * 查询Roles
	 */
	@Test
	public void test2(){
		Roles roles = this.rolesDao.findOne(3);
		System.out.println("角色信息："+roles);
		Set<Menus> menus = roles.getMenus();
		for (Menus menus2 : menus) {
			System.out.println("菜单信息："+menus2);
		}
	}

}
```









```java

```