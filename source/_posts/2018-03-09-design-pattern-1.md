---
title: 设计模式一
date: 2018-03-09 22:59:29
tags: 设计
categories: 后端
comments: true
toc: true
---

设计模式是一种经验总结，是软件开发人员在长期的实践中对所遇问题的常用解决方案。设计模式的目的是让我们的软件设计方法化，开发的程序更具重用性，灵活性和可扩展性。
<!--more-->
设计模式是针对面向对象的设计方法总结，有一个需要遵从的“开放-闭合原则”：**对扩展的开放，对修改的闭合**，即功能扩展时易增加且不改动原有代码，功能变更时可以少修改代码，这个原则可以借助面向对象的抽象和继承来设计实现。
设计模式有很多设计原则，我摘取了几个，认为只要符合下面几个原则，基本上是较好的设计：

- 单个类封装设计，功能职责单一
- 对对象或者行为接口进行隔离，减低耦合
- 针对接口编程，而非过程编程
- 依赖于抽象而非具体
- 优先对象组合而非继承，只在有意义的时候进行继承

> 以下例子均以C++实现

# 工厂模式
工厂模式封装对象的创建，配合类的抽象继承，隐藏子对象的创建过程，提供统一的创建接口
比如，宠物店的例子：

    class Pet {
        private:
            string name;
        public:
            Pet(string name) { name = name; }
            ~Pet() { name = ""; }
            void sayName() {
                cout << "My name is : " << name << " ! ";
            }
            virtual void shut() = 0;
    }
    class Dog: public Pet {
        Dog(string name): Pet(name) {}
        ~Dog() {}
        void shut() {
            sayName();
            cout << "旺旺～" << endl;
        }
    }
    class Cat: public Pet {
        Cat(string name): Pet(name) {}
        ~Cat() {}
        void shut() {
            sayName();
            cout << "喵喵～" << endl;
        }
    }
    class PetStore {
        Pet * pInst;
        PetStore() { pInst = null; }
        ~PetStore() { pInst = null; }
        getOnePet(string type,string name) {
            switch(type) {
                case "DOG":
                    pInst = new Dog(name);
                    break;
                case "CAT":
                    pInst = new Cat(name);
                    break;
                default: break;
            }
            return pInst;
        }
    }
    int main() {
        PetStore petStore = new PetStore();
        Pet * myPet;
        myPet = petStore.getOnePet("DOG","旺财");
        myPet->shut();
        myPet = petStore.getOnePet("CAT","加菲");
        myPet->shut();
    }

输出：
My name is : 旺财 ; 旺旺～
My name is : 加菲 ; 喵喵～

# 抽象工厂模式
抽象工厂是处理多集群的一种方式，将工厂抽象化，先创建工厂，再让具体的工厂创建具体的实例对象，比如：

- 宠物 - 小猫/小狗 - 宠物店 - 店铺
- 服装 - 衣服/裤子 - 服装店 - 店铺

如上，将工厂对象-店铺先作对象，创建具体的店铺后，由相应的店铺创建各自的对象，实现方式和工厂模式相似，这就不描述了

# 工厂方法模式
工厂模式可以看到隐藏了对象的创建细节，提供了统一的接口，但是可以看到的是当功能扩展时需要修改对象工厂的内部实现，比如：想增加个小猪，除了新增小猪类外，还要修改store的创建逻辑，这就不符合“开放-闭合”原则，可以使用工厂方法将各个对象的工厂独立开来

    class Pet {
        private:
            string name;
        public:
            Pet(string name) { name = name; }
            ~Pet() { name = ""; }
            void sayName() {
                cout << "My name is : " << name << " ! ";
            }
            virtual void shut() = 0;
    }
    class Dog: public Pet {
        Dog(string name): Pet(name) {}
        ~Dog() {}
        void shut() {
            sayName();
            cout << "旺旺～" << endl;
        }
    }
    class Cat: public Pet {
        Cat(string name): Pet(name) {}
        ~Cat() {}
        void shut() {
            sayName();
            cout << "喵喵～" << endl;
        }
    }
    class Store {
        Pet * pInst;
        Store() { pInst = null; }
        ~Store() { pInst = null; }
        virtual Pet * getOnePet(string name) = 0;
    }
    class DogStore: public Store {
        DogStore() { }
        ~DogStore() {}
        Pet * getOnePet(string name) {
            pInst = new Dog(name);
            return pInst;
        }
    }
    class CatStore: public Store  {
        CatStore() {}
        ~CatStore() {}
        Pet * getOnePet(string name) {
            pInst = new Cat(name);
            return pInst;
        }
    }
    int main() {
        Store *petStore = null;
        Pet * myPet;
        petStore = new DogStore();
        myPet = petStore->getOnePet("旺财");
        myPet->shut();
        petStore = new CatStore();
        myPet = petStore->getOnePet("加菲");
        myPet->shut();
    }

如上，每种宠物一个工厂，增加只要继承扩展就行。简单工厂模式就好比是大商城，啥都卖;而工厂方法模式就是专卖店，只卖一种类型的宠物。

> 两种模式各有优缺点，当种类不多时可以采用简单工厂，而当种类多而且各工厂内部复杂时可考虑工厂方法

# 策略模式
工厂模式是对事物属性变化的封装，当对象的属性相对稳定，行为变化频繁的时候，就需要对行为进行封装适应。而策略模式正是针对这种问题的一个解决方法。
比如，简易计算器，数字的四则运算，处理的属性不变，而计算方式却大不相同：

    class OPerate {
        private:
            int num1;
            int num2;
            int operResult;
        public:
            Operate(int n1,int n2) {
                num1 = n1;
                num2 = n2;
                operResult = 0;
            }
            ~Operate(int n1,int n2) {
                num1 = 0;
                num2 = 0;
                operResult = 0;
            }
            void displayResult() {
                cout << "The result is : " << operResult << endl;
            }
            virtual void doOperation() = 0;
    }
    class OperateAdd: public OPerate {
        OperateAdd(int n1,int n2): OPerate(n1,n2) {}
        ~OperateAdd() {}
        void doOperation() {
            operResult = num1 + num2;
        }
    }
    class OperateMultiple: public OPerate {
        OperateMultiple(int n1,int n2): OPerate(n1,n2) {}
        ~OperateMultiple() {}
        void doOperation() {
            operResult = num1 * num2;
        }
    }
    class OperateContext {
        private:
            OPerate * pInst;
        public:
            OperateContext(OPerate * pInstz) { pInst = pInstz; }
            ~OperateContext() { pInst = null; }
            void getOperateResult() {
                pInst->doOperation();
            }
            void dump() {
                pInst->displayResult();
            }
    }
    int main() {
        OperateContext context;
        context = new OperateContext(new OperateAdd(3,5));
        context.getOperateResult();
        context.dump();
        context = new OperateContext(new OperateMultiple(3,5));
        context.getOperateResult();
        context.dump();
    }

输出：
The result is : 8
The result is : 15

# 策略-工厂组合模式
策略模式最后的使用将对象的行为暴露出来不合适，结合工厂模式来隐藏内部过程，提供统一使用对象和接口，重构上面的部分代码：

    class OperateContext {
        private:
            OPerate * pInst = null;
        public:
            OperateContext(string type,int n1,int n2) {
                if(pInst == null) {
                    pInst = createInst(type,n1,n2);
                }
            }
            ~OperateContext() { pInst = null; }
            OPerate * createInst(string type,int n1,int n2) {
                switch(type) {
                    case "ADD":
                        return new OperateAdd(n1,n2);
                    case "MULTIPLE":
                        return new OperateMultiple(n1,n2);
                    default:
                        return null;
                }
            }
            void getOperateResult() {
                if(pInst != null) {
                    pInst->doOperation();
                }
            }
            void dump() {
                if(pInst != null) {
                    pInst->displayResult();
                }
            }
    }
    int main() {
        OperateContext *pcontext = null;
        pcontext = (OperateContext *)new OperateContext("ADD",3,5);
        context->getOperateResult();
        context->dump();
        pcontext = (OperateContext *)new OperateContext("MULTIPLE",3,5);
        context->getOperateResult();
        context->dump();
    }

# 单例模式
单例模式利用类的静态属性的特点使得一个类有且只能有一个实例，减少内存消耗
比如：

    char g_auData[sizeof(Sington)];
    Sington * Sington::pInst = null;
    class Sington {
        private:
            static Sington * pInst;
        public:
            static Sington * getInstance() {
                if(pInst != null) {
                    return pInst;
                } else {
                    pInst = (Sington * )g_auData;
                    new (pInst) Sington();
                    return pInst;
                }
                void display() {
                    cout << "I am sington!" << endl;
                }
            }
    }
    int main() {
        Sington * pContext = Sington::getInstance();
        pContext->display();
    }

输出：I am sington!


