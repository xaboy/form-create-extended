<h1 align="center">form-create</h1>

<p align="center">
  <a href="https://github.com/xaboy/form-create/blob/master/LICENSE">
    <img src="https://img.shields.io/badge/License-MIT-yellow.svg" />
  </a>
  <a href="https://github.com/xaboy">
    <img src="https://img.shields.io/badge/Author-xaboy-blue.svg" />
  </a>
  <a href="https://www.npmjs.com/package/form-create">
    <img src="https://badge.fury.io/js/form-create.svg" alt="version" />
  </a>
</p>

<p align="center">
  <b>具有数据收集、校验和提交功能的表单生成器，包含复选框、单选框、输入框、下拉选择框等元素以及,省市区三级联动,时间选择,日期选择,颜色选择,滑块,评分,框架,文件/图片上传功能，支持事件扩展。</b>
</p>
<br />

## form-cretea组件开发文档

### 相关文件

`src/components` 组件目录

`factory/handler`  组件处理器的构造器

`factory/render`  组件渲染器的构造器

`factory/make`  组件生成器的构造器



### 组件的生成规则 Rule

`type` 组件的类型

`field` 字段名称

`value` 字段值

Object: `props` 组件配置参数

Array: `options` 可选择参数,例如radio组件,checkbox组件,select组件

Object: `slot` 组件的插槽

Object: `event` 事件,**事件名称前会自动追加'on-'**,例如`click:()=>{}` ,触发时 `event['on-click']()`

Array: `validate` 验证规则



### 组件的构造

```javascript
import {handlerFactory} from "../factory/handler";
import {renderFactory} from "../factory/render";
import makeFactory from "../factory/make";
//组件处理器
const handler = handlerFactory({
    //TODO 
})
//组件渲染器
const render = renderFactory({
    //TODO
    parse(){
      	return this.makeDiv();
    },
  	makeDiv(){
        return this.cvm('div',this.props
                            .style('border','1px solid red')
                            //这个key很重要
                            .key('uniqueKey')
                            .get()
                        );
    }
})
//组件生成器
const make = makeFactory('组件名称',[
    //可配置参数,有'props','event','validate','slot','options'
])

const component = {handler,render,make};

//输出组件
export default component;

export {
    handler,render,make
}

```



### 组件处理器说明 handler

>   可覆盖需要定制的方法,例如:handler.handle(),handler.getValue()

方法

**handler.vm** 表单生成器Vue实例化对象

**handler.rule** 组件生成规则

```javascript
{
    field,type,title,value,options = [],props = {},slot = {},validate = [],event = {}
}
```

**handler.unique** 组件唯一值

**handler.parseValue** 组件内部的值

**handler.refName** 组件在`vm.$refs`的名称

**handler.el** 组件的dom对象,为`vm.$refs[handler.refName]`,在首次渲染完成之前为空对象

方法

**handler.verify()** 组件首次初始化时触发,验证和初始化组件规则

**handler.handle()** 组件value发生变化时触发,用来处理value,将用户传入的值处理为组件可用的值`handler.parseValue`;

```javascript
//default 无处理,直接复制给handler.parseValue
handler.changeParseValue(handler.rule.value);
```

**handler.getField()** 获取组件field

**handler.getValidate()** 获取组件validate

**handler.getRule()**  获取组件rule

**handler.getValue()**  获取组件最终输出的value值
```javascript
//default 无处理,直接输出parseValue
    return handler.parseValue;
```

**handler.changeValue(value)**  修改value,并且重新处理value

```javascript
//default 修改value,重新处理value
handler.rule.value = value;
handler.handle();
```

**handler.getParseValue()**  获取组件内部的value

**handler.changeParseValue(value,update = true)**  获取组件内部的value

```javascript
//default 修改parseValue并且修改vm.formData中该组件的值
if(update === true)
            handler.vm.changeFormData(handler.rule.field,parseValue);
        handler.parseValue = parseValue;
```

**handler.mounted()**  组件加载完成时触发

```javascript
//default 初始化el属性
handler.el = handler.vm.$refs[handler.refName];
```



### 组件渲染器render

>   render.parse方法必须重写
>
>   **注意! 每个vNode对象必须得有和其他vNode不重复的key **



**render.handler** 组件的处理器

**render.options**  全局配置

**render.vm** 表单生成器Vue实例化对象

**render.props**  生成vNode属性的数据对象,[参考](https://cn.vuejs.org/v2/guide/render-function.html#深入-data-对象)

**render.cvm** vNode生成器

```javascript
//使用方法
vNode1 = render.cvm.make('input',render.props({
    model:vm.value,
  	value:vm.value
}).attrs({
    type:'hidden'
}).get(),[vNode2,vNode3]);
```

**render.event** 组件的事件

方法

**render.init()**  组件首次初始化时触发,验证和初始化组件

**render.parse()** 组件渲染是出发,**该方法必须重写**

**render.inputProps()** input类组件属性的数据对象




### 组件生成器 make

>   用于使用 `formCreate.maker` 快速生成

```javascript
//举例
cascaderMake = makeFactory('cascader',['props','event','validate']);
radioMake = makeFactory('radio',['options','props','event','validate']);
//就可以使用maker快速生成
//formCreate.maker.radio()
```



### 内部Vue实例化对象 vm

属性

**vm.formData** 表单组件的键值对,对应`[handler.rule.field]=>handler.parseValue`

**vm.buttonProps** 提交按钮vNode属性的数据对象

方法

**vm.changeFormData(field,value)**  修改formData指定field字段的value

**vm.removeFormData(field)** 删除formData指定的field字段

**vm.changeButtonProps(props)** 修改提交按钮vNode属性的数据对象

**vm.setField(field,value)** 在formData中新增field字段