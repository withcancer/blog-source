---
title: Angular5 自定义管道与指令
date: 2016-5-21
categories:
- 前端
- angular
tags:
- 前端
- angular
---
Angular有很多内置指令与管道，但有时候还是要添加自定义的管道或指令。
# 自定义指令
可以通过``ng g d``选项来创建一个新的指令。
例如可以定义一个``highlight``指令，并可以在其中应用``@HostListener,@HostBingding,@Input``等装饰器。
``` typescript
@Directive({
  selector: '[highlight]'
})
export class HighlightDirective {

  constructor(private elem: ElementRef, private renderer: Renderer) { }
  private color :string;

  @Input('highlight') 
  highlightColor: string;

  @HostListener('mouseenter')
  onMouseEnter() {
      console.log(this.highlightColor);
      this.renderer.setElementStyle(this.elem.nativeElement, 'backgroundColor', this.highlightColor);
  }

  @HostListener('mouseleave')
  onMouseLeave() {
      this.renderer.setElementStyle(this.elem.nativeElement, 'backgroundColor', null);
  }
  
  @HostBinding('attr.role') role = 'button';
}
```
# 自定义管道
可以使用``ng g p``来创建一个新的自定义管道。
例如，可以创建一个判断是否为0的管道。
``` typescript
@Pipe({
  name: 'isZeroPipe'
})
export class IsZeroPipe implements PipeTransform {

  transform(value: any, args?: any): any {
    if(value){
      return 'NotZero';
    }else{
      return 'Zero';
    }
  }
}
```
## 内置管道


管道|类型|功能
---|---|---
DatePipe|	纯管道	|日期管道，格式化日期
JsonPipe|	非纯管道|	将输入数据对象经过JSON.stringify()方法转换后输出对象的字符串
UpperCasePipe|	纯管道|	将文本所有小写字母转换成大写字母
LowerCasePipe|	纯管道|	将文本所有大写字母转换成小写字母
DecimalPipe|	纯管道|	将数值按特定的格式显示文本
CurrentcyPipe|	纯管道|	将数值转百分比格式
SlicePipe|	非纯管道|	将数组或者字符串裁剪成新子集