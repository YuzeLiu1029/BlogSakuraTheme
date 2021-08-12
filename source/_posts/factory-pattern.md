---
title: factory-pattern
author: Liu Yuze
avatar: 'https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/custom/avatar.jpg'
authorLink: liuyuze.site
authorAbout: 好少年光芒万丈
authorDesc: 好少年光芒万丈
categories: 技术
comments: true
mathjax: true
date: 2020-10-23 22:47:31
tags:
    - java
    - Design Pattern
keywords: design pattern
description:
photos: https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/blogPic/123829.jpg
---
# 工厂类-从订购一个Pizza开始
## 简单工厂类
作为一个披萨店的老板，你可能需要这样一段代码来完成Pizza的订购：
```java
public class PizzaStore {
    public Pizza orderPizza() {
        Pizza pizza = new Pizza();
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```
如果你需要更多的不同类型的Pizza，可能会出现如下代码：
```java
public Pizza orderPizza(String types) {
    Pizza pizza;
    
    if(types.equals("cheese"){
        pizza = new CheesePizza();
    } else if(types.equals("greek")){
        pizza = new GreekPizza();
    } else if(types.equals("pepperoni")){
        pizza = new PepperoniPizza();
    }
    
    pizza.prepare();
    pizza.bake();
    pizza.cut();
    pizza.box();
    return pizza;
}
```
当Pizza的种类变得复杂，新的种类不断增多，而有些过时的种类不再受欢迎的时候，这段代码会被反复拿出来做修改，增加或删除一些种类来适应不断变化的菜单，但这其实违背了开放封闭原则。
开放封闭原则：当软件需要变化时，应尽量通过扩展的方式来实现变化，而不是通过修改的方式来实现。如果实例化某些具体类的时候，可能会使orderPizza这段代码出问题，因此说orderPizza这段代码没有对修改关闭。
既然已经知道了哪些代码会改变，哪些不会，那么我们可以采用refactor中的一个技巧封装来对这段代码进行改变。
先把```if...else```这段代码从```orderPizza```这个函数中抽离。
```java
public class SimplePizzaFactory {
    public Pizza createPizza(String type) {
        Pizza pizza = null;
        if(type.equals("cheese")) {
            pizza = new CheesePizza();
        }else if(types.equals("greek")){
            pizza = new GreekPizza();
        } else if(types.equals("pepperoni")){
            pizza = new PepperoniPizza();
        }
        return pizza;
    }
}
```
这样```orederPizza```这一段代码就变成了：
```java
public class PizzaStore {
    SimplePizzaFactory factory;
    public PizzaStore(SimplePizzaFactory factory) {
        this.factory = factory;
    }
    public Pizza orderPizza(String type) {
        Pizza pizza;
        pizza = factory.createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
}
```
这就是一个简单工厂类。整个过程其实是一个随着需求变化的重构的过程，涉及到了基本设计原则中的开放封闭原则，同时涉及到了重构中的一个技巧，封装函数。
很多地方并不把简单工厂模式看成是一个设计模式，而更像是一种变成习惯，但经常被使用。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/blogpic/simpleFact0.png)
## 工厂方法类
现在考虑更进一步的情况，这家PizzaStore的开业获得了极大的成功，各个地区开始有连锁PizzaStore加盟，但每一家加盟店因为区域的不同从而导致Pizza风味的地区化差异。

那么我们可能会有下面这种代码出现：
```java
NYPizzaFactory nyFactory = new NYPizzaFactory();
PizzaStore nyStore = new PizzaStore(nyFactory);
nyStore.orderPizza("veggie");

ChicagoPizzaFactory chicagoFactory = new ChicagoFactory();
PizzaStore chicgoStore = new PizzaStore(chicagoFactory);
chicagoStore.orderPizza("veggie");
```
这是一种实现的方法。
现在你想在PizzaStore中建立一个框架，把其他加盟店和创建Pizza的功能捆绑在一起的同时又保持一定弹性。之前我们把createPizza放在PizzeStore类中，这样达到了捆绑的效果可是丧失了弹性。
其实这段话翻译过来就是，我们不想把createPizza放到新建的一个工厂类里面，而是想放回这个PizzaStore类，但是我们还想保持不同地区的加盟店能够作出不同地区特色的pizza。
首先我们把createPizza()放回到PizzaStore中，但是把它设置为抽象方法。
```java
public abstract class PizzaStore {
    public Pizza orderPizza(String type) {
        Pizza pizza;
        pizza = createPizza(type);
        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();
        return pizza;
    }
    
    abstract Pizza createPizza(String type);
}
```
*抽象类自己是不能实例化对象的，它必须要被继承才能被使用。通常在设计阶段就要确定是否要使用抽象类。
抽象类和接口类的区别：
在语法层面上 ：抽象类可以提供成员方法的实现细节，而接口中只能存在public abstaract方法；抽象类中的成员变量可以是各种类型，接口类只能是public static final；接口类中不能含有静态代码块和静态方法，抽象类中可以有；一个类只能继承一个抽象类但是可以实现多个接口类；
设计层面上：抽象类是对一种事物的抽象而接口类是对行为的抽象；抽象类作为很多子类的父类是一种模版式设计而接口是一种行为规范。*
所以现在我们对拥有加盟店后的整体设计变成：有一个PizzaStore作为父类，每个子类都继承这个父类，但每个子类可以自行决定如何制造这个特色Pizza。这里用到了一个重构的技巧，叫做**提炼继承类(Extract Hierarchy)**。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/blogpic/factMethod01.png)
站在PizzaStore的orderPizza()的角度来看，此抽象方法是在PizzaStore内部定义的，但是是交由子类来具体实现的，orderPizza()对Pizza这个类做了很多事情，但是Pizza对象是抽象的，orderPizza()是并不知道实际参与的具体类是哪一个，这就是解耦(decouple)。
所以现在我们新开一个纽约加盟店中```createPizza```的代码是这样的：
```java
public class NYPizzaStore extends PizzaStore {
    Pizza createPizza (String type) {
        if (type.equals("cheese")) {
            return new NYStyleCheesePizza();
        } else if (type.equals("veggie")) {
            return new NYStyleVeggiePizza();
        } else if (type.equals("clam")) {
            return new NYStyleClamPizza();
        } else if (type.equals("pepperoni")) {
            return new NYStylePepperoniPizza();
        } else {
            return null;
        }
    }
}
```
同理，我们除了关注于PizzaStore之外，还更应关注Pizza本身,我们再丰富一下不同种类的pizza让整个例子看上去更加真实：
```java
public abstract class Pizza {
    String name;
    String dough;
    String sauce;
    ArrayList toppings = new ArrayList();
    void prepare() {
        System.out.println(“Preparing “ + name);
        System.out.println(“Tossing dough...”);
        System.out.println(“Adding sauce...”);
        System.out.println(“Adding toppings: “);
        for (int i = 0; i < toppings.size(); i++) {
            System.out.println(“ “ + toppings.get(i));
        }
    }
    
    void bake() {
        System.out.println(“Bake for 25 minutes at 350”);
    }
    void cut() {
        System.out.println(“Cutting the pizza into diagonal slices”);
    }
    
    void box() {
        System.out.println(“Place pizza in official PizzaStore box”);
    }
    public String getName() {
        return name;
    }
}
```
```java
public class NYStyleCheesePizza extends Pizza {
    public NYStyleCheesePizza() {
        name = “NY Style Sauce and Cheese Pizza”;
        dough = “Thin Crust Dough”;
        sauce = “Marinara Sauce”;
        toppings.add(“Grated Reggiano Cheese”);
    }
}

public class ChicagoStyleCheesePizza extends Pizza {
    public ChicagoStyleCheesePizza() {
        name = “Chicago Style Deep Dish Cheese Pizza”;
        dough = “Extra Thick Crust Dough”;
        sauce = “Plum Tomato Sauce”;
        toppings.add(“Shredded Mozzarella Cheese”);
    }
    void cut() {
        System.out.println(“Cutting the pizza into square slices”);
    }
}
```
现在我们有了加盟店，有了具体的Pizza，我们order一个pizza的过程就从之前的有一个工厂类变成：
```java
NYPizzaFactory nyFactory = new NYPizzaFactory();
PizzaStore nyStore = new PizzaStore(nyFactory);
nyStore.orderPizza("veggie");

ChicagoPizzaFactory chicagoFactory = new ChicagoFactory();
PizzaStore chicgoStore = new PizzaStore(chicagoFactory);
chicagoStore.orderPizza("veggie");
```
```java
PizzaStore nyStore = new NYPizzaStore();
Pizza pizza = nyStore.orderPizza(“cheese”);

PizzaStore chicagoStore = new ChicagoPizzaStore();
pizza = chicagoStore.orderPizza(“cheese”);
```
这个例子里，```PizzaStore```可以被视作创建者(Creator)类，```Pizza```本身是产品类。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/blogpic/factMethod2.png)
createPizza()作为工厂方法出现，orderPizza()和这个工厂方法联合起来共同构成了整个框架。
工厂方法模式的定义就是 ： 工厂方法模式中定义了构建对象的一个接口，但是具体如何实现，实现哪一个对象，是由子类自行控制的。工厂方法让类把实例化推迟到了子类。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/blogpic/factMethod3.png)
工厂方法模式中蕴涵了六大原则中之一的 ： 依赖倒置原则。依赖倒置原则的核心思想就是依赖抽象而不要依赖具体类。
## 抽象工厂类
现在再将整个设计的需求复杂化，除了需要考虑到加盟店和不同加盟店因地区而不同的风味变化，我们还要考虑不同加盟店使用的原材料在我们的控制之中。现在我们只保证了每一家加盟店的制作流程在我们框架中。
现在我们先来构建一个原料工厂的模型，这个原料工厂需要涵盖所有制作Pizza的所有原材料的构建,这次我们仅通过不同原料工厂准备的原料不同来区分不同地区的Pizza。
```java
public interface PizzaIngredientFactory {
    public Dough createDough();
    public Sauce createSauce();
    public Cheese createCheese();
    public Veggies[] createVeggies();
    public Pepperoni createPepperoni();
    public Clams createClam();
}
```
```java
public class NYPizzaIngredientFactory implements PizzaIngredientFactory {
    public Dough createDough() {
        return new ThinCrustDough();
    }
    public Sauce createSauce() {
        return new MarinaraSauce();
    }
    public Cheese createCheese() {
        return new ReggianoCheese();
    }
    public Veggies[] createVeggies() {
        Veggies veggies[] = { new Garlic(), new Onion(), new Mushroom(), new RedPepper() };
        return veggies;
    }
    public Pepperoni createPepperoni() {
        return new SlicedPepperoni();
    }
    public Clams createClam() {
        return new FreshClams();
    }
}
```
接下来重写Pizza这个父类中prepare的方法，我们可以参考之前工厂方法的模式将这个prepare方法写成抽象方法：
```java
public abstract class Pizza {
    String name;
    Dough dough;
    Sauce sauce;
    Veggies veggies[];
    Cheese cheese;
    Pepperoni pepperoni;
    Clams clam;
    abstract void prepare();
    void bake() {
        System.out.println(“Bake for 25 minutes at 350”);
    }
    void cut() {
        System.out.println(“Cutting the pizza into diagonal slices”);
    }
    void box() {
        System.out.println(“Place pizza in official PizzaStore box”);
    }
    void setName(String name) {
        this.name = name;
    }
    String getName() {
        return name;
    }
}
```
```java
public class CheesePizza extends Pizza {
    PizzaIngredientFactory ingredientFactory;
    public CheesePizza(PizzaIngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }
    void prepare() {
        System.out.println(“Preparing “ + name);
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
        cheese = ingredientFactory.createCheese();
    }
}
```
我们来对比一下纽约的CheesePizza：
```java
public class NYStyleCheesePizza extends Pizza {
    public NYStyleCheesePizza() {
        name = “NY Style Sauce and Cheese Pizza”;
        dough = “Thin Crust Dough”;
        sauce = “Marinara Sauce”;
        toppings.add(“Grated Reggiano Cheese”);
    }
}
```
通过上面纽约原材料工厂，我们可以制造出纽约风味的NYStyleCheesePizza；
Pizza类不关心具体每一种原材料是什么，这个仅由我们使用哪一个工厂来决定，pizza类仅仅需要知道做pizza的各个步骤。
```java
public class ClamPizza extends Pizza {
    PizzaIngredientFactory ingredientFactory;
    public ClamPizza(PizzaIngredientFactory ingredientFactory) {
        this.ingredientFactory = ingredientFactory;
    }
    void prepare() {
        System.out.println(“Preparing “ + name);
        dough = ingredientFactory.createDough();
        sauce = ingredientFactory.createSauce();
        cheese = ingredientFactory.createCheese();
        clam = ingredientFactory.createClam();
    }
}
```
而现在，以及纽约的加盟店制作Pizza的代码编程：
```java
public class NYPizzaStore extends PizzaStore {
    protected Pizza createPizza(String item) {
        Pizza pizza = null;
        PizzaIngredientFactory ingredientFactory = new NYPizzaIngredientFactory();
        if (item.equals(“cheese”)) {
            pizza = new CheesePizza(ingredientFactory);
            pizza.setName(“New York Style Cheese Pizza”);
        } else if (item.equals(“veggie”)) {
            pizza = new VeggiePizza(ingredientFactory);
            pizza.setName(“New York Style Veggie Pizza”);
        } else if (item.equals(“clam”)) {
            pizza = new ClamPizza(ingredientFactory);
            pizza.setName(“New York Style Clam Pizza”);
        } else if (item.equals(“pepperoni”)) {
            pizza = new PepperoniPizza(ingredientFactory);
            pizza.setName(“New York Style Pepperoni Pizza”);
        }
        return pizza;
    }
}
```
我们来对比一下之前纽约加盟店做Pizza的过程：
```java
public class NYPizzaStore extends PizzaStore {
    Pizza createPizza (String type) {
        if (type.equals("cheese")) {
            return new NYStyleCheesePizza();
        } else if (type.equals("veggie")) {
            return new NYStyleVeggiePizza();
        } else if (type.equals("clam")) {
            return new NYStyleClamPizza();
        } else if (type.equals("pepperoni")) {
            return new NYStylePepperoniPizza();
        } else {
            return null;
        }
    }
}
```
总结一下这次我们又做了啥：
我们建立了一个抽象的类：原料工厂类，这就是我们这个例子中的抽象工厂。这个抽象工厂定义了一系列的接口。通过抽象工厂所提供的接口，我们可以创建出一个产品的家族，通过不同的组合和实现方式创造出不同的产品。
最后我们再来看一下order一个NYStyleCheesePizza需要几步：
```java
PizzaStore nyPizzaStore = new NYPizzaStore();
nyPizzaStore.orderPizza("cheese");
```
在```orderPizza```中，我们调用了：
```java
Pizza pizza = createPizza("cheese");
```
在```createPizza```中，我们建立了纽约原料工厂，并用它来制作CheesePizza：
```java
Pizza pizza = new CheesePizza(nyIngredientFactory);
```
这样在Pizza的prepare()过程中，我们会使用nyIngredientFactory来创建各种原材料：```createDough()```, ```createSauce()```, ```createCheese()```, ```createClam()```。
最后我们把Pizza ```bake()```，```cut()```， ```box()```然后return；
这个设计模式就是抽象工厂模式，我们利用抽象工厂创建了一个产品家族。抽象工厂允许客户使用抽象的接口来创建一组相关的产品，而不需要关心实际产出的具体产品是什么。
产出的具体产品是什么。
![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/blogpic/abstractFactory01.png)

![](https://cdn.jsdelivr.net/gh/YuzeLiu1029/cdn/img/blogpic/abstractFactory2.png)
其实还有一个有意思的发现，抽象工厂中每一个```create method```都是一个工厂方法设计模式。确实如此，通常抽象工厂中的方法其实就是以工厂方法实现的，这其实也是很自然的事情，抽象工厂是用来创建一组产品的接口，这个接口内的每一个方法其实都负责创建一个具体的产品，而我们是通过抽象工厂的子类来实现创建这些具体产品。

## reference
***Head First Design Patterns 2004 by Eric Freeman, Elisabeth Freeman, Kathy Sierra, and Bert Bates                 ***                           