---
title: JavaScript设计模式-享元模式
date: 2019-08-24 13:48:08
tags: 
- JavaScript设计模式
categories: 
- JavaScript设计模式
---

## 定义

它使用共享物件，用来尽可能减少内存使用量以及分享资讯给尽可能多的相似物件；它适合用于当大量物件只是重复因而导致无法令人接受的使用大量内存。通常物件中的部分状态是可以分享。常见做法是把它们放在外部数据结构，当需要使用时再将它们传递给享元。



## 实现

假设一个一个情景, 有男女服装个100套, 需要租模特来试穿衣服, 传统做法就是男女各找100个模特试穿每一见衣服

```javascript
// 雇佣模特
let HireModel = function(sex,clothes){
  this.sex = sex;
  this.clothes = clothes;
};
  
HireModel.prototype.wearClothes = function(){
  console.log(this.sex + '试穿' + this.clothes);
};
/*******试穿**********/
for(let i=0;i<100;i++){
  let model = new HireModel('male','第'+i+'款男衣服');
  model.wearClothes();
}
for(let i=0;i<100;i++){
  let model = new HireModel('female','第'+i+'款女衣服');
  model.wearClothes();
}
```



采用享元模式则只需要男女模特各一名, 试穿所有衣服

```javascript
//雇佣模特
var HireModel = function(sex){
  //内部状态是性别
  this.sex = sex;
};
HireModel.prototype.wearClothes = function(clothes){
  console.log(this.sex+"穿了"+clothes);
};

//工厂模式，负责造出男女两个模特
var ModelFactory = (function(){
  var cacheObj = {};
  return {
    create:function(sex){
      //根据sex分组
      if(cacheObj[sex]){
        return cacheObj[sex];
      } else {
        cacheObj[sex] = new HireModel(sex);
        return cacheObj[sex];
      }
    }
  };
})();
//模特管理
var ModelManager = (function(){
  //容器存储：1.共享对象 2.外部状态
  var vessel = {};
  return {
    add:function(sex,clothes,id){
      //造出共享元素:模特
      var model = ModelFactory.create(sex);
      //以id为键存储所有状态
      vessel[id] = {
        model:model,
        clothes:clothes
      };
    },
    wear:function(){
      for(var key in vessel){
        //调用雇佣模特类中的穿衣服方法。
        vessel[key]['model'].wearClothes(vessel[key]['clothes']);
      }
    }
  };
})();


/*******通过运行时间测试性能**********/
for(var i=0;i<100;i++){
  ModelManager.add('male','第'+i+'款男衣服',i);
  ModelManager.add('female','第'+i+'款女衣服',i);
}
ModelManager.wear(); 
```

