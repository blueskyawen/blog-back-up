---
title: 表单-响应式构建表单
date: 2017-11-26 18:09:52
tags: Augular
categories: 前端
comments: true
---

# 什么是响应式表单

响应式表单主要是数据管理的方式不同，它是在类中显式的管理我们的数据流和数据模型，直接操作数据模型进行状态控制。同时，它还以可以直接在组件类中创建表单控件树，并绑定到相应的标签上来构建表单，简单方便。
<!--more-->
响应式表单有以下特点：

- 显式的管理数据方式，方便了校验和测试
- 表单的控件值和有效性状态的更新是同步的，不会有更新时序问题，更易单元测试;而之前的模板驱动表单是更新异步的，存在更新的时序和效率问题，在有些场景下会存在使用瓶颈
- 由于表单数据模型是自定义的，所以可以根据数据对象的结构来定义结构类似的模型树满足，这样数据更新更方便

# 基本组成

**AbstractControl**： 三个具体表单类的抽象基类。 并为它们提供了一些共同的行为和属性，其中有些是可观察对象（Observable）。

**FormControlDirective/FormControlName** : 用于跟踪一个单独的表单控件的值和有效性状态。它对应于一个HTML表单控件，比如输入框和下拉框。请FormControlDirective可单独使用，FormControlName和Group结合使用

**FormGroupDirective/FormGroupName** : 用于 跟踪一组AbstractControl的实例的值和有效性状态。 该组的属性中包含了它的子控件。 组件中的顶级表单就是一个FormGroup。而FormGroupName在多级group结构使用

**FormArray/FormArrayName** : 用于跟踪AbstractControl实例组成的有序数组的值和有效性状态。

这些指令都包含在ReactiveFormModule，使用只要包含即可：

    import { ReactiveFormsModule } from '@angular/forms';
    import { FormControl } from '@angular/forms';
    @NgModule({
      imports: [
        ReactiveFormsModule
      ],
      declarations: [
        AppComponent,
        FormControl
      ]
    }

# 控件指令的使用

上面的指令一般可以单独使用，也可以组合使用，下面描述几种常用的使用场景,为简单标识，我们约定各指令简写：

- FormGroup -- FG
- FormControl -- FC
- FormArray -- FA

**（1）FormControl单独使用**
这是最常见的方式，可以使用单独创建formControl并绑定到表单输入框上，而在模板驱动表单里这个是自动创建的
.ts

    import { Component } from '@angular/core';
    import { FormControl, Validators } from '@angular/forms';
    import { forbiddenNameValidator } from '../forbinden-name.directive';

    @Component({
      selector: 'reactive-form',
      templateUrl: './reactive-form.component.html',
      styleUrls: ['../template-form/template-form.component.css']
    })

    export class ReactiveFormComponent {
      name : any;

      constructor() {
        this.name = new FormControl('Jack',
                                    [Validators.required,
                                      Validators.minLength(4),
                                      forbiddenNameValidator(/[@\$&0-9]/)]);
      }
    }

.html

    <form>
      <div class="form-group">
        <label for="name">姓名</label>
        <div class="form-right">
          <input type="text" id="name" [formControl]="name"/>
        </div>
        <span class="reqiured"> * </span>
      </div>
    </form>
    <div class="error" *ngIf="(!name.pristine || name.dirty) && name.touched">
      <p *ngIf="name.errors?.required">Name is requred!!</p>
      <p *ngIf="name.errors?.minlength">Name length is under 3!!</p>
      <p *ngIf="name.errors?.forbiddenName">Name can not has @ $ & and number!!</p>
    </div>
    <p>{{name.value}}</p>
    <p>errors:{{name.errors | json}}</p>

可以看到，我们是在组件类里定义的控件实例，并绑定到控件标签使用：
***FormControl(初始值，同步校验函数，异步校验函数)***
表单校验直接使用的是函数形式,多个校验函数组成数据作为实例化入参，我们在模板同样可以像模板驱动表单一样直接额只用name去获取控件的状态和错误信息，用来显示提示
其中errors信息值这样的，当输入ja3：

    {
      "minlength": { "requiredLength": 4, "actualLength": 3 },
      "forbiddenName": { "value": "ja3" }
    }

**（2）FormControl/FormGroup组合使用**
主要是多属性组合使用，FG：[FC, FC, ... ]

    import { Component } from '@angular/core';
    import { FormControl, Validators, FormGroup } from '@angular/forms';
    import { forbiddenNameValidator } from '../forbinden-name.directive';

    @Component({
      selector: 'reactive-form',
      templateUrl: './reactive-form.component.html',
      styleUrls: ['../template-form/template-form.component.css']
    })
    export class ReactiveFormComponent {
      myForm : any;

      constructor() {
        this.myForm = new FormGroup({
          name: new FormControl('Jack',
                                [Validators.required,
                                  Validators.minLength(4),
                                  forbiddenNameValidator(/[@\$&0-9]/)]),
          password: new FormControl('',Validators.required)
        });
      }

      get name() { return this.myForm.get('name');}

      get password() { return this.myForm.get('password');}
    }

    <form [formGroup]="myForm">
      <div class="form-group">
        <label for="name">姓名</label>
        <div class="form-right">
          <input type="text" id="name" formControlName="name"/>
        </div>
        <span class="reqiured"> * </span>
      </div>
      <div class="error" *ngIf="(!name.pristine || name.dirty) && name.touched">
        <p *ngIf="name.errors?.required">Name is requred!!</p>
        <p *ngIf="name.errors?.minlength">Name length is under 3!!</p>
        <p *ngIf="name.errors?.forbiddenName">Name can not has @ $ & and number!!</p>
      </div>
      <p>{{name.errors | json}}</p>

      <div class="form-group">
        <label for="password">密码</label>
        <div class="form-right">
          <input type="password" id="password" formControlName="password"/>
        </div>
        <span class="reqiured"> * </span>
      </div>
    </form>
    <p>{{myForm.value | json}}</p>

其中form值为：

    {
      "name": "Jack",
      "password": ""
    }

还有一些不一样的特点：

- 跟单独使用不一样，FormControl在标签中的使用为formControlName,用来绑定formGroup里的一个属性
- 不能再直接使用formControl,而要通过formGroup来获取其他的formControl实例来访问，这里可以声明一个同名getter得到属性实例,然后就可以像单独使用一样访问了

当然，new实例可以使用FormBuilder类替代，上面代码可以改写为：

    import { Component } from '@angular/core';
    import { FormControl, Validators, FormGroup, FormBuilder } from '@angular/forms';
    import { forbiddenNameValidator } from '../forbinden-name.directive';

    @Component({
      selector: 'reactive-form',
      templateUrl: './reactive-form.component.html',
      styleUrls: ['../template-form/template-form.component.css']
    })
    export class ReactiveFormComponent {
      myForm : FormGroup;

      constructor(private fb : FormBuilder) {
        this.myForm = this.fb.group({
          name: ['Jack',
                 [Validators.required,Validators.minLength(4),forbiddenNameValidator(/[@\$&0-9]/)]],
          password: ['',Validators.required]
        });
      }

      get name() { return this.myForm.get('name');}

      get password() { return this.myForm.get('password');}
    }


**(3) 多级FormGroup**
主要是多Group和Control嵌套使用，FG：[FC, FG, ... ]

    <form #tForm="ngForm" (ngSubmit)="save()">
      <div class="form-group">
        <label>姓名</label>
        <div class="form-right">
          <input type="text" name="name" [(ngModel)]="name" minlength="3" required [forbiddenName]="name" #name1="ngModel"/>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <fieldset ngModelGroup="address" #address="ngModelGroup">
        <div class="form-group">
          <label>国家</label>
          <div class="form-right">
            <input type="text" name="coun" [(ngModel)]="address.country" />
          </div>
        </div>
        <div class="form-group">
          <label>城市</label>
          <div class="form-right">
            <input type="text" name="city" [(ngModel)]="address.city" />
          </div>
        </div>
      </fieldset>
    </form>

    this.myForm = this.fb.group({
      name: ['Jack',
             [Validators.required,Validators.minLength(4),forbiddenNameValidator(/[@\$&0-9]/)]],
      address: this.fb.group({
        country: '',
        city: ''
      }),
    });

**(4) FormControl数组**
FA：[FC, FC, ... ]

    <form [formGroup]="myForm" novalidate>
      <div formArrayName="skills">
        <div class="form-group" *ngFor="let skill of skills.controls;let i=index">
          <label *ngIf="i === 0" class="yes">技能</label>
          <label *ngIf="i !== 0" class="no"></label>
          <div class="form-right">
            <input type="text" [formControlName]="i" />
          </div>
          <a (click)="delSkill(i)" class="del"> - </a>
        </div>
      </div>
      <div class="form-group">
        <label class="no"></label>
        <div class="add">
          <a (click)="addSkill()"> + </a>
        </div>
      </div>
    </form>

    import { Component, OnInit } from '@angular/core';
    import { FormControl, Validators, FormGroup, FormBuilder,FormArray } from '@angular/forms';

    export class ReactiveFormComponent implements OnInit {
      myForm : FormGroup;
      skillList : any[] = [{'value':'Java'},{'value':'Scala'}];

      constructor(private fb : FormBuilder) {
        this.myForm = this.fb.group({
          skills: this.fb.array([])
        });
      }

      ngOnInit() {
        this.initSkillList();
      }

      initSkillList() {
        const tempArray = this.skillList.map((skill) => new FormControl(skill.value));
        const skillFormArray = this.fb.array(tempArray);
        this.myForm.setControl('skills', skillFormArray)
      }

      addSkill() {
        let temp = new FormControl();
        this.skills.push(temp);
      }

      delSkill(index : any) {
        this.skills.removeAt(index);
      }

      get skills(): FormArray {
        return this.myForm.get('skills') as FormArray;
      };
    }

可以看到，必须使用getter来访问控件数组，并使用.controls来访问控件

**(5) FormGroup数组**
FA：[FG, FG, ... ]

    <form [formGroup]="myForm" novalidate>
      <fieldset formArrayName="address">
        <div *ngFor="let oneAddress of address.controls; let i=index" [formGroupName]="i">
          <h4>Address #{{i + 1}}</h4>
          <div class="form-group">
            <label>国家</label>
            <div class="form-right">
              <input type="text" formControlName="country" />
            </div>
          </div>
          <div class="form-group">
            <label>城市</label>
            <div class="form-right">
              <input type="text" formControlName="city" />
            </div>
            <a (click)="delAdress(i)" class="del"> - </a>
          </div>
        </div>
        <div class="form-group">
          <label class="no"></label>
          <div class="add">
            <a (click)="addAdress()"> + </a>
          </div>
        </div>
      </fieldset>
    </form>

      myForm : FormGroup;

      constructor(private fb : FormBuilder) {
        this.myForm = this.fb.group({
          address: this.fb.array([]),)
        });
      }

      ngOnInit() {
        this.initAdressList();
      }

      initAdressList() {
        const tempArray = [this.fb.group({country: 'China', city: 'shanghai'})];
        const adressFormArray = this.fb.array(tempArray);
        this.myForm.setControl('address', adressFormArray)
      }

      addAdress() {
        let temp = this.fb.group({country: '', city: ''});
        this.address.push(temp);
      }

      delAdress(index : any) {
        this.address.removeAt(index);
      }

      get address(): FormArray {
        return this.myForm.get('address') as FormArray;
      };

# 完整表单例子

本节使用响应式构建与模板驱动表单一节中效果相同的表单

component.ts

    import { Component, OnInit } from '@angular/core';
    import { FormControl, Validators, FormGroup, FormBuilder,FormArray } from '@angular/forms';
    import { forbiddenNameValidator } from '../forbinden-name.directive';

    @Component({
      selector: 'reactive-form',
      templateUrl: './reactive-form.component.html',
      styleUrls: ['../template-form/template-form.component.css']
    })
    export class ReactiveFormComponent implements OnInit {
      myForm : FormGroup;
      sexs : any[] = [{'name':'女','value':'famale'},{'name':'男','value':'male'}];
      likes : any[] = [{'name':'看电视','value':'Watch Tv','isChecked':false},
                        {'name':'读书','value':'Book','isChecked':false}];
      selectedLikes : any[] = [];
      baocuns : any[] = [{'name':'是','value':'Yes','isChecked':false},{'name':'否','value':'No','isChecked':false}];
      skillList : any[] = [{'value':'Java'},{'value':'Scala'}];

      constructor(private fb : FormBuilder) {
        this.myForm = this.fb.group({
          name: ['Jack',
                 [Validators.required,Validators.minLength(4),forbiddenNameValidator(/[@\$&0-9]/)]],
          sex: ['male',Validators.required],
          shengri: ['1990-01-01',Validators.required],
          password: ['',Validators.required],
          address: this.fb.array([]),
          like: [],
          baocun:'',
          skills: this.fb.array([])
        });
      }

      ngOnInit() {
        this.initAdressList();
        this.initSkillList();

      }

      get name() { return this.myForm.get('name');}

      get password() { return this.myForm.get('password');}

      selectLikes(flag: any,value : any) {
        if(flag.target.checked) {
          this.selectedLikes.push(value);
        } else {
          this.selectedLikes = this.selectedLikes.filter((like) => {
            return like !== value;
          });
        }
        this.myForm.patchValue({
          like: this.selectedLikes
        });
      }

      initAdressList() {
        const tempArray = [this.fb.group({country: 'China', city: 'shanghai'})];
        const adressFormArray = this.fb.array(tempArray);
        this.myForm.setControl('address', adressFormArray)
      }

      addAdress() {
        let temp = this.fb.group({country: '', city: ''});
        this.address.push(temp);
      }

      delAdress(index : any) {
        this.address.removeAt(index);
      }

      get address(): FormArray {
        return this.myForm.get('address') as FormArray;
      };

      initSkillList() {
        const tempArray = this.skillList.map((skill) => new FormControl(skill.value));
        const skillFormArray = this.fb.array(tempArray);
        this.myForm.setControl('skills', skillFormArray)
      }

      addSkill() {
        let temp = new FormControl();
        this.skills.push(temp);
      }

      delSkill(index : any) {
        this.skills.removeAt(index);
      }

      get skills(): FormArray {
        return this.myForm.get('skills') as FormArray;
      };
    }

component.html

    <form [formGroup]="myForm" novalidate>
      <div class="form-group">
        <label for="name">姓名</label>
        <div class="form-right">
          <input type="text" id="name" formControlName="name"/>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <div class="form-group">
        <label>性别</label>
        <div  class="form-right">
          <select name="Sex" formControlName="sex">
            <option *ngFor="let sex of sexs" [value]="sex.value">{{sex.name}}</option>
          </select>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <div class="form-group">
        <label>出生年月</label>
        <div class="form-right">
          <input type="date" formControlName="shengri" required/>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <div class="form-group">
        <label for="password">密码</label>
        <div class="form-right">
          <input type="password" id="password" formControlName="password"/>
        </div>
        <span class="reqiured"> * </span>
      </div>

      <fieldset formArrayName="address">
        <div *ngFor="let oneAddress of address.controls; let i=index" [formGroupName]="i">
          <h4>Address #{{i + 1}}</h4>
          <div class="form-group">
            <label>国家</label>
            <div class="form-right">
              <input type="text" formControlName="country" />
            </div>
          </div>
          <div class="form-group">
            <label>城市</label>
            <div class="form-right">
              <input type="text" formControlName="city" />
            </div>
            <a (click)="delAdress(i)" class="del"> - </a>
          </div>
        </div>
        <div class="form-group">
          <label class="no"></label>
          <div class="add">
            <a (click)="addAdress()"> + </a>
          </div>
        </div>
      </fieldset>

      <div  class="form-group"  *ngFor="let like2 of likes;let i=index">
        <label *ngIf="i ===0" class="yes">兴趣爱好</label>
        <label *ngIf="i !==0" class="no"></label>
        <div>
          <input type="checkbox" #lock formControlName="like" [value]="like2.value"
                 (change)="selectLikes($event,lock.value)"/>{{like2.name}}
        </div>
      </div>

      <div formArrayName="skills">
        <div class="form-group" *ngFor="let skill of skills.controls;let i=index">
          <label *ngIf="i === 0" class="yes">技能</label>
          <label *ngIf="i !== 0" class="no"></label>
          <div class="form-right">
            <input type="text" [formControlName]="i" />
          </div>
          <a (click)="delSkill(i)" class="del"> - </a>
        </div>
      </div>
      <div class="form-group">
        <label class="no"></label>
        <div class="add">
          <a (click)="addSkill()"> + </a>
        </div>
      </div>

      <div *ngFor="let bao of baocuns;let i=index" class="form-group">
        <label *ngIf="i === 0" class="yes">是否保存</label>
        <label *ngIf="i !== 0" class="no"></label>
        <div>
          <input  type="radio" formControlName="baocun" [value]="bao.value"/>{{bao.name}}
        </div>
      </div>

      <div class="button-group">
        <button class="confirm" [disabled]="!myForm.valid">提交</button>
        <button class="concel" (click)="myForm.reset()">取消</button>
      </div>
    </form>
    <p>{{myForm.value | json}}</p>

样子如下：
![reactiveForm1](/images/reactiveForm1.png)
![reactiveForm2](/images/reactiveForm2.png)


# 其他有用的东西：

**1 . 同步和异步更新**

响应式表单是同步的。
使用响应式表单，我们会在代码中创建整个表单控件树。 我们可以立即更新一个值或者深入到表单中的任意节点，因为所有的控件都始终是可用的。

模板驱动表单是异步的。
模板驱动表单会委托指令来创建它们的表单控件。 为了消除“检查完后又变化了”的错误，这些指令需要消耗一个以上的变更检测周期来构建整个控件树。 这意味着在从组件类中操纵任何控件之前，我们都必须先等待一个节拍。
比如，如果我们用@ViewChild(NgForm)查询来注入表单控件，并在生命周期钩子ngAfterViewInit中检查它，就会发现它没有子控件。 我们必须使用setTimeout等待一个节拍才能从控件中提取值、测试有效性，或把它设置为新值。

**2 . 模板驱动表单和响应式表单的区别**

**模板驱动表单**

- 数据更新是异步的，更新流程多，依赖于变更周期
- 表单控件在模板中通过指令创建和更新操作，如：ngModel
- 表单校验使用指令验证，指令包装校验函数
- 每次数据更新只是更新部分值，在变更周期后触法
- 难测试

**响应式表单**

- 数据更新是同步的，更新流程少
- 表单控件在ts中直接创建和更新操作,如：FormControl
- 表单校验使用校验函数进行验证
- 每次数据更新都是返回一个新实例
- 易测试

[数据更新流程](https://www.angular.cn/guide/forms-overview#data-flow-in-forms)
