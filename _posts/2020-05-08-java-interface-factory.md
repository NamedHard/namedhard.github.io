---
layout: post
title: 浅谈Java接口和工厂模式的应用
date: 2020-05-08
Author: 来自Clarence
categories: 
tags: [Java, 工厂模式]
---

浅谈Java接口，工厂模式的理解

1.尽管学习和使用Java的时间不短，但是对于接口的理解一直不是很透彻，本次通过实例剖析接口设计的奥妙之处，从而加深理解

  业务背景：存在一个支付手段控制类PayController，若干支付手段类(AliPay,WeChatPay...)，控制类可以根据用户选择的支付方式，调用对应的支付类来进行支付操作。

​    V1版本实现：

```
// 支付控制类
public class PayController {
    public static void main(String[] args) throws Exception {
        String payMethod = PayController.promptForPay();
        switch (payMethod){
        case "AliPay":
            AliPay aliPay = new AliPay();
            aliPay.pay();
            break;
        case "WeChatPay":
            WeChatPay weChatPay = new WeChatPay();
            weChatPay.pay();
            break;
        default:
            throw new Exception();
        }
    }
    // 提示用户选择支付方式
    public static String promptForPay() {
        System.out.println("请选择支付方式：");
        Scanner scanner = new Scanner(System.in);
        return scanner.nextLine();
    }
}
```



    // AliPay
    public class AliPay {
        public void pay(){
            System.out.println("您已选择支付宝支付");
        }
    }
    
    // WeChatPay
    public class WeChatPay {
        public void pay(){
            System.out.println("您已选择微信支付");
        }
    }

从以上V1版本可以看到基本功能虽然已经实现，但是代码仍有很大改进空间；

1.对于具有相同性质行为[比如说pay()方法]的类对象来说，每次实例化都要调用一次当前对象的方法，略显罗嗦
比如说，假设项目扩张，用户支付手段更加多样化的时候，每增加一种支付手段类，就要新添加一行调用pay()方法的代码。

如果在实例化支付手段类之后，只需要一行调用pay()方法的代码，能否实现呢？答案是可以的，这时候引入接口的作用就体现出来了。

V2版本实现：引入接口

    // 支付手段接口
    public interface PayForU {
        void pay();
    }
    
    // 支付控制类
    public class PayController {
        public static void main(String[] args) throws Exception {
            String payMethod = PayController.promptForPay();
            PayForU payForU;
            switch (payMethod){
            case "AliPay":
                payForU = new AliPay();
                break;
            case "WeChatPay":
                payForU = new WeChatPay();
                break;
            default:
                throw new Exception();
            }
            payForU.pay();
        }
        // 提示用户选择支付方式
        public static String promptForPay() {
            System.out.println("请选择支付方式：");
            Scanner scanner = new Scanner(System.in);
            return scanner.nextLine();
        }
    }
    
    // AliPay
    public class AliPay implements PayForU {
        public void pay(){
            System.out.println("您已选择支付宝支付");
        }
    }
    
    // WeChatPay
    public class WeChatPay implements PayForU {
        public void pay(){
            System.out.println("您已选择微信支付");
        }
    }

通过引入支付手段接口后，不再需要编写重复性调用pay()方法的代码，使得代码更简洁了。
但是，对于代码的可维护性并没有显著提升，当支付方式增多时，仍然需要修改代码。
问题的根本在于实例化对象这一部分过于具体，应该进一步抽象，这时工厂模式的作用就显而易见。

V3版本的实现：引入工厂类

    // 工厂类
    public class BeanFactory {
        public static PayForU getInstance(String name) throws Exception {
            name = "com.demo.paymodel.v3." + name; // 包名+类名
            Class<?> clazz = Class.forName(name);
            PayForU obj = (PayForU) clazz.newInstance();
            return obj;
        }
    }
    
    // 支付控制类 
    public class PayController {
        public static void main(String[] args) throws Exception {
            String payMethod = PayController.promptForPay();
            PayForU payForU = BeanFactory.getInstance(payMethod);
            payForU.pay();
        }
    
        // 提示用户选择支付方式
        public static String promptForPay() {
            System.out.println("请选择支付方式：");
            Scanner scanner = new Scanner(System.in);
            return scanner.nextLine();
        }
    }

引入工厂类之后，支付控制类主方法就实现了对项目需求变更的灵活性选择了，在一定程度上也体现了OCP思想。
由此也进一步理解了开放封闭性原则(OCP)对于实现可维护性代码具有指导意义。