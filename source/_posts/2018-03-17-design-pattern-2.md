---
title: 设计模式二
date: 2018-03-17 21:39:54
tags: 设计
categories: 后端
comments: true
---
设计模式作为软件设计的经验总结，有三大原则需要遵守：

- 类对象职责单一，即在封装数据时尽量使类的功能单一，各类之间通过消息或者接口组合在一起
- 开放-闭合原则，即对扩展的开放，对修改的闭合
- 依赖倒转原则，对象抽象时应该依赖于抽象，而不应该依赖于细节，即面向接口编程
<!--more-->

### 建造者模式
建造者模式是将多个不同的对象组合成负责复杂对象的一种设计模式，而各个子对象可以独立抽象和发展而互不影响，通过建造者可以组合成不同的复杂对象，灵活多变。
同时，可以构建多层的建造者对象来得到更加复杂的类对象
建造者模式是设计中比较常用的设计方法，我们在程序设计中处理的事物总是零散，通过分析抽象可将零散的事物进行抽象分类，在使用中就可以直接使用
下面通过吃火锅点单系统来说明模式的使用方式，火锅订单包括：

- 锅底，麻辣/清汤
- 蘸料碟，芝麻/花生
- 餐具，筷子/碗
- 荤菜，牛肉/鱼肉/鸡肉
- 素菜，白菜/土豆/玉米
- 盛菜盘子，陶瓷盘/玻璃盘

下面来进行构造：

    class ProductItem {
        int price = 0;
        ProductItem(int price) { price = price; }
        ProductItem() {}
        ~ProductItem() {price = 0;}
        virtual void display() = 0;
        int getPrice() {
            return price;
        }
    }
    //锅底
    //麻辣锅底
    class SpicyHotPotSoup: public ProductItem {
        SpicyHotPotSoup(int price): ProductItem(price) {}
        ~SpicyHotPotSoup() {}
        void display() {
            cout << "名称：麻辣锅底" << " 价格：" << price << " ";
        }
    }
    //清汤锅底
    class LightHotPotSoup: public ProductItem {
        LightHotPotSoup(int price): ProductItem(price) {}
        ~LightHotPotSoup() {}
        void display() {
            cout << "名称：清汤锅底" << " 价格：" << price << " ";
        }
    }

    //蘸料
    //芝麻蘸料
    class SesameSauce: public ProductItem {
        SesameSauce(int price): ProductItem(price) {}
        ~SesameSauce() {}
        void display() {
            cout << "名称：芝麻蘸料" << " 价格：" << price << " ";
        }
    }
    //花生蘸料
    class PeanutSauce: public ProductItem  {
        PeanutSauce(int price): ProductItem(price) {}
        ~PeanutSauce() {}
        void display() {
            cout << "名称：花生蘸料" << " 价格：" << price << " ";
        }
    }

    //餐具
    //碗
    class Bowl: public ProductItem {
        Bowl() {}
        ~Bowl() {}
        void display() {
            cout << "名称：碗" << " 价格：" << price << " ";
        }
    }
    //筷子
    class Chopsticks: public ProductItem {
        Chopsticks() {}
        ~Chopsticks() {}
        void display() {
            cout << "名称：筷子" << " 价格：" << price << " ";
        }
    }

    //盛菜盘子
    class Plate {
        Plate() {}
        ~Plate() {}
    }
    //陶瓷盘
    class CermicPlate: public Plate {
        CermicPlate() {}
        ~CermicPlate() {}
    }
    //玻璃盘
    class GlassPlate: public Plate {
        GlassPlate() {}
        ~GlassPlate() {}
    }

    //荤菜
    class Meat: public ProductItem {
        Plate m_plate;
        Meat(Plate plate,int price): ProductItem(price) {
            m_plate = new CermicPlate();
        }
        ~Meat() {m_plate = null;}
    }
    //牛肉
    class Beef: public Meat {
        Beef(int price): Meat(price) {
        }
        ~Beef() {}
        void display() {
            cout << "名称：牛肉" << " 价格：" << price << " ";
        }
    }
    //鱼肉
    class Fish: public Meat {
        Fish(int price): Meat(price) {}
        ~Fish() {}
        void display() {
            cout << "名称：鱼肉" << " 价格：" << price << " ";
        }
    }
    //鸡肉
    class Chicken: public Meat {
        Chicken(int price): Meat(price) {}
        ~Chicken() {}
        void display() {
            cout << "名称：鸡肉" << " 价格：" << price << " ";
        }
    }

    //素菜
    class Vagetble: public ProductItem {
        Plate m_plate;
        Vagetble(int price): ProductItem(price) {
            m_plate = new GlassPlate();
        }
        ~Vagetble() {m_plate = null;}
    }
    //白菜
    class Cabbage: public Vagetble {
        Cabbage(int price): Vagetble(price) {}
        ~Cabbage() {}
        void display() {
            cout << "名称：白菜" << " 价格：" << price << " ";
        }
    }
    //土豆
    class Potato: public Vagetble {
        Potato(int price): Vagetble(price) {}
        ~Potato() {}
        void display() {
            cout << "名称：土豆" << " 价格：" << price << " ";
        }
    }
     //玉米
    class Maize: public Vagetble {
        Maize(int price): Vagetble(price) {}
        ~Maize() {}
        void display() {
            cout << "名称：玉米" << " 价格：" << price << " ";
        }
    }

    class Order {
        ProductItem *m_product;
        int num;

        Order(ProductItem *pitem,int num) {
            m_productList = pitem;
            num = num;
        }
        ~Order() {
            m_productList = null;
            num = 0;
        }

        int getOrderPrice() {
            return m_productList->getPrice() * num;
        }

        void display() {
            m_product->display();
            cout << " 数量：" << num << endl;
        }
    }

    class OrderMeal {
        Order orders[] = [];
        OrderMeal() {}
        ~OrderMeal() {}

        void addOrder(string type,int price,int num) {
            switch(type) {
                case "SpicyHotPotSoup": {
                    orders.push(new SpicyHotPotSoup(price),num);
                    break;
                }
                case "LightHotPotSoup": {
                    orders.push(new LightHotPotSoup(price),num);
                    break;
                }
                case "SesameSauce": {
                    orders.push(new SesameSauce(price),num);
                    break;
                }
                case "PeanutSauce": {
                    orders.push(new PeanutSauce(price),num);
                    break;
                }
                case "Bowl": {
                    orders.push(new Bowl(price),num);
                    break;
                }
                case "Chopsticks": {
                    orders.push(new Chopsticks(price),num);
                    break;
                }
                case "Beef": {
                    orders.push(new Beef(price),num);
                    break;
                }
                case "Fish": {
                    orders.push(new Fish(price),num);
                    break;
                }
                case "Chicken": {
                    orders.push(new Chicken(price),num);
                    break;
                }
                case "Cabbage": {
                    orders.push(new Cabbage(price),num);
                    break;
                }
                case "Potato": {
                    orders.push(new Potato(price),num);
                    break;
                }
                case "Maize": {
                    orders.push(new Maize(price),num);
                    break;
                }
            }
        }

        int getTotalBill() {
            int total = 0;
            for(int index = 0;index < orders.length;index++) {
                total += orders[index].getOrderPrice();
            }
            return total;
        }

        void dump() {
            for(int index = 0;index < orders.length;index++) {
                total += orders[index].display();
            }
            cout << " 总共：" << getTotalBill() << endl;
        }
    }

    int main() {
        OrderMeal myMeal = new OrderMeal();
        myMeal.addOrder("SpicyHotPotSoup",30,1);
        myMeal.addOrder("LightHotPotSoup",30,1);
        myMeal.addOrder("SesameSauce",5,1);
        myMeal.addOrder("PeanutSauce",5,1);
        myMeal.addOrder("Bowl",0,2);
        myMeal.addOrder("Chopsticks",0,2);
        myMeal.addOrder("Beef",20,4);
        myMeal.addOrder("Fish",35,1);
        myMeal.addOrder("Chicken",15,1);
        myMeal.addOrder("Cabbage",10,1);
        myMeal.addOrder("Potato",8,2);
        myMeal.addOrder("Maize",12,1);
        myMeal.dump();
    }

输出：
名称：麻辣锅底  价格：30  数量：1
名称：清汤锅底  价格：30  数量：1
名称：芝麻蘸料  价格：5   数量：1
名称：花生蘸料  价格：5   数量：1
名称：碗       价格：0   数量：2
名称：筷子     价格：0   数量：2
名称：牛肉     价格：20  数量：4
名称：鱼肉     价格：35  数量：1
名称：鸡肉     价格：30  数量：1
名称：白菜     价格：10  数量：1
名称：土豆     价格：8   数量：2
名称：玉米     价格：12  数量：1
总共：253

### 组合模式
组合模式比较简单，主要是解决那些树形接口的系统或者事物，而又想在各个系统节点上有类似的属性和接口
比如一个集团，有总公司，总公司下属有财务和人事部门,负责财务的结算和人员招聘;总公司下面有区分中心公司，中心公司下属又有各个城市的子公司，这些不同级别的子公司都需要财务和人事部门，这就涉及到节点的接口统一问题了，可以用到组合模式，例子：

    class Department {
        Department() {}
        ~Department() {}
        virtual void display() = 0;
    }
    class Finance: public Department {
        Finance() {}
        ~Finance() {}
        void display() {
            cout << "部门: 财务" << endl;
        }
    }
    class Personnel: public Department {
        Personnel() {}
        ~Personnel() {}
        void display() {
            cout << "部门: 人事" << endl;
        }
    }
    class Company {
        string name;
        Department * m_departments[] = [];
        Company *m_pcompanys[] = [];
        string qinzui = "=";
        Company(string name) {
            name = name;
            m_departments.push(new Finance());
            m_departments.push(new Personnel());
        }
        ~Company() {}

        virtual void addCompany() = 0;
        virtual void removeCompany() = 0;
        virtual void display() = 0;
        void addQinzui(string qinzui) {
            qinzui = qinzui;
        }
    }
    class ConCompany: public Company {
        ConCompany(string name): Company(name) {}
        ~ConCompany() {}

        void addCompany(Company *pcomp) {
            pcomp.addQinzui(qinzui+"===");
            m_pcompanys.push(pcomp);
        }

        void removeCompany(Company *pcomp) {
            m_pcompanys.remove(pcomp);
        }

        void dump() {
            cout << qinzui << name << endl;
            if(m_departments.length != 0) {
                for(int index=0;index < m_departments.length;index++) {
                    cout << qinzui;
                    m_departments[index].display();
                }
            }
            if(m_pcompanys.length != 0) {
                for(int index=0;index < m_departments.length;index++) {
                    m_pcompanys[index].dump();
                }
            }
        }
    }

    int main() {
        ConCompany huadong = new ConCompany("华东分公司");
        huadong.addCompany(new ConCompany("上海分公司"))；
        huadong.addCompany(new ConCompany("杭州分公司"));
        ConCompany huabei = new ConCompany("华北分公司");
        huabei.addCompany(new ConCompany("北京分公司"));
        huabei.addCompany(new ConCompany("天津分公司"));
        ConCompany zong = new ConCompany("总公司");
        zong.addCompany(huadong);
        zong.addCompany(huabei);
        zong.dump();
    }

    输出：
    =总公司
    =部门: 财务
    =部门: 人事
    ====华东分公司
    ====部门: 财务
    ====部门: 人事
    ========上海分公司
    ========部门: 财务
    ========部门: 人事
    ========杭州分公司
    ========部门: 财务
    ========部门: 人事
    ====华北分公司
    ====部门: 财务
    ====部门: 人事
    ========北京分公司
    ========部门: 财务
    ========部门: 人事
    ========天津分公司
    ========部门: 财务
    ========部门: 人事

