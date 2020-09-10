## 泛化(Generalization)
泛化关系表示 类与类之间的继承关系,接口与接口直接的继承关系,类与接口之间的实现关系


## 继承关系 (Inheritance)
动物都有眼睛,都需要呼吸,但是老虎自己会奔跑

![Inheritance](https://raw.githubusercontent.com/neoshero/CloudNotes/master/Images/UML/Inheritance.png)
```C#
public abstract class Animal 
{
    public int Eyes {get;set;}
    public virtual void Breath(){}
}

public class Tiger:Animal
{
    public void Run() {}
}
```

## 实现关系 (Implementation)
一个类实现了某个接口的功能

![Implementation](https://raw.githubusercontent.com/neoshero/CloudNotes/master/Images/UML/Implementation.png)
```C#
public interface IPower
{
    void Swiming();
}

public class Shark:IPower
{
    public void Swiming(){}
}

public class Dolphin:IPower
{
     public void Swiming(){}
}

```

## 依赖关系(Dependency)
一个方法需要依赖另一个对象来完成整个方法的功能,则他们属于依赖关系(树木需要土壤和水源)

![Dependency](https://raw.githubusercontent.com/neoshero/CloudNotes/master/Images/UML/Dependency.png)
```C#
public class Tree
{
    public Tree(Water water,Soil soil)
    {

    }
}

public class Water{}
public class Soil{}
```

## 关联关系(Association)
当一个对象创建时,另一个对象也要随之创建出来,那么他们属于关联关系(学生存在时,学生应该拥有自己的课程)

![Association](https://raw.githubusercontent.com/neoshero/CloudNotes/master/Images/UML/Association.png)
```C#
public class Teacher
{
    private List<Student> Students {get;private set;}
}

public class Student
{
    private List<Theacher> Teachers {get;private set;}
    private List<Course> Courses {get;private set;}
}

public class Course 
{
    private Student Student {get;private set;}
}
```

## 聚合关系(Aggregation)
聚合关系也是关联关系的一种,他们的关系可以组合在一起,也可以单独存在

![Aggregation](https://raw.githubusercontent.com/neoshero/CloudNotes/master/Images/UML/Aggregation.png)
```C#
public class Mouse｛｝
public class Keyboard {}

public class Computer
{
    public Mouse Mouse {get;set;}
    public Keyboard Keyboard {get;set;}
}


```

## 组合关系(Composition)
他们之间是强的关联关系,彼此之间密不可分,没有公司就没有部门,没有部分就没有公司

![Composition](https://raw.githubusercontent.com/neoshero/CloudNotes/master/Images/UML/Composition.png)
```C#
public class Company
{
    public List<Department> Departments {get;set;}
}

public class Department
{
    public Company company {get;set;}
}
```