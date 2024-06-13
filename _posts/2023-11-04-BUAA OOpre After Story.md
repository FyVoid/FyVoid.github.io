---
layout: post
title: "BUAA OOpre After Story"
category: BUAA
tags: OOP
---

## 写在最前

OOpre本身的体验是很好的，写代码的过程也很有趣，在这个学期的课程中体验可以算是靠前的，不过仍然有一些我认为可以改进的地方

总体来说，OOpre是不错的java入门课程和相对复杂程序设计的入门课程

## 架构设计

抛开非迭代的两次作业不谈，作业的宗旨是开发一个模拟性的程序，根据课程顺序主要包括以下知识点

* Git和现代IDE的使用（不知道为什么很多同学都习惯使用devcpp这种东西）
* java基础：变量，运算，循环，函数和基础IO
* JUnit单元测试
* 容器
* 正则表达式
* 面向对象：继承
* 接口和多态
* 设计模式

我的代码架构为：

```
./src
├── Main.java
├── adventurer
│   ├── Adventurer.java
│   └── Backpack.java
├── bottle
│   ├── Bottle.java
│   ├── RecoverBottle.java
│   ├── RegularBottle.java
│   └── ReinforcedBottle.java
├── commodity
│   ├── Commodity.java
│   └── Store.java
├── equipment
│   ├── CritEquipment.java
│   ├── EpicEquipment.java
│   ├── Equipment.java
│   └── RegularEquipment.java
├── food
│   └── Food.java
└── log
    ├── AoeLog.java
    ├── AttackLog.java
    ├── BattleLog.java
    ├── LogItem.java
    └── PotionLog.java

7 directories, 19 files

```

代码架构图：![架构.drawio](/Assets/架构.drawio.png)

显然，这个架构是合理的但是是不够完美的。

### 完美的架构

#### 更合理的继承关系

Bottle、Equipment和Food三个类中有大量类似的方法和属性，例如

```
removeBottle(int id);
removeEquipment(int id);
removeFood(int id);
```

这样的方法大可以通过让这三个类同时继承一个抽象类Item，并提供方法

```
removeItem(int id);
```

来统一到同一个接口，这样的话，例如Main中处理指令的方法：

```
    public static void carryCommand(Adventurer adv, int carryId, int commandType) {
        switch (commandType) {
            case 9:
                adv.tryCarryEquipment(carryId);
                break;

            case 10:
                adv.tryCarryBottle(carryId);
                break;

            case 11:
                adv.tryCarryFood(carryId);
                break;

            default:
                break;
        }
    }
```

完全可以通过调用同一个方法来解决

```
    public static void carryCommand(Adventurer adv, int carryId) {
        adv.tryCarryItem(carryId);
    }
```

> 一开始没有这样做是出于防止同一个id被赋予不同的物品的考虑，由于在Adventurer中三种物品被(id, Item)的键值对保存于HashMap，id产生冲突会很麻烦，不过似乎id并不会冲突

事实上，通过在抽象类中添加一些方法，我们可以很容易的将所有的Item以(hash, Item)的键值对保存于同一个hashmap，同时做到对id，name的快速查找，例如提供一个方法

```
getHash() {
		return name + String.valueOf(id);
}
```

#### 在Main外部管理输入输出和指令解析

应该不少同学都是这样做的，我在Main里完成输入输出管理和解析纯粹是一开始顺手写了后面懒得重构

#### 替代switch，更好看的指令调用

到了开发中期，我们会面临指令解析的类中一定有一个函数会带有很长的switch语句，导致checkstyle很难控制在100行以内，事实上，有一个简明且可读性佳的方法可以解决这个问题——闭包（lambda表达式）

对于除了1以外的每一个指令，我们可以定义一个函数

```
command_i(Adventurer adv, String[] command);
```

来处理这个指令，而通常，我们会采取对指令编号switch的方法来调用每个指令的执行方法，例如

```
switch(commandType) {
		case 2: case 4: case 6:
				...
				break;
		...
}
```

这样的结构事实上可以通过一个形如

```
interface Command {
        void executeCommand(Adventurer adv, String[] command);
    }
private static HashMap<Integer, Command> commandHashMap = new HashMap<>();
```

的HashMap来替代，在将处理指令的函数通过lambda表达式存入HashMap后，上文的switch语句可以被以下语句简单替代：
```
commandHashMap.get(commandType).executeCommand(adv, command);
```

具体细节有兴趣的同学可以自行学习java的lambda表达式

### 一些经验

在我的架构中，有一些部分我认为是值得讨论一下的，在此做一点简单的介绍

#### 抽象类

在我的架构中，战斗记录的处理依赖于对LogItem子类的解析，LogItem本身是一个抽象类（只能被继承，不能被实例化的类），在LogItem中定义了存储log表达式，存储相关的解析正则表达式和获取log的方法，解析的过程完全置放在LogItem类的相关方法中，存储log的逻辑置放在静态工具类BattleLog中。

事实上，还有更清晰的方法：定义一个接口Logable，其中定义Log必须要实现的方法（存储表达式，存储正则，解析表达式，返回log），LogItem类继承Logable，再作为抽象基类

#### 删除

当删除Adventurer的东西时，需要同时删除本身和backpack中的引用

如果这个过程全部都手动操作容器，很容易造成遗漏，这一点在最后一次作业中最容易发生问题

应该在Adventurer中实现remove方法，同时调用backpack中的remove方法，在任何删除的情况下都直接调用方法，而不要手动操作容器

## JUnit Test

* 课程组本意是好的，本身单元测试也是重要的
* 但是由于缺乏自动生成的方法，加上写单元测试也是很花时间的过程，实际上同学们肯定都是随便写完成任务要求的
* 由于在标准流进行io，在OOpre课程中单元测试发现问题的概率远不如搭建自动评测机来的容易
* 希望不要要求Main类中的单元测试覆盖率，意义很小很敷衍

希望OOpre未来能找到更好的使用单元测试的方法，而不是单纯要求数据达标

## 心得

其实在做OOpre之前自己就组织过一些上千行的项目，所以整个开发还算是很顺利，而且很享受

> 用Cpp做大型项目非常有助于强调架构设计的重要性和锻炼同学的心态

如果要我说的话，Java作为编程语言远远算不上是好用的，作为面向对象的入门语言也不一定是最好的，仅仅能算作是最常用的语言，在面向对象上我认为C#是更适合学习的语言，目前对OOpre课程中Java语法产生的意见有：

* jdk版本不够高不支持自动类型推断（很不方便很不现代）
* java不支持默认参数（我不理解）
* java没有set，get，没有像swift和C#一样方便的结构体
* 用.equals()判断字符串相等太过反人类

OOpre课程的迭代顺序整体来说是不错的，不过或许有一部分应该改进：

* 第六次开发学习继承和接口，然而这个时候大部分代码都定型了，最后几乎一定会导致对以前代码的重构，这一次作业的设计我认为是不合理的
* 我认为合理的顺序是
  * git的使用和java语法入门
  * 对象基础，属性和方法
  * 继承
  * 容器
  * 接口和多态

## 建议

* 稍微调整迭代顺序，把继承放到更早的位置，有助于同学进行架构设计
* 升级jdk版本，希望下个学年的同学们能用上自动类型推断
* 能不能不用csdn.........................................


​	