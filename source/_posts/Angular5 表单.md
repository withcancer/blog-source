---
title: angular5 表单
date: 2016-5-21
categories:
- 前端
- angular
tags:
- 前端
- angular
---
1. 模板式验证

我们把HTML表单控件（比如``<input>``和``<select>``）放进组件模板中，并用ngModel等指令把它们绑定到组件中数据模型的属性上。

不用自己创建Angular表单控件对象。Angular指令会使用数据绑定中的信息创建它们。 我们不用自己推送和拉取数据。Angular使用ngModel来替你管理它们。 当用户做出修改时，Angular会据此更新可变的数据模型。

利用双向绑定，模板引用值，当name为invalid时，表单提示将会出现。对于数据的控制都放在模板内，可用的属性可能包括：required,minLength等。
``` html
{{diagnostic}}

<div class="form-group">
  <label for="name">Name</label>
<input type="text" class="form-control" required [(ngModel)]="model.name" name="name" #name="ngModel">
<div [hidden]="name.valid || name.pristine" class="alert alert-danger">
  Name is required
</div>
</div>

<div class="form-group">
  <label for="alterEgo">Alter Ego</label>
  <input type="text" class="form-control" [(ngModel)]="model.alterEgo" miniLength="3" name="alterEgo" #alterEgo>
</div>
<div *ngIf="alterEgo.errors?.miniLength" class="alert alert-danger">Should be at least 3 characters</div>

<div class="form-group">
  <label for="power">Hero Power</label>
  <select class="form-control required" [(ngModel)]="model.power" name="power">
    <option *ngFor="let pow of powers" [value]="pow">{{pow}}</option>
  </select>
</div>
```
2. 响应式验证

把对表单的控制都放在组件一侧，统一控制，不在模板内进行处理。
``` javascript

constructor(private fb: FormBuilder) {}

ngOnInit(): void {
  this.heroForm = this.fb.FormGroup({
    'name': new FormControl(this.hero.name, [
      Validators.required,
      Validators.minLength(4),
      forbiddenNameValidator(/bob/i)
    ]),
    'alterEgo': new FormControl(this.hero.alterEgo),
    'power': new FormControl(this.hero.power, Validators.required)
  });
}

get name() { return this.heroForm.get('name'); }

get power() { return this.heroForm.get('power'); }

export function forbiddenNameValidator(nameRe: RegExp): ValidatorFn {
  return (control: AbstractControl): {[key: string]: any} => {
    const forbidden = nameRe.test(control.value);
    return forbidden ? {'forbiddenName': {value: control.value}} : null;
  };
}
```