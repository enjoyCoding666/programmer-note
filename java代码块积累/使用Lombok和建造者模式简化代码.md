在项目开发中，我们经常需要构建对象。常见的做法有getter/setter，或者构造器构建对象。
可能会有人写出类似如下的代码：
```
Company company=new Company();
company.setAgentId(agentId);
company.setAgentUserId(agentUserId);
company.setCompanyName( companyName );
company.setAgentUserName(agentUserName);
company.setDomain( domain );
company.setTaxNo( taxNo );
company.setCreateTime( new Date() );
company.setIsauth(1);
company.setActivationCode(activationCode);
company.setAuthCode(authCode);
company.setDomain(domain);
company.setUseType(1);
company.setContactor(phoneNumber);
```
也可能写了一个参数非常冗长，传参特别容易出错的构造方法。
### Lombok注解
我们可以使用Lombok和建造者模式简化代码。
(注意，Lombok除了要加入依赖包，还需要配置一下。几分钟可以搞定，具体做法参考以下。)
https://blog.csdn.net/Nicholas_GUB/article/details/120929648
如果你的同事不喜欢用Lombok，或者你想了解一下Lombok注解所表示的代码块，你可以在安装插件后，通过选择IDEA导航栏的"Refactor"---"Delombok"将Lombok注解逆向生成代码。
首先，在Company类上方加入Lombok注解，如下所示：
```
@Data
@Builder
@AllArgsConstructor
@NoArgsConstructor
@RequiredArgsConstructor(staticName = "getCompany")
public class Company {
    private Integer agentUserId;
    @NonNull
    private Integer agentId;
    private String companyName;
    private String agentUserName;
    private String domain;
    private String taxNo;
    private Date createTime;
    private Integer isAuth;
    private String activationCode;
    private String authCode;
    private Integer userType;
    private String phoneNumber;
}
```
其中的注释意思如下：
```
@Getter：表示各个属性的get()方法。
@Setter：表示各个属性的set()方法。
@Builder：可通过Builder模式构建对象。
@NonNull：变量不能为空
@ToString： 表示toString()方法。
@EqualsAndHashCode：表示equals()、hashcode()方法。
@ToString(exclude = {"authCode"})：表示toString()方法中，不计入authCode这个属性变量
@EqualsAndHashCode(of = {"authCode"})：表示equals和计算hasCode时，不计入authCode这个属性变量。
@EqualsAndHashCode(callSuper = false)：表示equals和计算hasCode时，不会比较其继承的父类的属性，可能会导致错误判断，谨慎使用 callSuper = false；
@EqualsAndHashCode(callSuper = true)：表示equals和计算hasCode时，会比较其继承的父类的属性；
@Data：包含了getter、setter、toString、equals、hashcode方法。
@NoArgsConstructor ： 生成一个无参数的构造方法。
@AllArgsContructor： 会生成一个包含所有变量的构造方法。
@RequiredArgsConstructor： 会生成一个包含常量，和标识了NotNull的变量 的构造方法。
@RequiredArgsConstructor(staticName = "getCompany")：生成的构造方法是private，外部可以使用static方法访问。
```
其他的：
```
* @Accessors(chain = true) ，该注解设置为chain=true，可以用链式访问，生成setter方法返回this（也就是返回的是对象），代替了默认的返回void。
 //开起chain=true后可以使用链式的set
 Person person=new Person().setAge(27).setName("kevin");
* @EqualsAndHashCode 标在子类上。。
  1. callSuper = true，根据子类自身的字段值和从父类继承的字段值 来生成hashcode，当两个子类对象比较时，只有子类对象的本身的字段值和继承父类的字段值都相同，equals方法的返回值是true。
  2. callSuper = false，根据子类自身的字段值 来生成hashcode， 当两个子类对象比较时，只有子类对象的本身的字段值相同，父类字段值可以不同，equals方法的返回值是true。

```
如果不理解，我们可以选择IDEA导航栏的"Refactor"---"Delombok"逆向生成代码。
只加注解@RequiredArgsConstructor表示的代码如下，由于只有变量agentId为@NonNull，所以构造方法只有这个变量：
```
    public Company(Integer agentId) {
        this.agentId = agentId;
    }
```
使用注解@RequiredArgsConstructor(staticName = "getCompany")表示的代码如下：
```
    private Company() {
    }

    public static Company getCompany() {
        return new Company();
    }
```
### Builder模式构建对象
加入了@Builder后，那么可以将代码改写成如下：
```
   Company company=Company.builder().agentId(1).agentUserId(1).companyName("google")
                                  .agentUserName("lin").domain("test").taxNo("1111111")
                                  .createTime(new Date()).isAuth(1).activationCode("0587-1235")
                                  .userType(1).phoneNumber("666666666")
                                 .build();
```
基本形式就是： 类名.builder().build() ，在中间加入变量方法及变量的具体值。
通过这种方式构建对象，没有那么多的setter，参数也不容易出错。

参考资料：
http://kriszhang.com/lombok/