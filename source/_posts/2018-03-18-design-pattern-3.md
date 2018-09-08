---
title: 设计模式三
date: 2018-03-18 20:08:42
tags: 设计
categories: 后端
comments: true
---

本节介绍几种结构型模式，看着比较相似，比较容易混淆，在此描述希望有所区分
<!--more-->

### 装饰器模式
装饰器模式主要是为了复用现有对象和类，在不修改现有类的前提下添加新功能。
一般扩展类功能通常采用继承来实现，而装饰器可直接扩展新功能，防止继承造成的子类膨胀
下面以X战警的例子来举个例子，X-man是变异人，是在普通人类的基础上拥有特殊能力的一类人：

    class Person {
        string name;

        Person(string name) {name = name;}
        ~Person() {name = "";}

        void display() {
            cout << "===名字：" << name << endl;
        }
    }
    class Man: public Person {
        Man(string name): Person(name) {}
        ~Man() {}
    }
    class Woman: public Person {
        Woman(string name): Person(name) {}
        ~Woman() {}
    }
    class Skill{
        string m_skill;
        Skill(string skill) {m_skill = skill;}
        ~Skill() {m_skill = "";}
        void display() {
            cout << m_skill << " ; ";
        }
    }

    class XMan {
        Person m_tPerson;
        Skill m_skillList = [];

        XMan(Person person) {
            m_tPerson = person;
        }
        ~XMan() {m_skillList = [];}

        void addSkill(Skill skill) {
            m_skillList.push(skill);
        }
        void display() {
            m_tPerson.display();
            cout << "技能：" << endl;
            for(int index=0;index < m_skillList.length;index++) {
                m_skillList[index].display();
            }
            cout << endl;
        }
    }
    //暴风女
    class Storm: public XMan {
        Storm(Person person): XMan(person) {}
        ~Storm() {}
    }
    //镭射眼
    class Cyclops: public XMan {
        Cyclops(Person person): XMan(person) {}
        ~Cyclops() {}
    }
    //金刚狼
    class Wolverine: public XMan {
        Wolverine(Person person): XMan(person) {}
        ~Wolverine() {}
    }

    int main() {
        XMan storm = new Storm(new Woman("暴风女"));
        storm.addSkill(new Skill("操纵雷电"));
        storm.addSkill(new Skill("操纵暴风"));
        storm.addSkill(new Skill("操纵龙卷风"));
        storm.display();
        XMan cyclops = new Storm(new Man("镭射眼"));
        cyclops.addSkill(new Skill("发射冲击波"));
        cyclops.display();
        XMan wolverine = new Wolverine(new Man("金刚狼"));
        wolverine.addSkill(new Skill("延缓衰老"));
        wolverine.addSkill(new Skill("自愈"));
        wolverine.addSkill(new Skill("金刚爪"));
        wolverine.display();
    }

输出：
===名字：暴风女
技能 ： 操纵雷电; 操纵暴风; 操纵龙卷风;
===名字：镭射眼
技能 ： 发射冲击波;
===名字：金刚狼
技能 ： 延缓衰老; 自愈; 金刚爪;

### 代理模式
代理模式是新增代理类来对基础类进行访问控制的一种设计手段，由代理类提供调用接口，隐藏基础类的直接访问，此外还可以完成其他事物处理，包括：

- 隐藏基础类访问的复杂性
- 控制外部用户对基础类的直接访问，比如：进行访问算法优化
- 中间代理层可以对访问方式进行转化，比如：用户加解密

以访问数据库的代理处理为例：

    class DataBase {
        OracleDB m_db;
        DataBase() {}
        ~DataBase() {}

        object getData(string id) {
            return m_db.get(id);
        }

        void writeData(object data) {
            m_db.write(data);
        }

        void deleteData(string id) {
            m_db.delete(id);
        }
    }
    class UserService {
        string userName;
        string password;
        UserService(string name,string pwd) {
            userName = name;
            password = pwd;
        }
        ~UserService() {}

        boolean testUser(string name,string pwd) {
            return (userName == name) && (password == pwd);
        }
    }
    class DataBaseService {
        DataBase m_database;
        UserService m_user = new UserService("aaa","123456");
        DataBaseService() {}
        ~DataBaseService() {}

        object get(string id) {
            m_database.getData(id);
        }

        void del(string id,string name,string pwd) {
            if(m_user.testUser(name,pwd)) {
                m_database.deleteData(id);
            }
        }

        void post(object data,string name,string pwd) {
            if(m_user.testUser(name,pwd)) {
                m_database.writeData(id);
            }
        }
    }

    int main() {
        DataBaseService dbService.;
        dbService.post(data,"aaa","123456");
        dbService.get("aaa");
        dbService.del("aaa");
    }


### 外观模式

### 代理模式